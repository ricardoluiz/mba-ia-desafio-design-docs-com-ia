# RFC — Sistema de Webhooks de Notificação de Pedidos

## Metadados
- **Autor:** Larissa / Time de Pedidos e Plataforma
- **Status:** Para revisão
- **Data:** 2026-11-24
- **Revisores:** Larissa (Tech Lead), Marcos (PM), Bruno (Engenharia), Diego (Plataforma), Sofia (Segurança)

## Resumo executivo (TL;DR)
Implementar notificações outbound de mudanças de status usando a tabela de outbox no MySQL existente. A transação de `changeStatus` cria um snapshot do evento; um worker separado consulta pendências a cada 2 segundos e usa retry limitado e DLQ. O endpoint cadastrado recebe payload assinado com HMAC-SHA256 e chave por endpoint. O desenho adota at-least-once com `X-Event-Id` para deduplicação pelo cliente.

## Contexto e problema
Os clientes B2B hoje precisam consultar pedidos periodicamente. A mudança de status é uma operação crítica envolvendo estoque e histórico, então chamadas HTTP síncronas não podem bloquear ou desfazer o domínio. A solução precisa manter consistência entre status e evento, sem Redis ou outra infraestrutura nesta fase.

## Proposta técnica
1. **Publicação transacional:** estender `OrderService.changeStatus` para chamar `publishWebhookEvent(tx, order, fromStatus, toStatus)`. O evento guarda UUID, payload JSON renderizado, status selecionado e dados necessários ao envio.
2. **Persistência:** usar outbox no MySQL e uma tabela separada de DLQ para falhas permanentes.
3. **Processamento:** iniciar um processo separado por `src/worker.ts`/`npm run worker`, com PrismaClient próprio e polling de 2 segundos.
4. **Entrega:** filtrar endpoints ativos por customer e status, aplicar timeout de 10 segundos e assinar o corpo com HMAC-SHA256 da secret por endpoint.
5. **Recuperação:** cinco tentativas com backoff fixo e replay manual por `ADMIN` para itens da DLQ.
6. **Reuso:** seguir módulos, Zod, `AppError`, Pino, `requireRole`, logger e middleware de erro existentes.

Detalhes de modelos, payload, endpoints e matriz de erros estão no [FDD](./FDD.md).

## Alternativas consideradas
- **Chamada HTTP síncrona em `changeStatus`:** descartada porque bloquearia alterações de status e acoplaria OMS e clientes lentos. Trade-off: menor latência aparente, mas pior consistência e disponibilidade.
- **Redis Streams/Redis Cluster:** descartado por adicionar infraestrutura antes de a outbox no MySQL provar insuficiência. Trade-off: maior capacidade futura, mas maior operação nesta fase.
- **Trigger de banco:** descartado por misturar notificação à persistência e dificultar filtros, retries e observabilidade. Trade-off: menos código de aplicação, mas menor controle técnico.
- **Exactly-once:** descartado em favor de at-least-once. Trade-off: coordenação distribuída complexa versus duplicatas deduplicáveis pelo cliente.
- **Worker dentro da API:** descartado para isolar entrega da API; o trade-off é um processo operacional adicional.

## Questões em aberto
- Definir a política futura de arquivamento das linhas entregues na outbox.
- Definir quando o rate limiting de saída será necessário; observar o comportamento nesta fase.
- Definir a documentação pública completa no portal para clientes, ainda não detalhada.
- Definir a escala para múltiplos workers, incluindo particionamento por `order_id`, locking ou outra coordenação.
- Eventuais permissões além de `ADMIN` no replay podem ser endurecidas depois.

## Impacto e riscos
- **Banco:** adiciona outbox, entregas/DLQ e índices de polling.
- **Operação:** exige processo worker separado e monitoramento das filas.
- **Segurança:** exige armazenamento seguro de secrets e validação HTTPS/HMAC.
- **Domínio:** mudança de status precisa ser estendida sem perder transacionalidade.
- **Mitigação:** retries limitados, timeout, DLQ, métricas, logs estruturados e revisão de segurança.

## Decisões relacionadas
- [ADR-001 — Outbox no MySQL](./adrs/ADR-001-outbox-no-mysql.md)
- [ADR-002 — Retry, backoff e DLQ](./adrs/ADR-002-retry-backoff-e-dlq.md)
- [ADR-003 — HMAC com secret por endpoint](./adrs/ADR-003-hmac-secret-por-endpoint.md)
- [ADR-004 — At-least-once e `X-Event-Id`](./adrs/ADR-004-at-least-once-e-event-id.md)
- [ADR-005 — Worker separado com polling](./adrs/ADR-005-worker-processo-separado-polling.md)
- [ADR-006 — Reuso dos padrões existentes](./adrs/ADR-006-reuso-padroes-existentes.md)
