# Documentos do domínio

## Antes de explorar

Este workspace usa um contexto compartilhado:

- Leia `GLOSSARY.md` na raiz do workspace para os termos do domínio.
- Leia em `docs/adr/` as decisões relacionadas à área de trabalho.
- Quando uma tarefa indicar referências congeladas, use o glossário e as decisões desse conjunto para o escopo da tarefa.

Os repositórios dos componentes ficam sob esta raiz. Resolva os caminhos acima a partir do workspace, inclusive quando o diretório de execução for um componente.

Se um documento de domínio não existir, prossiga com os disponíveis. A skill `domain-modeling` cria glossários e ADRs quando termos ou decisões forem efetivamente definidos.

## Vocabulário e decisões

Use os termos definidos no glossário em tickets, testes, hipóteses e propostas. Quando faltar um conceito necessário, registre a lacuna para `domain-modeling`.

Se uma proposta contrariar uma ADR vigente, indique a ADR e explique o conflito antes de propor sua revisão.

## Organização

- `GLOSSARY.md`: vocabulário compartilhado já existente.
- `docs/adr/`: ADRs criadas conforme decisões futuras.

O setup mantém o glossário atual e não cria ADRs vazias nem um `GLOSSARY-MAP.md`.
