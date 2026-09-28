# Glossário

Termos de domínio usados no resto da documentação técnica. Ordenados por categoria.

## Protocolos e técnicas

| Termo | Definição |
|---|---|
| **I2C** | Protocolo serial de 2 fios (SDA/SCL). Todos os dispositivos compartilham o mesmo barramento, identificados por endereço único. |
| **SPI** | Protocolo serial de 3-4 fios (dados, clock, seleção). Em modo *daisy-chain*, os dados atravessam cada chip em sequência, sem uso de endereços. |
| **SPI daisy-chain (cadeia)** | Topologia onde cada chip repassa dados ao próximo; a identidade do chip é sua posição física na fila, não um endereço configurado. |
| **PWM (Pulse Width Modulation)** | Técnica de simular intensidade variável alternando rapidamente ligado/desligado, variando a proporção de tempo ligado (*duty cycle*). |
| **BCM (Binary Code Modulation)** | Técnica de gerar PWM por software: atualiza um shift register várias vezes por ciclo, com durações ponderadas em potências de 2, sem hardware de PWM dedicado. Usada na rota SPI/TPIC6B595. |
| **DMA (Direct Memory Access)** | Transmite dados diretamente sem ocupar a CPU; usado para manter o timing do BCM estável. |
| **MQTT** | Protocolo leve de mensagens para telemetria/comandos entre dispositivo e nuvem. |
| **OTA (Over-The-Air)** | Atualização de firmware via Wi-Fi, sem conexão física USB. Neste projeto: `git pull` + restart de serviço `systemd`. |

## Componentes de hardware

| Termo | Definição |
|---|---|
| **Shift register** | Chip que recebe bits em série e os disponibiliza em saídas paralelas simultâneas; controla muitos pinos usando poucos fios do microcontrolador. |
| **Array de transistores Darlington** | Chip com múltiplas "chaves" eletrônicas internas que amplificam um sinal lógico fraco para a corrente necessária de uma carga (ex.: eletroímã). |
| **Catch diode (diodo de proteção / flyback)** | Absorve o pico de tensão gerado ao cortar abruptamente a corrente de um eletroímã, protegendo o transistor de controle. |
| **Coil driving circuitry** | Termo genérico para o conjunto de eletrônica que fornece corrente controlada aos eletroímãs (um ou vários chips). |
| **PCA9685** | Chip gerador de PWM via I2C, 16 canais, saída totem-pole. Gera PWM real em hardware. Usado no projeto de referência Fetch V2. Ver [ADR 0001](./decisions/0001-driver-eletroima-i2c-vs-spi.md). |
| **ULN2803** | Array de transistores Darlington; amplifica o sinal de PWM do PCA9685 para a corrente real do eletroímã. |
| **DRV8908** | Chip "tudo-em-um" (PWM + potência + diagnóstico) via SPI em cadeia; grau automotivo (-Q1), difícil de comprar avulso. **Descartado** — ver [ADR 0001](./decisions/0001-driver-eletroima-i2c-vs-spi.md). |
| **TPIC6B595** | Shift register de potência com diodo de proteção embutido; não gera PWM sozinho, precisa de BCM via firmware. |
| **INA219 / INA226** | Sensores de corrente (I2C), usados para detectar eletroímã queimado ou curto-circuito. |
| **DS18B20 / NTC** | Sensores de temperatura, usados para permitir desligamento de segurança por software. |

## Conceitos do projeto

| Termo | Definição |
|---|---|
| **Ferrofluido** | Fluido que reage a campo magnético formando picos visuais — a "tinta" física do display. |
| **Matriz de eletroímãs** | Grade de eletroímãs atrás de um tanque de vidro, formando os "pixels" do display; cada um precisa de controle individual e contínuo de intensidade. |
| **Gate (Fase de Validação)** | Ponto de decisão formal na timeline onde um resultado medido é comparado contra um critério pré-definido antes de travar design downstream (ex.: [gate RPi5 vs. Teensy](./decisions/0002-plataforma-rpi5-vs-teensy.md)). |
| **POC (Proof of Concept)** | Versão reduzida que valida a arquitetura, sem compromisso de escala/acabamento final. |
| **Monorepo** | Estrutura de repositório único com pastas separadas para firmware, frontend, infraestrutura/AWS, hardware/PCB e documentação. |
| **Fetch (Applied Procrastination)** | Projeto de referência principal. Teve V1 (falho) e V2 (funcional); orienta várias decisões técnicas deste projeto. |
| **beastie417** | Segunda referência técnica; reduziu o custo do projeto original em mais de 70%. |

## Documentação e processo (deste repositório)

| Termo | Definição |
|---|---|
| **ADR (Architecture Decision Record)** | Formato usado em [`decisions/`](./decisions) para registrar decisões técnicas: `Status` / `Contexto` / `Alternativas` / `Decisão` / `Consequências`. Uma decisão que muda gera atualização (ou, se for uma decisão realmente nova, um ADR numerado seguinte) — não se edita silenciosamente o histórico do debate. |
| **Fix Version (Jira)** | Campo nativo do Jira usado para representar a **Semana** da timeline (13 versões, "Semana 01"–"Semana 13"), pois o board Kanban do projeto não suporta Sprints. Ver [`project/jira.md`](./project/jira.md). |
| **Issue Link "Blocks" (Jira)** | Tipo de link usado para representar as **Dependências** da timeline como relação nativa e consultável entre issues. Ver [`project/jira.md`](./project/jira.md). |
