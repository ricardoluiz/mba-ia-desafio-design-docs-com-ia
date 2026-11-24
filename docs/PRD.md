# PRD — Sistema de Webhooks de Notificação de Pedidos

## Resumo e contexto
A feature permite que clientes B2B sejam notificados automaticamente sobre mudanças no status de seus pedidos. A integração é outbound: o OMS envia eventos a endpoints cadastrados pelos clientes; não recebe webhooks de clientes.

## Problema e motivação
Os clientes precisam consultar `GET /orders/:id` periodicamente para saber quando um pedido muda de estado. A feature elimina essa sondagem e entrega mudanças selecionadas ao destino configurado pelo customer.

## Público-alvo e cenários de uso
- **Clientes B2B:** recebem notificações e atualizam seus próprios sistemas sem polling da API.
- **Operadores/administradores:** cadastram endpoints e reexecutam entregas permanentes falhadas.
- **Time de engenharia:** mantém o processamento assíncrono e investiga falhas via histórico e observabilidade.

### Cenários
1. Um cliente cadastra uma URL HTTPS e seleciona os status desejados.
2. Um pedido passa de `PAID` para `PROCESSING`; o OMS cria um evento na outbox e o worker envia o evento ao endpoint elegível.
3. A entrega falha temporariamente; o worker usa backoff e tenta novamente.
4. Após cinco falhas, o evento entra na DLQ e um administrador pode solicitar replay.

## Objetivos e métricas de sucesso
- Entrega de eventos elegíveis com latência inferior a 10 segundos em pelo menos 95% das entregas observadas durante o período de validação.
- Coração operacional: 100% das mudanças elegíveis criam uma entrada na outbox na mesma transação da mudança de status.
- Confiabilidade: nenhuma falha temporária elimina o evento antes do consumo das cinco tentativas ou da DLQ.
- Segurança: 100% dos endpoints cadastrados passam por HTTPS e 100% das entregas possuem HMAC válida.

## Escopo
### Incluso
- Cadastro, edição, remoção e listagem de webhooks por customer.
- Seleção dos status recebidos por endpoint.
- Secret gerada pelo servidor e endpoint de solicitação de nova secret.
- Outbox transacional, worker separado com polling, retry, DLQ e replay manual.
- HMAC-SHA256, headers obrigatórios, timeout e at-least-once.
- Histórico das últimas 100 entregas por endpoint.
- Autenticação JWT nos endpoints normais e `ADMIN` no replay.

### Fora de escopo
- Webhooks recebidos pelo OMS vindos de clientes.
- E-mail automático ao cliente após falha nesta fase.
- Dashboard ou painel visual de entregas nesta fase.
- Exactly-once e ordering global.
- Redis/infraestrutura adicional nesta fase.
- Rate limiting de saída imediato.
- Arquivamento das linhas entregues nesta fase.
- Itens completos do pedido no payload.

## Requisitos funcionais
| ID | Requisito |
|---|---|
| PRD-FR-01 | Notificar clientes B2B quando o status de pedido mudar. |
| PRD-FR-02 | Permitir cadastro por `POST` com URL e lista de status. |
| PRD-FR-03 | Gerar a secret no servidor e devolvê-la na criação. |
| PRD-FR-04 | Permitir edição por `PATCH`, remoção por `DELETE` e listagem por `GET`. |
| PRD-FR-05 | Exigir `customer_id` no body ou path e não derivá-lo do JWT. |
| PRD-FR-06 | Entregar apenas eventos cujo status esteja selecionado pelo endpoint. |
| PRD-FR-07 | Mostrar histórico das últimas 100 entregas com status, payload, response e tempo. |
| PRD-FR-08 | Permitir replay manual de item da DLQ por administrador. |
| PRD-FR-09 | Gerar novo secret por endpoint por meio de endpoint dedicado. |
| PRD-FR-10 | Criar evento na outbox dentro da transação de mudança de status. |
| PRD-FR-11 | Enviar payload mínimo com `event_id`, tipo, timestamp, pedido, status, customer e total. |
| PRD-FR-12 | Enviar `X-Event-Id`, `X-Signature`, `X-Timestamp` e `X-Webhook-Id`. |
| PRD-FR-13 | Registrar quem realizou replay para auditoria. |
| PRD-FR-14 | Permitir deduplicação no cliente pelo `X-Event-Id`/`event_id`. |

## Requisitos não funcionais
- Outbound apenas; URLs devem usar HTTPS.
- Payload máximo de 64 KB.
- Timeout HTTP de 10 segundos por tentativa.
- Worker separado da API, com polling de 2 segundos e apenas um worker nesta fase.
- Cinco tentativas com backoff de 1 minuto, 5 minutos, 30 minutos, 2 horas e 12 horas.
- Entrega at-least-once, tolerando duplicatas.
- Secret antiga válida por 24 horas durante rotação.
- Logs estruturados sem expor secrets.
- Reuso de JWT, `requireRole`, Zod, Pino e middleware centralizado de erro.

## Decisões e trade-offs principais
- **Outbox no MySQL:** consistência transacional em troca de latência assíncrona e armazenamento adicional.
- **Um worker com polling:** simplicidade operacional em troca de ausência de scale-out imediato e de ordering global.
- **Cinco retries com DLQ:** recupera falhas transitórias em até cerca de 14 horas de agenda, mas exige operação manual após esgotamento.
- **At-least-once:** garante recuperação em troca de duplicidade.
- **Secret por endpoint:** isolamento de vazamento em troca de operação de rotação mais elaborada.

## Dependências
- Banco MySQL existente e Prisma.
- Máquina de estados e transação de `OrderService.changeStatus`.
- Middleware `authenticate` e `requireRole`.
- Middleware centralizado de erro e logger Pino.
- Schemas Zod existentes no padrão da API.

## Riscos e mitigação
| Risco | Probabilidade | Impacto | Mitigação |
|---|---|---|---|
| Vazamento de secret | Média | Alto | Secret por endpoint, redaction de logs e troca antiga válida somente 24 horas. |
| Falhas persistentes em endpoints | Média | Médio | DLQ, cinco tentativas, histórico e replay autorizado. |
| Latência acima de 10 segundos | Média | Médio | Polling de 2 segundos, timeout de 10 segundos e observabilidade de duração. |
| Duplicação de entregas | Média | Médio | `X-Event-Id` estável e instrução ao cliente para deduplicar. |
| Crescimento da outbox/DLQ | Média | Médio | Índices por estado/criação e política futura de arquivamento não definida nesta fase. |

## Critérios de aceitação
- O cadastro só aceita URL HTTPS e secret é gerada no servidor.
- Uma alteração elegível grava evento na mesma transação de status, histórico e estoque.
- O worker entrega status selecionados pelo endpoint e respeita filtros.
- O cliente recebe os headers definidos e a assinatura pode ser validada pelo corpo JSON.
- Falhas seguem o backoff definido e chegam à DLQ após cinco tentativas.
- Replay exige `ADMIN` e registra o usuário responsável.
- Todo erro esperado usa integração com `AppError`/middleware quando ocorre em endpoints da API.

## Estratégia de testes e validação
- Testes unitários de transições e publicação na outbox.
- Testes de integração do cadastro/edição/listagem/removal e da rota de replay.
- Testes de assinatura HMAC, secret por endpoint e rejeição de HTTP.
- Testes do worker com destinos 2xx, timeout, 4xx/5xx e backoff.
- Testes de retry/deduplicação e entrada na DLQ.
- Revisão de segurança com Sofia antes do deploy.
- Verificação em ambiente integrado da latência e das métricas, complementada pelos testes Vitest/Supertest existentes.
