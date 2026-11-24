# Processo de Produção da Documentação

## Sobre o desafio
Este desafio transformou a transcrição da reunião técnica e o código existente em um pacote documental para o Sistema de Webhooks de Notificação de Pedidos. A entrega separa requisitos de produto, proposta arquitetural, decisões pontuais, especificação de implementação e rastreabilidade das fontes.

A produção foi feita sem alterar `src/`, `prisma/`, `tests/` ou configurações da aplicação. O código foi usado somente como contexto e evidência dos padrões já existentes.

## Ferramentas de IA utilizadas
- **GitHub Copilot / MAI Code 2 Flash:** exploração da transcrição e do repositório, geração e revisão dos documentos Markdown.
- **Análise por subagentes:** extração independente das decisões da reunião e dos pontos de integração reais do código.
- **Prompts customizados de geração e auditoria:** conduziram a produção em camadas e a validação contra os critérios de aceite.

## Workflow adotado
1. Analisei `TRANSCRICAO.md` para separar decisões fechadas, requisitos, alternativas descartadas e pontos em aberto.
2. Mapeei os padrões reais de pedidos, Prisma, autenticação, erros e logging no código.
3. Produzi primeiro os seis ADRs, pois eles formalizam a base arquitetural.
4. Escrevi o RFC com a proposta, alternativas descartadas e questões abertas.
5. Escrevi o FDD com fluxos, modelos, contratos, erros e integração com arquivos existentes.
6. Consolidei o PRD em nível de produto.
7. Preenchi o tracker com IDs cruzados e fontes por timestamp ou caminho de arquivo.
8. Substituí o README pela documentação do processo e realizei uma revisão final.

## Prompts customizados
### Prompt 1 — Extração rastreável da transcrição
```text
Leia TRANSCRICAO.md integralmente e produza uma tabela separando decisões fechadas,
requisitos funcionais, requisitos não funcionais, alternativas descartadas, itens adiados,
itens fora de escopo e participantes. Para cada item, preserve o timestamp e o nome exato
do falante. Não infira requisitos que não estejam explícitos na reunião.
```

### Prompt 2 — FDD integrado ao código
```text
Gere o FDD desta feature a partir da transcrição e do código existente. Inclua fluxos de
outbox, worker, retry, DLQ e replay; pelo menos quatro endpoints com request, response e
status; matriz WEBHOOK_*; observabilidade; e uma seção obrigatória que cite pelo menos
quatro caminhos reais do repositório. Não use nomes de arquivos fictícios nem invente
integrações além dos padrões presentes no código.
```

### Prompt 3 — Auditoria contra critérios de aceite
```text
Audite PRD, RFC, FDD, ADRs, tracker e README contra a checklist fornecida. Para cada
critério, indique cumprido ou não cumprido, evidência no arquivo e correção objetiva.
Não aprovar o pacote enquanto houver lacuna formal, conteúdo sem origem rastreável ou
arquivo citado que não exista no repositório.
```

## Iterações e ajustes
A produção passou por três iterações principais:

1. **Estruturação inicial:** foram criados os documentos base e os ADRs. A revisão mostrou que ainda era necessário ampliar a rastreabilidade e verificar todos os contratos HTTP.
2. **Revisão dirigida:** os documentos foram corrigidos para respeitar a fronteira entre RFC e FDD, fechar metadados do RFC e detalhar os endpoints, payloads e códigos `WEBHOOK_*` do FDD.
3. **Rastreabilidade e fechamento:** o tracker foi ampliado com referências por timestamp e por arquivos reais; em seguida, o README foi substituído pelo relato deste processo.

Correções concretas incluíram remover afirmações sem origem, reforçar itens descartados/adiados, evitar detalhes de implementação redundantes no RFC e confirmar que os caminhos citados existem no workspace.

## Como navegar a entrega
- [`docs/PRD.md`](docs/PRD.md): visão de produto, público, escopo, métricas e critérios.
- [`docs/RFC.md`](docs/RFC.md): proposta arquitetural, alternativas e questões abertas.
- [`docs/adrs/`](docs/adrs/): decisões arquiteturais isoladas.
- [`docs/FDD.md`](docs/FDD.md): fluxos, contratos, erros, integração e observabilidade.
- [`docs/TRACKER.md`](docs/TRACKER.md): rastreabilidade entre documentos e fontes.

Ordem sugerida de leitura: PRD → RFC → ADRs → FDD → Tracker.

## Referência
Repositório base do desafio: [mba-ia-desafio-design-docs-com-ia](https://github.com/devfullcycle/mba-ia-desafio-design-docs-com-ia).
