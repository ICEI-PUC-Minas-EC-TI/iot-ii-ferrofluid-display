# 0002 — Plataforma de controle: Raspberry Pi 5 vs. Teensy 4.0

- **Status:** Gate de validação pendente — RPi5 é a opção principal em avaliação, Teensy 4.0 é o fallback condicional
- **Fase da timeline:** Fase 0 (definição do critério, `criterio_rpi_teensy`) e Fase 2 — Validação/Gate (`compra_rpi`, `fw_poc_intensidade`, `fw_medir_timing`, `gate_rpi_teensy`)
- **Decide:** qual hardware roda o firmware de controle principal do display
- **Depende de:** [ADR 0001](./0001-driver-eletroima-i2c-vs-spi.md) (rota de driver escolhida)

## Contexto

⚠️ **Correção de framing:** o roteiro de apresentação original do projeto (e o slide 13 do deck) descreve o microcontrolador como um **ESP32** ("sensores → ESP32 → drivers → eletroímãs/tanque"; "Wi-Fi nativo... suporta I2C e SPI nativamente"). Essa framing está **desatualizada**. A plataforma real em avaliação neste momento do projeto é **Raspberry Pi 5**, com **Teensy 4.0** como alternativa de fallback — não ESP32. O deck não foi corrigido ainda (ver [`README.md`](../README.md) e nota de status abaixo); essa correção é tarefa da Fase 9 (`doc_slides_update`), feita só ao final.

O motivo da dúvida entre RPi5 e Teensy é o mesmo motivo que torna a [decisão de driver](./0001-driver-eletroima-i2c-vs-spi.md) relevante aqui: o **Raspberry Pi 5 roda Linux, que não é um sistema operacional de tempo real (RTOS)**. Isso é irrelevante se o driver gerar PWM em hardware (rota I2C/PCA9685), mas é um risco real se o firmware precisar gerar BCM por software com timing fino (rota SPI/TPIC6B595) — nesse caso, o SO pode introduzir jitter que degrada a qualidade da intensidade exibida.

## Alternativas

- **Raspberry Pi 5 (8GB)** — opção principal, mais poder de processamento, mais fácil de integrar com o restante da stack (Wi-Fi, MQTT, cliente cloud), mas sem garantias de tempo real.
- **Teensy 4.0** — microcontrolador dedicado, sem SO, timing determinístico — elimina o risco de jitter, mas exige reescrever a integração de rede/cloud que seria mais natural num Linux completo.

## Desenho da decisão: um Gate de validação, não uma escolha a priori

Em vez de decidir entre as duas plataformas por análise teórica, o projeto trata isso como um **gate formal**: um critério objetivo é definido *antes* de qualquer teste (evitando viés de confirmação), e só depois o resultado medido é comparado contra esse critério.

1. **`criterio_rpi_teensy`** (Fase 0) — define o limite objetivo de estabilidade de timing que, se ultrapassado, força a migração do RPi5 para o Teensy 4.0. **O critério exato depende da rota de driver escolhida no [ADR 0001](./0001-driver-eletroima-i2c-vs-spi.md)**: mais rígido na rota SPI/BCM (sensível a jitter de CPU), bem mais tolerante na rota I2C/PCA9685 (PWM gerado em hardware pelo próprio chip, então o timing do SO importa muito menos).
2. **`compra_rpi`** (Fase 2) — compra do RPi5 8GB, unidade principal a ser validada antes de qualquer compromisso de design de PCB/firmware.
3. **`fw_poc_intensidade`** (Fase 2, depende da decisão do ADR 0001) — prova de conceito de geração de intensidade rodando no RPi5:
   - Se rota SPI: implementar BCM em C/C++ (via `spidev`/`pigpio`, usando DMA para estabilidade).
   - Se rota I2C: apenas usar a biblioteca I2C do PCA9685 — não há BCM a validar, o teste é mais leve.
4. **`fw_medir_timing`** (Fase 2) — mede a estabilidade/jitter real sob carga de CPU. Este é **o teste crítico do gate** na rota SPI; na rota I2C, o teste equivalente é mais sobre throughput/latência do barramento do que sobre timing fino.
5. **`gate_rpi_teensy`** (Fase 2, GATE) — compara o resultado medido contra o critério definido no passo 1. **Esta decisão trava o restante do design de firmware e PCB** — é por isso que ela acontece cedo (Fase 2, logo depois da decisão de driver), antes de qualquer compra ou design que dependa da plataforma final.

## Status atual

**Pendente.** O critério (`criterio_rpi_teensy`) só pode ser fechado depois da decisão de driver ([ADR 0001](./0001-driver-eletroima-i2c-vs-spi.md), hoje em aberto), e o teste em si (`fw_medir_timing`) só roda depois que o RPi5 for comprado e o PoC de intensidade implementado. Nenhuma medição foi feita ainda.

## Consequências / dependências

- Todo o design de firmware (Fase 5) e boa parte do design de PCB (Fase 4, dimensões/conectores) assume a plataforma final decidida aqui.
- Se o gate falhar (RPi5 não sustenta o timing exigido) e o projeto migrar para Teensy 4.0, a integração com AWS IoT Core / MQTT (Fase 7) precisa ser revisada — um Teensy não roda um SO completo com stack de rede tão pronta quanto o Linux do RPi5.

## Como atualizar este ADR quando o gate for resolvido

1. `Status` no topo para `Aceito` (RPi5) ou `Aceito com migração` (Teensy).
2. Adicionar seção `## Resultado do gate` com os números medidos (jitter/estabilidade) e a comparação com o critério definido.
3. Se migrar para Teensy, adicionar nota sobre o impacto na integração cloud (Fase 7) e linkar para o ADR que documentar essa adaptação, se necessário.
