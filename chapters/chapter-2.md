# Capítulo 2 - Core components: Large language models, prompting, and agents

> Anotações de leitura do capítulo 2 do livro **AI Agents in Action (2ª edição)**

---

## 2.1 Understanding large language models

### 2.1.1 LLMs: Probabilistic token machines

LLMs recebem tokens e produzem uma **distribuição de probabilidade** para o próximo token.

```text
input tokens
    ↓
   LLM
    ↓
probability distribution
    ↓
sampling
    ↓
next token
````

O modelo não escolhe diretamente uma palavra.
Primeiro ele gera probabilidades e depois um processo de **sampling** seleciona o token.

### 🟦 Backpropagation

Processo usado durante o treinamento para ajustar os **weights** do modelo a partir da loss.

```text
prediction
    ↓
  loss
    ↓
backpropagation
    ↓
gradients
    ↓
optimizer
    ↓
update weights
```

### 🟦 Gradient

Indica **como e quanto uma pequena mudança em um parâmetro afeta a loss**.

> Gradiente = direção e intensidade da variação da loss em relação aos parâmetros.

Um parâmetro pode ser, por exemplo, um weight `w`.

### Pretraining vs. Alignment

**Pretraining**

* aprende através de next-token prediction;
* constrói conhecimento e capacidades nos weights.

**Alignment**

* ocorre depois do treinamento base;
* ajusta o comportamento do modelo.

Exemplo: **RLHF (Reinforcement Learning from Human Feedback)**.

O objetivo é ensinar o modelo a:

* seguir instruções;
* responder de forma útil;
* evitar comportamentos indesejados.

> Alignment ≠ ensinar conhecimento novo.

Melhorias específicas de reasoning podem utilizar técnicas separadas, como:

* chain-of-thought fine-tuning;
* process reward models.

### Training vs. Inference

**Training**

```text
prediction → loss → backpropagation → update weights
```

Os parâmetros são alterados.

**Inference**

```text
prompt → forward pass → probability distribution → sampling → token
```

Os weights permanecem congelados.

O processo é autoregressivo:

```text
P(token₁ | context)
        ↓
token₁ entra no contexto
        ↓
P(token₂ | context + token₁)
        ↓
...
```

---

### 2.1.2 What is a token?

**Tokenization** é o processo de transformar texto em unidades menores que o modelo consegue processar.

```text
text
 ↓
tokenizer
 ↓
tokens / subtokens
 ↓
token IDs
 ↓
LLM
```

> Número de caracteres / palavras ≠ número de tokens.

### 🟩 Trade-off: JSON

JSON pode gerar **token overhead** por causa de:

* nomes de campos;
* aspas;
* `{}`;
* `:`;
* `,`;
* repetição de estrutura.

Mas oferece:

* estrutura;
* parsing previsível;
* validação;
* interoperabilidade.

> JSON pode custar mais tokens, mas reduzir complexidade de integração.

### 🟦 TOON

**Token-Oriented Object Notation** é um formato voltado a representar dados estruturados com menos overhead de tokens em alguns cenários.

Pode ser útil quando grandes volumes de dados estruturados precisam entrar no contexto do LLM.

> Não substitui JSON universalmente; é uma otimização de token efficiency.

### Tokens em sistemas agênticos

Em sistemas com múltiplas chamadas de LLM, o consumo de tokens pode crescer rapidamente.

```text
orchestrator
    ↓
worker A
    ↓
worker B
    ↓
critic
    ↓
orchestrator
```

Cada hop adiciona:

* input tokens;
* output tokens;
* custo;
* latência.

> Quanto mais contexto circula entre agentes, maior o custo e maior a chance de carregar informação irrelevante.

---

### 2.1.3 Tuning temperature, top-p, and more

Os parâmetros de geração modificam **como o token é escolhido a partir da distribuição de probabilidades**.

### Temperature

Controla o grau de aleatoriedade.

```text
low temperature  → distribuição mais concentrada → output mais consistente
high temperature → distribuição mais plana → output mais variado
```

🟩 **Trade-off:**

* baixa → consistência / previsibilidade;
* alta → criatividade / variedade.

> Temperature influencia o comportamento, mas não garante determinismo.

### Top-p

Também chamado de **nucleus sampling**.

Considera apenas o menor conjunto de tokens cuja probabilidade acumulada atinge `p`.

```text
tokens ordenados por probabilidade
        ↓
mantém apenas os que somam até p
        ↓
sampling dentro desse conjunto
```

→ reduz a chance de selecionar tokens da cauda de baixa probabilidade.

### Max tokens

Define o limite máximo de tokens gerados na resposta.

Útil para:

* controlar custo;
* evitar respostas excessivamente longas;
* servir como guardrail.

### Presence penalty

Penaliza tokens que **já apareceram**.

→ incentiva introdução de novos conteúdos / tópicos.

### Frequency penalty

Penaliza tokens proporcionalmente à **quantidade de vezes que já apareceram**.

→ reduz repetição excessiva.

### 🟩 Trade-off: Generation parameters

Esses parâmetros **influenciam**, mas não impõem comportamento de forma absoluta.

Na prática, os mais importantes tendem a ser:

* `temperature`;
* `max_tokens`.

O restante normalmente pode permanecer próximo do default, usando prompt engineering para controlar o comportamento.
