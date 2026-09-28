# Timeline do projeto

Visão geral das 10 fases e ~98 tarefas que estruturam o projeto, do kickoff até a finalização. Referência da Semana 1: 24/09/2026.

O board Jira (`KAN`) espelha esta timeline 1:1 — ver [`jira.md`](./jira.md) para como cada conceito aqui (fase, semana, dependência) virou um objeto nativo do Jira.

## Fases

| Fase | Semanas | Foco |
|---|---|---|
| **0 — Kickoff & Setup** | 1 | Setup do repositório (monorepo + Claude Code) e pesquisa inicial de fornecedores. |
| **1 — Decisão de Driver** | 1-2 | Decidir entre a rota SPI (TPIC6B595) e a rota I2C (PCA9685+ULN2803) para os drivers dos eletroímãs. **Em aberto** — ver [ADR 0001](../decisions/0001-driver-eletroima-i2c-vs-spi.md). |
| **2 — Validação (Gate)** | 2-4 | Provar que o RPi5 sustenta a geração de intensidade da rota escolhida antes de comprometer PCB/firmware. Ver [ADR 0002](../decisions/0002-plataforma-rpi5-vs-teensy.md). |
| **3 — Arquitetura** | 4-5 | Fechar decisões de design (matriz, fonte, tanque, arquitetura de firmware) e disparar as compras principais. |
| **4 — Design de PCB** | 5-7 | Esquemático, layout e envio para fabricação da PCB, com o driver escolhido. |
| **5 — Firmware Core** | 6-9 | Implementação do firmware, em paralelo à fabricação da PCB. |
| **6 — Montagem Física** | 7-10 | Solda, impressão 3D, tanque, montagem da matriz completa. |
| **7 — Cloud & Frontend** | 7-10 | AWS IoT Core, DynamoDB, Amplify e app React, em paralelo ao hardware. |
| **8 — Integração** | 10-12 | Sistema completo funcionando de ponta a ponta, calibração e testes de falha. |
| **9 — Finalização** | 12-13 | Documentação, vídeo, apresentação e buffer de contingência. |

Categorias de tarefa: Setup, Decisão, Pesquisa, Compra, Hardware, Firmware, Cloud, Frontend, Integração, Documentação.

## Por que essa ordem

- A **Fase 1 (Decisão de Driver)** vem antes de qualquer compra de PCB ou implementação de firmware porque quase toda decisão posterior é condicional a ela (BCM em software só existe na rota SPI; a arquitetura de sensoriamento de diagnóstico muda com o chip escolhido).
- A **Fase 2 (Gate RPi5 vs. Teensy)** vem logo depois, e não antes, da decisão de driver, porque o critério do gate depende de qual rota foi escolhida (rota SPI é sensível a jitter; rota I2C é tolerante). Fazer esse gate antes seria testar a plataforma errada.
- **PCB (Fase 4) só é desenhada depois do protótipo elétrico validado** (Fases 1-3) — evitar refazer uma placa fabricada por causa de uma decisão de driver ainda não fechada.
- **Cloud & Frontend (Fase 7)** roda em paralelo ao hardware (Fases 4-6) porque não depende de nenhuma decisão de driver/plataforma — é a integração AWS que pode avançar independente do resto.

## Decisões e gates explícitos na timeline

- **Driver dos eletroímãs** (Fase 1, tarefa `driver_decisao`) — [ADR 0001](../decisions/0001-driver-eletroima-i2c-vs-spi.md). Em aberto.
- **Plataforma de controle** (Fase 2, tarefa `gate_rpi_teensy`) — [ADR 0002](../decisions/0002-plataforma-rpi5-vs-teensy.md). Gate pendente, depende do ADR 0001.
- **Contagem/proporção da matriz** (Fase 3, `matriz_contagem`) — faixa de trabalho definida (90-120 eletroímãs, ex. 9×10 ou 10×12); número exato fecha só após protótipo e orçamento real.
- **Topologia da fonte** (Fase 3, `fonte_topologia`) — fonte central de alta corrente vs. múltiplas fontes por bloco; decidido depois de fechar a contagem da matriz.
- **Tipo de tanque** (Fase 3, `tanque_tipo`) — vidro pré-cortado é a preferência declarada (acrílico manchou em testes de referência; corte manual de vidro é mais arriscado), mas a decisão formal ainda é uma tarefa distinta.
- **Atualização do deck de slides** (Fase 9, `doc_slides_update`) — deliberadamente adiada para o fim do projeto, para corrigir de uma vez as menções desatualizadas (driver "já escolhido", plataforma como ESP32) refletindo as decisões efetivamente tomadas.
- **Buffer de contingência** (Fase 9, `buffer_contingencia`) — margem reservada sem tarefas alocadas, para atrasos de qualquer fase anterior.

## Setup do repositório (Fase 0)

As duas primeiras tarefas do projeto (`T001`, `T002`) não são sobre o produto — são sobre como o time (humano + IA) vai trabalhar:

1. **Reestruturar o repositório como monorepo** (`repo_restructure`) — pastas separadas para firmware, frontend, infraestrutura/AWS, hardware/PCB e documentação, feito antes de qualquer código se acumular em qualquer parte.
2. **Configurar Claude Code no repositório** (`T002`) — incluindo este documento de contexto, transferindo o conhecimento levantado no planejamento (decisões técnicas, glossário, trade-offs) para que qualquer sessão futura de IA tenha contexto completo desde o início.

## Trabalho futuro (fora do escopo deste semestre)

- **Sistema de plugins**: interface de software padronizada para consumir diferentes fontes de dados (ex.: cotação de BTC, relógio, previsão do tempo) sem alterar o firmware base.
- Sensores opcionais fora do núcleo do POC: presença (PIR/ToF — display "acorda" quando alguém se aproxima) e microfone I2S (equalizador visual em tempo real).
