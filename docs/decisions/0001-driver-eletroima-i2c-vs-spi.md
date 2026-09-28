# 0001 — Driver dos eletroímãs: rota SPI vs. rota I2C

- **Status:** Em aberto (reaberta) — não decidido
- **Fase da timeline:** Fase 1 — Decisão de Driver (tarefas `driver_pesquisa_spi`, `driver_pesquisa_i2c`, `driver_pesquisa_fornecedores`, `driver_prototype_compra`, `driver_decisao`)
- **Decide:** que arquitetura de driver controla os ~90–120 eletroímãs da matriz

## Contexto

O display precisa controlar de forma individual e contínua a intensidade de dezenas de eletroímãs (90–120, ver [timeline](../project/timeline.md), Fase 3). Isso exige um circuito de driver (*coil driving circuitry*) capaz de:

1. Amplificar um sinal lógico fraco do microcontrolador para a corrente real exigida por um eletroímã (~200mA cada).
2. Gerar intensidade variável, não só ligado/desligado — via PWM (Pulse Width Modulation).
3. Escalar para dezenas de canais sem explodir em custo, complexidade de fiação ou dificuldade de solda manual.

Três chips foram avaliados como núcleo do driver.

## Alternativas consideradas

### Opção A — I2C: PCA9685 + ULN2803

Reproduz a arquitetura validada pelo projeto de referência **Fetch V2**.

- **PCA9685**: chip gerador de PWM via I2C, 16 canais, saída totem-pole. Gera **PWM real em hardware** — não consome ciclos do microcontrolador nem depende de timing preciso em software.
- **ULN2803**: array de transistores Darlington que amplifica o sinal de PWM do PCA9685 para a corrente real do eletroímã.
- **Prós:** PWM em hardware elimina risco de jitter; amplamente disponível e barato.
- **Contras:** 2 chips por bloco de canais; I2C é um **barramento compartilhado** — todos os dispositivos nos mesmos 2 fios (SDA/SCL), identificados por endereço. A capacitância do barramento degrada com muitos dispositivos em paralelo, e a topologia não permite o estilo "PCB encadeada, soldada direto, sem conectores" que a rota SPI permite.

### Opção B — DRV8908 (descartada)

Um único chip por canal faria tudo: PWM + potência + diagnóstico de falhas, encadeado via SPI.

- **Descartada** por ser peça de **grau automotivo (-Q1)**, praticamente impossível de comprar em unidade avulsa no Brasil, e por vir em pacote TSSOP-24, inviável para solda manual por iniciantes.

### Opção C — SPI daisy-chain: TPIC6B595

- **TPIC6B595**: shift register de potência com diodo de proteção (catch/flyback diode) embutido. Um chip só por bloco: junta shift register + estágio de potência + proteção.
- **Prós:** não é peça automotiva, fácil de achar no Brasil, vem em pacote DIP (solda manual viável). Em modo *daisy-chain*, a identidade de cada chip é sua **posição física na cadeia**, não um endereço configurado — permite soldar PCBs em cadeia direto nos eletroímãs, sem conectores intermediários entre blocos.
- **Contras:** **não gera PWM em hardware.** Exige a técnica **BCM (Binary Code Modulation)** rodando em software no microcontrolador — atualizar o shift register várias vezes por ciclo, com durações ponderadas em potências de 2, usando timers/DMA para manter o timing estável. Isso introduz risco de **jitter**, principalmente relevante se o controlador rodar um SO que não é de tempo real (ver [ADR 0002](./0002-plataforma-rpi5-vs-teensy.md)).

## Decisão original (agora superada como premissa fixa)

No roteiro de apresentação inicial do projeto, a **Opção C (SPI / TPIC6B595)** foi tratada como escolhida ("Opção C — Escolhida", slide 11 do deck), com o trade-off resumido como: *"I2C é mais fácil de achar no Brasil; SPI evita degradação por muitos dispositivos no barramento."*

Essa decisão foi tomada cedo, sem dados reais de custo/disponibilidade de fornecedores nem teste comparativo prático — só com base nas características técnicas dos chips.

## Reabertura

Ao replanejar a timeline do projeto (ver [`project/timeline.md`](../project/timeline.md)), a decisão de driver deixou de ser uma premissa fixa herdada do roteiro e voltou a ser uma **decisão explícita**, numa Fase 1 dedicada, **antes** do gate de validação RPi5 vs. Teensy 4.0 — porque o escopo daquele teste muda dependendo da rota escolhida aqui (a rota SPI/BCM é sensível a jitter de CPU; a rota I2C/PCA9685 é tolerante, pois o PWM é gerado em hardware).

Toda tarefa downstream da timeline que antes assumia TPIC6B595 como fechado foi reescrita com linguagem condicional às duas rotas ("se SPI: ...; se I2C: ...").

## Critérios que vão pesar na decisão final

A tarefa `driver_decisao` define explicitamente os critérios a serem pesados:

- Viabilidade de solda manual (por quem vai montar a placa).
- Comportamento esperado sob a matriz final (90–120 eletroímãs) — risco de degradação de barramento (I2C) vs. exigência de timing (SPI/BCM).
- Necessidade de PWM em hardware (I2C) vs. software (SPI/BCM).
- Custo total real, com base em disponibilidade/preço confirmados no Brasil (não só características técnicas de datasheet).
- Peso estético/prático de "PCBs encadeadas sem conectores" (vantagem exclusiva da rota SPI) como critério de design.

Além disso, a pesquisa de fornecedores (`driver_pesquisa_fornecedores`) e a compra de amostras de ambas as rotas (`driver_prototype_compra`) devem embasar a decisão com dados reais antes da escolha final.

## Status atual

**Em aberto.** Confirmado explicitamente em 2026-09-28: a decisão ainda não foi tomada. O slide 11 do deck de apresentação segue desatualizado (mostra SPI/TPIC6B595 como "escolhida") e **não deve ser editado unilateralmente** — a atualização do deck é uma tarefa própria da Fase 9 (`doc_slides_update`), a ser feita só depois que a decisão real for tomada.

## Consequências / dependências

Decisões e tarefas que dependem desta:

- [ADR 0002](./0002-plataforma-rpi5-vs-teensy.md) — o critério de jitter do gate RPi5/Teensy depende da rota escolhida aqui.
- Design da PCB (Fase 4) — esquemático e layout mudam conforme o driver.
- Camada de driver do firmware (Fase 5) — implementação de BCM só existe na rota SPI.
- Compra final de componentes de driver (Fase 3/4).

## Como atualizar este ADR quando a decisão for tomada

Não criar um ADR novo para isso — esta é a continuação natural da mesma decisão. Atualizar:
1. `Status` no topo para `Aceito`.
2. Adicionar uma seção `## Decisão final` com a rota escolhida, data, e o resultado dos testes/pesquisa que embasaram a escolha.
3. Atualizar a seção "Consequências" acima se algo mudar no impacto downstream.
