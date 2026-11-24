# ADR-003 — HMAC-SHA256 com secret por endpoint

- **Status:** Aceita
- **Data:** 2026-11-24

## Contexto
Os clientes B2B precisam autenticar que uma entrega veio do OMS e impedir alteração do corpo durante o transporte. Cada endpoint pode pertencer a um customer e ter responsabilidades distintas.

## Decisão
Assinar o corpo JSON renderizado com HMAC-SHA256 usando uma secret distinta por endpoint. Enviar `X-Signature` e suportar rotação com a secret antiga válida por 24 horas. URLs HTTPS são obrigatórias no cadastro.

## Alternativas Consideradas
- **Secret global da plataforma:** descartada porque uma vazamento afetaria todos os clientes e impediria isolamento.
- **Assinatura sem timestamp:** não adotada porque a reunião definiu explicitamente `X-Timestamp` junto com `X-Signature`.

## Consequências
### Positivas
- Vazamentos ficam isolados por endpoint.
- HMAC permite verificação simples pelo receptor.

### Negativas
- Rotação exige suporte a duas secrets válidas temporariamente.
- A geração e armazenamento seguro da secret são responsabilidades adicionais.

**Trade-off:** o modelo reduz o impacto de compromissos, mas aumenta a complexidade de administração e rotação.