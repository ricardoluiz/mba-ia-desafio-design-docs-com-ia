# Tracker de Rastreabilidade

| ID | Documento | Tipo | Conteúdo (resumo) | Fonte | Localização |
|---|---|---|---|---|---|
| PRD-FR-01 | docs/PRD.md | Requisito Funcional | Notificar clientes B2B em mudanças de status | TRANSCRICAO | `[09:00] Marcos` |
| PRD-FR-02 | docs/PRD.md | Requisito Funcional | POST cadastra URL e status desejados | TRANSCRICAO | `[09:31] Marcos` |
| PRD-FR-03 | docs/PRD.md | Requisito Funcional | Secret gerada no servidor na criação | TRANSCRICAO | `[09:31] Marcos` |
| PRD-FR-04 | docs/PRD.md | Requisitos Funcionais | PATCH, DELETE e GET de webhooks | TRANSCRICAO | `[09:33] Bruno` |
| PRD-FR-05 | docs/PRD.md | Restrição | customer_id vem do body/path, não do JWT | TRANSCRICAO | `[09:33] Larissa` |
| PRD-FR-06 | docs/PRD.md | Requisito Funcional | Filtrar entregas pelos status selecionados | TRANSCRICAO | `[09:34] Marcos` |
| PRD-FR-07 | docs/PRD.md | Requisito Funcional | Histórico das últimas 100 entregas | TRANSCRICAO | `[09:34] Marcos` |
| PRD-FR-08 | docs/PRD.md | Requisito Funcional | Replay manual da DLQ por administrador | TRANSCRICAO | `[09:35] Diego` |
| PRD-FR-09 | docs/PRD.md | Requisito Funcional | Endpoint para solicitar nova secret | TRANSCRICAO | `[09:21] Sofia` |
| PRD-FR-10 | docs/PRD.md | Requisito Funcional | Outbox na transação de mudança de status | TRANSCRICAO | `[09:40] Bruno` |
| PRD-FR-11 | docs/PRD.md | Requisito Funcional | Payload mínimo com evento, pedido, status e total | TRANSCRICAO | `[09:43] Diego` |
| PRD-FR-12 | docs/PRD.md | Requisito Funcional | Enviar os quatro headers definidos | TRANSCRICAO | `[09:44] Diego` |
| PRD-FR-13 | docs/PRD.md | Requisito Não Funcional | Auditar usuário responsável pelo replay | TRANSCRICAO | `[09:36] Sofia` |
| PRD-FR-14 | docs/PRD.md | Requisito Funcional | Cliente deduplica pelo event_id | TRANSCRICAO | `[09:25] Diego` |
| PRD-NFR-01 | docs/PRD.md | Métrica de sucesso | Entrega em menos de 10 segundos | TRANSCRICAO | `[09:02] Marcos` |
| PRD-NFR-02 | docs/PRD.md | Restrição | Payload máximo de 64 KB | TRANSCRICAO | `[09:24] Larissa` |
| PRD-NFR-03 | docs/PRD.md | Restrição | Timeout HTTP de 10 segundos | TRANSCRICAO | `[09:42] Diego` |
| PRD-NFR-04 | docs/PRD.md | Requisito Não Funcional | Secret antiga válida por 24 horas | TRANSCRICAO | `[09:21] Sofia` |
| PRD-OUT-01 | docs/PRD.md | Fora de escopo | Não recebe webhooks de clientes | TRANSCRICAO | `[09:02] Marcos` |
| PRD-OUT-02 | docs/PRD.md | Fora de escopo | E-mail não entra nesta fase | TRANSCRICAO | `[09:38] Larissa` |
| PRD-OUT-03 | docs/PRD.md | Fora de escopo | Dashboard/painel visual fica fora | TRANSCRICAO | `[09:39] Marcos` |
| PRD-OUT-04 | docs/PRD.md | Fora de escopo | Exactly-once e itens completos do payload não entram | TRANSCRICAO | `[09:25] Diego` |
| PRD-RISK-01 | docs/PRD.md | Risco | Vazamento de secret isolado por endpoint | TRANSCRICAO | `[09:22] Diego` |
| PRD-RISK-02 | docs/PRD.md | Risco | Falhas permanentes tratadas via DLQ | TRANSCRICAO | `[09:18] Diego` |
| RFC-TLDR-01 | docs/RFC.md | Decisão | Outbox no MySQL com worker separado | TRANSCRICAO | `[09:06] Diego` |
| RFC-ALT-01 | docs/RFC.md | Trade-off | HTTP síncrono em changeStatus descartado | TRANSCRICAO | `[09:04] Larissa` |
| RFC-ALT-02 | docs/RFC.md | Trade-off | Redis Streams/Cluster descartado nesta fase | TRANSCRICAO | `[09:07] Diego` |
| RFC-ALT-03 | docs/RFC.md | Trade-off | Exactly-once descartado em favor de at-least-once | TRANSCRICAO | `[09:25] Diego` |
| RFC-OPEN-01 | docs/RFC.md | Questão em aberto | Rate limiting de saída será decidido depois de observação | TRANSCRICAO | `[09:39] Diego` |
| RFC-OPEN-02 | docs/RFC.md | Questão em aberto | Arquivamento da outbox será decidido depois | TRANSCRICAO | `[09:08] Diego` |
| RFC-OPEN-03 | docs/RFC.md | Questão em aberto | Múltiplos workers ficam para escala futura | TRANSCRICAO | `[09:13] Diego` |
| ADR-001 | docs/adrs/ADR-001-outbox-no-mysql.md | Decisão | Outbox no MySQL na mesma transação | TRANSCRICAO | `[09:06] Diego` |
| ADR-002 | docs/adrs/ADR-002-retry-backoff-e-dlq.md | Decisão | Cinco retries, backoff e DLQ | TRANSCRICAO | `[09:16] Larissa` |
| ADR-003 | docs/adrs/ADR-003-hmac-secret-por-endpoint.md | Decisão | HMAC-SHA256 e secret por endpoint | TRANSCRICAO | `[09:19] Sofia` |
| ADR-004 | docs/adrs/ADR-004-at-least-once-e-event-id.md | Decisão | At-least-once com X-Event-Id | TRANSCRICAO | `[09:24] Diego` |
| ADR-005 | docs/adrs/ADR-005-worker-processo-separado-polling.md | Decisão | Worker separado executado por polling a 2s | TRANSCRICAO | `[09:11] Larissa` |
| ADR-006 | docs/adrs/ADR-006-reuso-padroes-existentes.md | Decisão | Reusar padrões do projeto | TRANSCRICAO | `[09:29] Larissa` |
| FDD-FLOW-01 | docs/FDD.md | Fluxo Técnico | Publicar evento dentro da transação de changeStatus | TRANSCRICAO | `[09:40] Bruno` |
| FDD-FLOW-02 | docs/FDD.md | Fluxo Técnico | Worker faz polling separado a cada 2 segundos | TRANSCRICAO | `[09:09] Diego` |
| FDD-FLOW-03 | docs/FDD.md | Fluxo Técnico | Replay manual da DLQ por endpoint administrativo | TRANSCRICAO | `[09:18] Diego` |
| FDD-CONTRATO-01 | docs/FDD.md | Contrato HTTP | POST /api/v1/webhooks e status 201 | TRANSCRICAO | `[09:31] Marcos` |
| FDD-CONTRATO-02 | docs/FDD.md | Contrato HTTP | PATCH /api/v1/webhooks/:id | TRANSCRICAO | `[09:33] Bruno` |
| FDD-CONTRATO-03 | docs/FDD.md | Contrato HTTP | DELETE /api/v1/webhooks/:id e 204 | TRANSCRICAO | `[09:33] Bruno` |
| FDD-CONTRATO-04 | docs/FDD.md | Contrato HTTP | GET entregas e histórico das últimas 100 | TRANSCRICAO | `[09:34] Marcos` |
| FDD-CONTRATO-05 | docs/FDD.md | Contrato HTTP | Replay da DLQ com papel ADMIN | TRANSCRICAO | `[09:36] Sofia` |
| FDD-PAYLOAD-01 | docs/FDD.md | Contrato de Payload | Campos mínimos e event_type fixo | TRANSCRICAO | `[09:43] Diego` |
| FDD-ERR-01 | docs/FDD.md | Matriz de Erros | Prefixo WEBHOOK_ nos códigos do módulo | TRANSCRICAO | `[09:29] Bruno` |
| FDD-RES-01 | docs/FDD.md | Resiliência | Backoff de 1m, 5m, 30m, 2h e 12h | TRANSCRICAO | `[09:17] Larissa` |
| FDD-CODE-01 | docs/FDD.md | Integração | changeStatus usa a transação e valida transições | CODIGO | `src/modules/orders/order.service.ts` |
| FDD-CODE-02 | docs/FDD.md | Integração | Máquina de estados e regras de estoque | CODIGO | `src/modules/orders/order.status.ts` |
| FDD-CODE-03 | docs/FDD.md | Integração | Rotas aplicam autenticação JWT | CODIGO | `src/modules/orders/order.routes.ts` |
| FDD-CODE-04 | docs/FDD.md | Integração | App monta controllers e middleware global | CODIGO | `src/app.ts` |
| FDD-CODE-05 | docs/FDD.md | Integração | buildApiRouter compõe módulos e rotas | CODIGO | `src/routes/index.ts` |
| FDD-CODE-06 | docs/FDD.md | Integração | authenticate e requireRole existem | CODIGO | `src/middlewares/auth.middleware.ts` |
| FDD-CODE-07 | docs/FDD.md | Integração | AppError e middleware centralizado tratam códigos | CODIGO | `src/shared/errors/app-error.ts` |
| FDD-CODE-08 | docs/FDD.md | Integração | Pino fornece logger estruturado | CODIGO | `src/shared/logger/index.ts` |
| FDD-CODE-09 | docs/FDD.md | Integração | Schema Prisma contém modelos/domínio atual | CODIGO | `prisma/schema.prisma` |
| FDD-CODE-10 | docs/FDD.md | Integração | Testes Vitest/Supertest seguem padrões atuais | CODIGO | `tests/orders.test.ts` |
