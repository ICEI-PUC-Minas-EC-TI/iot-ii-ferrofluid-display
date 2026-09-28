# Estrutura do board Jira (projeto KAN)

Como a [timeline](./timeline.md) foi importada e estruturada no Jira via Atlassian MCP, e por que os campos foram modelados dessa forma.

## Projeto

- **Chave:** `KAN` ("Ferrofluid Display")
- **Tipo:** team-managed (next-gen), board Kanban simples — **sem suporte a Sprints**.
- **Tipos de issue:** Epic, Tarefa, Subtask, História.
- **Hierarquia:** Epic → Tarefa via campo `parent` (não `Epic Link` — esse é o nome usado em projetos company-managed/clássicos; este projeto é team-managed).

## Mapeamento Epic/Tarefa

- 10 **Epics** (`KAN-4`–`KAN-13`), uma por fase da timeline (0–9), cada uma com uma descrição curta do foco da fase.
- 98 **Tarefas** (`KAN-14`–`KAN-111`), uma por tarefa da timeline, cada uma com `parent` = epic da sua fase.
- Fórmula de conversão de ID: `KAN-(N+13)`, onde N é o número da tarefa original (`T001` → `KAN-14`, `T098` → `KAN-111`) — decorre da ordem sequencial de criação (epics primeiro, depois tarefas).

## Por que campos nativos em vez de custom fields

A timeline original tem duas colunas que não existem por padrão no Jira: **Semana** e **Dependências**. A ideia inicial era criar *custom fields* para elas, mas o conector do Atlassian MCP usado nesta sessão não expõe nenhuma operação de administração de campos customizados (API de admin do Jira, fora do escopo do conector). Em vez de pedir para o usuário criar os campos manualmente na UI, optou-se por mapear os dois conceitos para **recursos nativos do Jira** que já existem e são consultáveis via JQL:

| Conceito da timeline | Campo nativo do Jira usado | Por quê |
|---|---|---|
| **Semana** | **Fix Version** ("Semana 01" – "Semana 13") | Sprint seria o mapeamento mais óbvio, mas o board é Kanban e **não aceita Sprints** (confirmado via erro `listJiraBoardSprints`: "O quadro não aceita sprints"). Fix Version é o próximo recurso nativo mais próximo — permite agrupar/filtrar por período sem precisar de custom field. |
| **Dependências** | **Issue Link, tipo `Blocks`** | Em vez de um campo de texto solto com IDs (`T012, T015`), cada dependência virou uma relação real entre issues: se a tarefa A depende da tarefa B, criou-se um link `B blocks A` (`inwardIssue = B`, `outwardIssue = A`). Isso é consultável via JQL, aparece no grafo de dependências do Jira, e bloqueia visualmente o board quando aplicável — muito mais útil que texto solto. |

O texto legível (`Task ID`, `Semana`, `Dependências`) permanece também na **descrição** de cada issue, como referência humana redundante — não foi removido.

### Resultado

- 13 versões criadas (`Semana 01`–`Semana 13`), todas no projeto `KAN`, não lançadas (`released: false`).
- Todas as 98 tarefas têm `fixVersions` atribuído, batendo exatamente com a semana original da timeline.
- **152 issue links** do tipo `Blocks` criados, um para cada aresta de dependência da timeline original.

## Notas operacionais (conector Atlassian MCP)

Registradas aqui porque são conhecimento útil para qualquer automação futura sobre este board, não só um detalhe de execução:

- `manageJiraProjectVersion` (criar versão) é uma operação **destrutiva** no conector — precisa ser chamada via `executeDestructive`, não `executeWrite`.
- `createJiraIssueLink` não é invocável como tool nomeada diretamente, mesmo aparecendo no catálogo — precisa ser chamada via `executeWrite` com `name: "createJiraIssueLink"`.
- O conector fez uma substituição automática de conteúdo em pelo menos um caso (texto "Claude Code" virou "Agente"/"Agemte" no título/descrição de uma issue na criação) — provavelmente uma redação automática de marca. Vale conferir o conteúdo de issues criadas via automação antes de considerá-las finais.
