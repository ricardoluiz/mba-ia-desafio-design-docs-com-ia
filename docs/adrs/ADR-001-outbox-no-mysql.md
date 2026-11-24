# ADR-001 — Outbox no MySQL

- **Status:** Aceita
- **Data:** 2026-11-24

## Contexto
A atualização de status é crítica e ocorre dentro de uma transação que também altera estoque e grava histórico em `src/modules/orders/order.service.ts`. Chamadas HTTP síncronas nesse fluxo bloqueariam o cliente e poderiam falhar independentemente da transação. O banco já é MySQL via Prisma.

## Decisão
Usar o padrão Outbox no mesmo MySQL: a inserção do evento deve ocorrer na mesma transação de mudança de status, com payload renderizado em JSON no momento da criação da outbox.

## Alternativas Consideradas
- **Redis Streams/Cluster:** evitaria compartilhar MySQL, mas exigiria infraestrutura adicional e não estava disponível; a reunião descartou essa opção.
- **Trigger de banco:** foi descartado por acoplar notificação à camada de banco e dificultar controle da aplicação.
- **Chamada HTTP síncrona em `changeStatus`:** descartada porque bloquearia o fluxo e acoplaria disponibilidade do OMS ao cliente.

## Consequências
### Positivas
- Consistência: status, estoque, histórico e evento compartilham o rollback da transação.
- Nenhuma infraestrutura adicional é necessária nesta fase.

### Negativas
- O banco passa a armazenar eventos pendentes e exige indexação e possível arquivamento futuro.
- A entrega continua assíncrona e pode ter latência.

**Trade-off:** troca-se entrega imediata por consistência transacional e menor acoplamento; a latência mínima aceitável é compatível com o polling de 2 segundos.