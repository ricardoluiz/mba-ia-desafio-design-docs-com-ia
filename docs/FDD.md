# FDD — Sistema de Webhooks de Notificação de Pedidos

## Contexto e motivação técnica
O OMS atual atualiza status, estoque e histórico dentro de uma transação Prisma. Webhooks externos não devem participar dessa transação. O desenho usa outbox no MySQL, processamento separado e entrega HTTP resiliente.

## Objetivos técnicos
- Garantir consistência entre alteração de status e criação do evento.
- Entregar apenas eventos elegíveis ao endpoint/customer.
- Assinar entregas com HMAC-SHA256 e permitir deduplicação.
- Tratar retries sem bloquear a API.
- Reutilizar autenticação, erros, Zod, Pino e padrões modulares.

## Escopo e exclusões
Inclui modelos Prisma de endpoints, outbox, deliveries e DLQ; rotas de configuração; worker; replay; métricas e logs. Exclui webhook inbound, e-mail, dashboard, exactly-once, ordering global e scale-out com múltiplos workers nesta fase.

## Modelo de dados proposto
Os nomes abaixo são acionáveis para migração; os detalhes finais devem seguir convenções Prisma existentes.

- `WebhookEndpoint`: `id UUID`, `customerId`, `url`, `secretHash ou secretEncrypted`, `active`, `selectedStatuses`, `secretPrevious`, `secretPreviousExpiresAt`, timestamps.
- `WebhookOutbox`: `id UUID`, `orderId`, `customerId`, `eventType`, `payloadJson`, `statusFilter`, `attemptCount`, `nextAttemptAt`, `deliveredAt`, timestamps.
- `WebhookDelivery`: `id UUID`, `endpointId`, `outboxId`, `success`, `httpStatus`, `responseBody`, `durationMs`, timestamps.
- `WebhookDeadLetter`: `id UUID`, `outboxId`, `endpointId`, `payloadJson`, `failureReason`, `createdAt`, replay metadata opcional.

A secret não deve ser registrada em texto livre em logs nem retornada em operações subsequentes, exceto a criação e solicitação explícita de nova secret.

## Fluxos detalhados
### 1. Criação do evento na outbox
1. `OrderService.changeStatus` busca pedido e valida a transição.
2. Dentro do mesmo `prisma.$transaction`, atualiza estoque quando aplicável, atualiza status e cria histórico.
3. Após essas operações, chama `publishWebhookEvent(tx, refreshedOrder, fromStatus, toStatus)`.
4. A função renderiza o payload JSON, verifica filtros selecionados e cria uma entrada por endpoint elegível ou uma entrada de evento que será filtrada pelo worker. Qualquer falha na outbox lança erro e aborta a transação.
5. O registro usa UUID e payload snapshot; dados posteriores da order não alteram o payload já persistido.

### 2. Processamento pelo worker
1. O worker inicia com `src/worker.ts`, nova instância Prisma e `DATABASE_URL` compartilhada.
2. A cada 2 segundos consulta linhas pendentes cuja tentativa é elegível.
3. Para uma única execução nesta fase, seleciona a próxima linha por UUID/criação e processa seus endpoints elegíveis.
4. Monta headers e aplica timeout de 10 segundos.
5. Uma resposta 2xx é sucesso; 4xx/5xx, timeout e erros de rede são falhas.

### 3. Retry
O número máximo é cinco. Os atrasos são derivados da tentativa: 1 minuto, 5 minutos, 30 minutos, 2 horas e 12 horas. Cada entrega registra tentativa, status, response e duração. Excepcionalmente, uma falha deve entrar na DLQ após a quinta tentativa.

### 4. DLQ e replay
A DLQ contém payload, motivo da falha e timestamp. `POST /admin/webhooks/dead-letter/:id/replay` exige JWT `ADMIN`, cria nova execução do worker ou deixa a entrada elegível para processamento, registra auditoria e usa o mesmo `X-Event-Id`.

## Contratos públicos
Todos os endpoints normais exigem `Authorization: Bearer <jwt>`. `customer_id` vem do body ou path.

### POST /api/v1/webhooks
**Request**
```json
{
  "customer_id": "cus_123",
  "url": "https://cliente.example.com/oms/status",
  "statuses": ["PAID", "SHIPPED"]
}
```
**Response 201**
```json
{
  "id": "wh_123",
  "customer_id": "cus_123",
  "url": "https://cliente.example.com/oms/status",
  "statuses": ["PAID", "SHIPPED"],
  "active": true,
  "secret": "secret_gerada_no_servidor"
}
```
Erros: `400 WEBHOOK_INVALID_HTTPS_URL`, `400 WEBHOOK_PAYLOAD_TOO_LARGE`, `404 WEBHOOK_CUSTOMER_NOT_FOUND`.

### PATCH /api/v1/webhooks/:id
**Request**
```json
{"url":"https://cliente.example.com/novo","statuses":["DELIVERED"],"active":true}
```
**Response 200**
```json
{"id":"wh_123","url":"https://cliente.example.com/novo","statuses":["DELIVERED"],"active":true}
```
Erros: `404 WEBHOOK_NOT_FOUND`, `400 WEBHOOK_INVALID_HTTPS_URL`.

### DELETE /api/v1/webhooks/:id
**Response 204** sem body.
Erros: `404 WEBHOOK_NOT_FOUND`.

### GET /api/v1/customers/:customerId/webhooks
**Response 200**
```json
{"items":[{"id":"wh_123","url":"https://cliente.example.com/oms/status","statuses":["PAID","SHIPPED"],"active":true}]}
```

### GET /api/v1/webhooks/:id/deliveries
**Response 200**
```json
{"items":[{"id":"dlv_123","eventId":"evt_123","success":true,"httpStatus":200,"durationMs":180,"deliveredAt":"2026-11-24T12:00:00.000Z","payload":{"event_id":"evt_123","event_type":"order.status_changed","timestamp":"2026-11-24T12:00:00.000Z","order_id":"ord_123","order_number":"ORD-000123","from_status":"PENDING","to_status":"PAID","customer_id":"cus_123","total_cents":9990},"response":"ok"}]}
```

### POST /api/v1/admin/webhooks/dead-letter/:id/replay
**Request**
```json
{}
```
**Response 202**
```json
{"id":"dlq_123","status":"queued","replayedBy":"usr_123","replayedAt":"2026-11-24T12:00:00.000Z"}
```
Exige role `ADMIN`.

### POST /api/v1/webhooks/:id/rotate-secret
**Request**
```json
{}
```
**Response 200**
```json
{"secret":"nova_secret","previousSecretExpiresAt":"2026-11-25T12:00:00.000Z"}
```

## Payload e headers da entrega outbound
```json
{
  "event_id": "evt_uuid",
  "event_type": "order.status_changed",
  "timestamp": "2026-11-24T12:00:00.000Z",
  "order_id": "ord_uuid",
  "order_number": "ORD-000123",
  "from_status": "PENDING",
  "to_status": "PAID",
  "customer_id": "cus_uuid",
  "total_cents": 9990
}
```
Headers:
- `X-Event-Id: evt_uuid`
- `X-Webhook-Id: wh_uuid`
- `X-Signature: HMAC-SHA256(secret, raw-body-json)`
- `X-Timestamp: ISO 8601`
- `Content-Type: application/json`

Não incluir `items`. O receptor pode consultar `GET /api/v1/orders/:id` para detalhes.

## Matriz de erros previstos
| Código | HTTP | Condição |
|---|---|---|
| `WEBHOOK_INVALID_HTTPS_URL` | 400 | URL não usa HTTPS. |
| `WEBHOOK_PAYLOAD_TOO_LARGE` | 400 | Payload renderizado excede 64 KB. |
| `WEBHOOK_CUSTOMER_NOT_FOUND` | 404 | Customer não existe. |
| `WEBHOOK_NOT_FOUND` | 404 | Endpoint não existe. |
| `WEBHOOK_FORBIDDEN` | 403 | Usuário não possui papel requerido. |
| `WEBHOOK_SECRET_ROTATION_CONFLICT` | 409 | Rotação solicitada fora do estado permitido. |
| `WEBHOOK_DELIVERY_TIMEOUT` | 504/registro interno | Timeout de 10 segundos. |
| `WEBHOOK_DELIVERY_FAILED` | registro interno | Rede ou resposta não 2xx. |
| `WEBHOOK_DLQ_NOT_FOUND` | 404 | Item da DLQ não existe. |

## Estratégias de resiliência
- Timeout: 10 segundos por tentativa.
- Retries: cinco tentativas com backoff 1m, 5m, 30m, 2h, 12h.
- At-least-once: o receptor usa `X-Event-Id`/`event_id`.
- Circuito manual: ausência de retry indefinido e destino transferido à DLQ.
- Não usar fallback síncrono no `changeStatus`.
- Secret anterior aceita por 24 horas durante rotação.

## Observabilidade
- **Métricas:** filas pendentes, tentativas, entregas 2xx/falhas por endpoint, duração p50/p95, idade da outbox e itens DLQ.
- **Logs:** Pino com `eventId`, `webhookId`, `orderId`, `attempt`, `statusCode`, `durationMs` e `outcome`; redigir secrets, assinatura e conteúdo sensível.
- **Tracing:** propagar `X-Request-Id` nas operações HTTP da API e `traceId` entre outbox, worker e entrega. A implementação não adiciona um backend de tracing nesta fase.

## Integração com o sistema existente
1. `src/modules/orders/order.service.ts`: o método `changeStatus` é estendido para publicar evento após estoque, atualização e histórico, recebendo `tx` no helper `publishWebhookEvent(tx, order, fromStatus, toStatus)` e abortando tudo em falha da outbox.
2. `src/modules/orders/order.status.ts`: usa `canTransition` e os helpers de estoque existentes; o evento nasce somente após uma transição válida.
3. `src/modules/orders/order.routes.ts`: mantém JWT e validação Zod; o módulo de webhooks cria rotas próprias com `authenticate` e `requireRole` quando aplicável.
4. `src/app.ts`: `buildControllers` e `buildApp` podem registrar repositório, service e controller do módulo; o middleware global continua recebendo `AppError`.
5. `src/routes/index.ts`: monta `buildWebhookRouter` em `/api/v1/webhooks` e `/api/v1/admin/webhooks`.
6. `src/middlewares/auth.middleware.ts`: reutiliza `authenticate` e `requireRole('ADMIN')`.
7. `src/shared/errors/app-error.ts` e `src/middlewares/error.middleware.ts`: modelos de erro usam `AppError` e prefixo `WEBHOOK_`.
8. `src/shared/logger/index.ts`: worker e API usam o logger Pino singleton e seus campos estruturados.
9. `prisma/schema.prisma`: adiciona enums/modelos de endpoint, outbox, delivery e DLQ sem alterar os modelos de Order existentes.
10. `package.json`: adiciona `worker` e mantém Node 20/tsx.

## Dependências e compatibilidade
- Node.js >=20, TypeScript, Express, Prisma e MySQL existentes.
- PrismaClient separado no worker.
- Zod para configuração e validação.
- UUID da lib `uuid` já instalada.
- Nenhuma fila externa nesta fase.

## Critérios de aceite técnicos
- `changeStatus` cria outbox na mesma transação e faz rollback caso a criação falhe.
- IDs da outbox são UUID.
- Endpoints HTTP retornam JSON padronizado e códigos `WEBHOOK_*` previstos.
- Polling é iniciado apenas pelo processo separado.
- Timeout é 10 segundos e backoff segue a sequência definida.
- Replay exige `ADMIN` e registra ator.
- Métricas, logs e trace/correlation ID estão presentes sem secrets.

## Riscos e mitigação
- **Perda operacional por múltiplos workers:** usar apenas um worker nesta fase e deixar escala futura aberta.
- **Ordering global:** não prometido; ordering é por `order_id` enquanto há um worker, sem lock nessa fase.
- **Payload sensível:** snapshot limitado a campos definidos e sem itens.
- **Crescimento de dados:** índices por pendência/timestamp e política futura de arquivamento.
- **Reprocessamento:** replay mantém o mesmo event ID e o cliente deve deduplicar.
