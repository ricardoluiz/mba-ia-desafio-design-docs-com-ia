# ADR-002 — Retry com backoff e DLQ

- **Status:** Aceita
- **Data:** 2026-11-24

## Contexto
Falhas temporárias ou permanentes de destinos externos precisam ser tratadas sem manter chamadas dentro da transação de status. A reunião definiu uma janela limitada de retry.

## Decisão
Executar até cinco tentativas, com backoff de 1 minuto, 5 minutos, 30 minutos, 2 horas e 12 horas. Após o esgotamento, persistir o evento em `webhook_dead_letter` com payload, motivo e timestamp, permitindo replay manual autenticado.

## Alternativas Consideradas
- **Retry indefinido:** descartado para evitar acúmulo permanente e reduzir carga.
- **Apenas três tentativas:** descartado porque encerraria eventos em cerca de 30 minutos, abaixo da janela operacional discutida.
- **Manter falhas apenas em `failed` na outbox:** alternativa rejeitada em favor de uma tabela DLQ separada.

## Consequências
### Positivas
- O sistema absorve falhas temporárias sem bloquear a API.
- Operadores podem investigar e repetir eventos permanentemente falhos.

### Negativas
- A janela total pode atingir cerca de 14 horas úteis de schedule.
- Falhas são reprocessadas de modo assíncrono e podem exigir replay.

**Trade-off:** limita-se a quantidade de trabalho assíncrono para evitar retry infinito, ao custo de intervenção manual após a DLQ.