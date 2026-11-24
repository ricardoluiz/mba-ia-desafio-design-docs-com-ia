# ADR-005 — Worker separado com polling

- **Status:** Aceita
- **Data:** 2026-11-24

## Contexto
O OMS precisa retirar eventos da outbox sem acoplar o processamento à API. O projeto não possui filas externas.

## Decisão
Criar um entry-point separado, como `src/worker.ts`, executado por `npm run worker`, usando uma nova instância de PrismaClient e polling a cada 2 segundos. Nesta fase será usado apenas um worker.

## Alternativas Consideradas
- **Worker no mesmo processo da API:** descartado para isolar falhas e permitir operação independente.
- **Trigger de banco:** descartado por acoplamento e complexidade de operação.
- **Redis Streams/Cluster:** descartado por adicionar infraestrutura.
- **Múltiplos workers agora:** adiado até haver necessidade de escala e coordenação.

## Consequências
### Positivas
- Reutiliza banco e stack Node/TypeScript existentes.
- O polling produz latência compatível com o objetivo de menos de 10 segundos.

### Negativas
- Consultas periódicas ao banco são necessárias.
- A solução não garante ordering global nem suporte seguro imediato a concorrência de vários workers.

**Trade-off:** simplicidade operacional é priorizada nesta fase, com escala horizontal deixada para evolução.