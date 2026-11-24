# ADR-004 — At-least-once com X-Event-Id

- **Status:** Aceita
- **Data:** 2026-11-24

## Contexto
Timeouts, falhas de rede e retries podem fazer o destino receber uma entrega mais de uma vez. Exactly-once foi considerado inviável nesta integração.

## Decisão
Adotar entrega at-least-once e usar `X-Event-Id` contendo o UUID da outbox/evento. O cliente deve armazenar IDs recebidos e ignorar duplicidades.

## Alternativas Consideradas
- **Exactly-once:** descartado como custo e coordenação não compatíveis com o desenho discutido.
- **Deslocar coordenação ao serviço receptor sem ID:** descartado porque não permitiria deduplicação estável.

## Consequências
### Positivas
- Retries recuperam entregas perdidas.
- O receptor possui chave estável para deduplicação.

### Negativas
- Pode haver duplicidades temporárias.
- Reprocessamento manual mantém o mesmo ID lógico do evento.

**Trade-off:** aceita-se duplicidade controlada em troca de maior garantia de entrega.