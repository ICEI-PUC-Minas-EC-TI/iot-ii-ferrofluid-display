# Arquitetura e Estrutura do Repositório

Definições técnicas do projeto. Este arquivo é a fonte de verdade sobre "o que existe" no
repositório; `.claude/CLAUDE.md` trata apenas de como o agente deve se comportar.

## O que é este projeto

Trabalho de "IoT II" (Engenharia de Computação, PUC Minas): um display cujos pixels são
feitos de ferrofluido, controlados por uma matriz de eletroímãs (em vez do LED/LCD
convencional). O repositório segue o template padrão de entrega de projetos da disciplina.

## Estado atual

O repositório está no estágio de **template ainda não preenchido**: quase todo o conteúdo
markdown (README de cada pasta, `Documentacao/*.md`, `Manual/manual de utilização.md`,
`CITATION.cff`) é texto instrucional/placeholder deixado pelo template do curso, indicando
o que deve ser colocado em cada lugar — não é documentação real do produto ainda.

A única automação funcional hoje é `scripts/import_to_github_project.py`.

## Estrutura de pastas

| Pasta | Propósito pretendido | Conteúdo atual |
|---|---|---|
| `Codigo/` | Código-fonte do Arduino/ESP (`.ino`) | vazio, só README placeholder |
| `App/` | Projeto do App Inventor (`.aia` + `.apk`) | vazio, só README placeholder |
| `Banco de Dados/` | Modelo ER, modelo lógico e físico do BD | vazio, só README placeholder |
| `Apresentacao/` | Vídeo de funcionamento e fotos do projeto | vazio, só README placeholder |
| `Manual/` | Manual de utilização (setup de hardware/software) | placeholder |
| `Documentacao/` | Relatório acadêmico: Introdução, Metodologias Ágeis, Desenvolvimento, Testes, Conclusão, Referências (numerados `01-`…`06-`) | seções com instruções do template, ainda não escritas |
| `scripts/` | Ferramentas auxiliares (não fazem parte da entrega acadêmica) | script real de importação para GitHub Projects |
| `CITATION.cff` | Metadados de citação do projeto (autores, orientador, título) | campos em branco |
| `README.md` | Índice/hub que linka para as pastas acima | preenchido parcialmente (nome do projeto ainda é `[NOME-DO-PROJETO]`) |

O `README.md` da raiz é o hub de navegação: cada seção do relatório linka para o arquivo
correspondente dentro de `Documentacao/`, e as demais seções linkam para os READMEs de
`Codigo/`, `App/`, `Apresentacao/` e `Manual/`.

## `scripts/` — importador de plano de projeto para o GitHub Projects

`scripts/import_to_github_project.py` lê `scripts/import_data.json` (uma lista de tarefas
do planejamento do projeto — campos `Title`, `Task ID`, `Fase`, `Semana`, `Categoria`,
`Status`, `Dependências`, `Descrição`) e cria/atualiza *draft items* em um GitHub Project
(v2) via API GraphQL, criando os campos customizados (`Task ID`, `Semana`, `Dependências`,
`Descrição` como texto; `Fase`, `Categoria` como single-select) que ainda não existirem no
board.

Setup e execução:

```bash
python3 -m venv .venv
.venv/bin/pip install requests

export GITHUB_TOKEN=ghp_xxx   # PAT clássico com escopo "project"

# simula sem escrever nada no GitHub
.venv/bin/python3 scripts/import_to_github_project.py --owner mello-felipe --number 5 --dry-run

# execução real
.venv/bin/python3 scripts/import_to_github_project.py --owner mello-felipe --number 5
```

Opções relevantes: `--owner` (dono do project, default `mello-felipe`), `--number` (número
do project, default `5`), `--owner-type` (`user` ou `org`, default `user`), `--data`
(caminho do JSON de entrada, default `import_data.json` — ao rodar a partir da raiz do
repo, use `--data scripts/import_data.json`), `--dry-run`.

Não há build, lint ou suíte de testes configurados neste repositório — é um repositório de
documentação de entrega acadêmica mais um script utilitário isolado, não uma aplicação com
pipeline de CI/CD.
