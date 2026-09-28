# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Este arquivo trata apenas de **como o agente deve se comportar** neste repositório. As
definições técnicas (estrutura de pastas, comandos, o que cada coisa faz) vivem em
`docs/` — veja @../docs/ARCHITECTURE.md.

## Contexto que muda o comportamento esperado

- Este é o repositório de entrega de um trabalho acadêmico (IoT II, PUC Minas) e está,
  hoje, majoritariamente em estado de **template não preenchido**: os `README.md` de cada
  pasta e os arquivos em `Documentacao/` são instruções do template do curso para o aluno
  preencher, não documentação real do produto.
- Toda a documentação existente está em português. Responda e escreva novo conteúdo em
  português, no mesmo tom, salvo pedido explícito em contrário.

## Regras específicas deste repositório

- **Nunca invente conteúdo factual para preencher os placeholders acadêmicos** —
  nome do projeto, integrantes, orientador, referências ABNT, resultados de testes,
  conclusões, autores em `CITATION.cff`. Se o usuário pedir para "preencher a
  documentação", escreva a estrutura/rascunho pedido mas deixe marcado o que depende de
  informação real do usuário (nomes, dados de teste, fotos, vídeo) em vez de fabricar.
- `scripts/import_to_github_project.py` faz mutações reais via API GraphQL do GitHub
  (cria campos e draft issues num Project real). Ao sugerir ou rodar esse script, prefira
  `--dry-run` primeiro e nunca hardcode ou commite o `GITHUB_TOKEN`.
- Antes de assumir que uma pasta como `Codigo/`, `App/` ou `Banco de Dados/` já tem
  conteúdo real, confira — o template do curso cria essas pastas vazias por padrão, então
  "vazio" é o estado esperado, não um bug.
