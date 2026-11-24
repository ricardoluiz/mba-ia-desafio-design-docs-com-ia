# ADR-006 — Reuso dos padrões existentes do projeto

- **Status:** Aceita
- **Data:** 2026-11-24

## Contexto
A feature será implementada em uma aplicação Node.js/TypeScript existente com módulos Express, Prisma, Zod, Pino e tratamento centralizado de erros.

## Decisão
Implementar webhooks em `src/modules/webhooks/`, reutilizar `AppError`, `errorMiddleware`, Zod, Pino e `requireRole`, além de manter `changeStatus` como ponto crítico de integração. Os códigos específicos usarão o prefixo `WEBHOOK_`.

## Alternativas Consideradas
- **Criar um padrão paralelo de erro ou roteamento:** descartado por gerar inconsistência e retrabalho.
- **Implementar um framework próprio de workers:** descartado em favor de scripts Node/TypeScript simples nesta fase.

## Consequências
### Positivas
- Equipes usam contratos e observabilidade já conhecidos.
- Configuração, autenticação HTTP e testes podem seguir convenções existentes.

### Negativas
- A extensão do sistema exige cuidar do acoplamento entre `OrderService` e o módulo de webhooks.
- Novos modelos Prisma e migrações serão necessários.

**Trade-off:** o reuso reduz custo de integração, mas exige implementar a feature sem contornar as invariantes já centralizadas no domínio.