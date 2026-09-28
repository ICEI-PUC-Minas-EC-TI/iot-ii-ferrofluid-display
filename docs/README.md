# Documentação técnica — Display de Ferrofluido

Esta pasta é a fonte de verdade sobre **o que existe e o que foi decidido** no projeto. Não trata de como um agente de IA deve se comportar (isso vive em `.claude/`) — trata de arquitetura, decisões técnicas e como o projeto está organizado.

## Índice

- [`decisions/`](./decisions) — Architecture Decision Records (ADRs). Cada decisão técnica relevante tem um arquivo próprio, com contexto, alternativas, decisão e status. Uma decisão que muda **não edita o ADR antigo** — cria um novo ADR que a substitui e referencia o anterior. Isso preserva o histórico de raciocínio.
  - [`0001-driver-eletroima-i2c-vs-spi.md`](./decisions/0001-driver-eletroima-i2c-vs-spi.md) — **Em aberto.** Rota SPI (TPIC6B595) vs. rota I2C (PCA9685+ULN2803) para os drivers dos eletroímãs.
  - [`0002-plataforma-rpi5-vs-teensy.md`](./decisions/0002-plataforma-rpi5-vs-teensy.md) — **Gate pendente.** Raspberry Pi 5 vs. Teensy 4.0 como controlador principal.
- [`glossario.md`](./glossario.md) — termos de domínio (hardware, protocolos, referências de projeto) usados no resto da documentação.
- [`project/timeline.md`](./project/timeline.md) — as 10 fases e ~98 tarefas do projeto, com o racional de cada fase.
- [`project/jira.md`](./project/jira.md) — como o board Jira (projeto `KAN`) espelha essa timeline: epics, campos nativos usados no lugar de custom fields, e links de dependência.

## Como manter isso atualizado

Este conteúdo nasceu de uma migração manual do histórico de planejamento (chat) para o repositório. Daqui pra frente:

- Toda decisão técnica nova ou revisitada vira um novo arquivo em `decisions/`, numerado sequencialmente (`000N-titulo.md`).
- Quando uma decisão "em aberto" for finalmente tomada (ex.: `0001`, driver), o próprio ADR é atualizado: o campo `Status` muda para `Aceito`, e a seção "Decisão" registra a escolha final e a data — sem apagar o histórico do debate que já estava lá.
- O deck de slides (`proposta-display-ferrofluido.pptx`) ainda reflete decisões antigas/desatualizadas (driver "já escolhido" como SPI, plataforma como ESP32) — está fora do escopo desta pasta atualizá-lo automaticamente; isso é uma tarefa própria da timeline (Fase 9, `doc_slides_update`).
