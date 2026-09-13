# Capítulo 1 - Entendendo Agentes, Assistentes e Padrões de LLM

> Anotações de leitura do livro **AI Agents in Action (2ª edição)**
> Período: 11/09/26 – 13/09/26

O livro é sobre construir e trabalhar com sistemas agênticos inteligentes. Não somente criar entidades autônomas, mas desenvolver agentes que conseguem ser eficientes e resolver problemas do mundo real.

---

## Legenda das anotações

| Marca | Tipo | Significado |
|---|---|---|
| 🟦 | **New** | algo que eu não sabia |
| 🟩 | **Trade-off** | decisão / prós + contras |
| 🟪 | **Failure** | como / porque pode falhar |
| 🟧 | **Pattern** | arquitetura / padrão útil |
| ⬜ | **Apply** | onde eu usaria isso |
| ❓ | **Question** | pesquisar / entender melhor |

---

## Índice

- [1.1 Agentes, assistentes e padrões de LLM](#11-agentes-assistentes-e-padrões-de-llm)
- [1.2 Introdução ao MCP](#12-introdução-ao-mcp)
- [1.3 As 5 camadas funcionais dos agentes](#13-as-5-camadas-funcionais-dos-agentes)
- [1.4 Sistemas multiagênticos](#14-sistemas-multiagênticos)
- [Takeaway do capítulo](#takeaway-do-capítulo)

---

## 1.1 Agentes, assistentes e padrões de LLM

### Assistente
Refere-se à LLM que usa ferramentas para responder o usuário, mas que **precisa de aprovação em cada etapa** e não tem autonomia para planejar e concluir uma tarefa sozinho.

### Agente
Consegue funcionar de forma **autônoma** para raciocinar, planejar e tomar decisões.
→ como enviar e-mails, por exemplo.

### 🟧 Pattern: Graduated HITL (Human-in-the-loop)

A autonomia é baseada no **nível da ação**, não no agente inteiro.

| Nível de risco | Comportamento |
|---|---|
| Low risk | autônomo |
| Medium risk | pode existir confirmação |
| High risk | aprovação explícita |

```
agent → decide comprar → ┌──────────────────┐   ← HITL
                         │ humano confirma  │
                         └────────┬─────────┘
                                  ↓
                            tool executa
```

Isso é **graduated HITL**.

### Autonomia

A ideia é que autonomia é a capacidade de **planejar e executar múltiplas etapas sem intervenção constante**.
↳ mas isso não quer dizer que é operado sem supervisão.

### 🟧 Pattern: Levels of autonomy

| Nível | Autonomia | Descrição | Exemplo |
|---|---|---|---|
| **Direct LLM** | zero | apenas gera texto | GPT, Claude no início ou uma chamada de API "crua" |
| **Tool augmented LLM** | baixa | o usuário provoca uma chamada específica de tool | GPT com DALL·E |
| **Assistant** | média | pode realizar ações, mas existe aprovação por tarefa | GitHub Copilot Chat |
| **Agent** | alta | usuário define o objetivo, o agente decide como chegar lá | Claude Code, Codex |

- **Assistant** → o humano controla a ação.
- **Agent** → o humano define o objetivo e o agent controla os passos.

### 🟧 Pattern: Tool chaining

O output de uma tool é o input / contexto de outra. Permite executar tarefas dependentes em sequência para atingir um objetivo.

> ⚠️ **tool chaining ≠ agent**
> Para ser agêntico, o sistema recebe o input do usuário e decide **em runtime** quais tools deverá usar.

### 🟦 New: Agency

Capacidade do agente de **decidir o que fazer** e **usar ferramentas** para atingir um objetivo, com algum grau de autonomia.

### 🟧 Pattern: SPAL (Sense → Plan → Act → Learn)

1. Processa o input;
2. Raciocina, decompõe o goal e decide tools / ações;
3. Executa o plano;
4. Avalia os outputs e atualiza a memória.

↳ o loop continua até atingir o objetivo.

### 🟦 New: built-in reasoning ≠ reasoning patterns desnecessários

Frontier models (modelos mais atualizados) já têm capacidade **nativa** de reasoning e planning.
↳ mesmo que os modelos já façam isso, às vezes vale colocar uma **estrutura externa** para organizar esse raciocínio.

### 🟦 New: Long-horizon

São goals longos (complexos) até serem completados — o objetivo inicial é distante da conclusão.

### 🟦 New: Out-of-distribution (OOD)

Quando o problema é muito específico e foge dos padrões do que o modelo foi treinado.

↳ Nesse caso, não adianta (muito) jogar um problema num frontier model e esperar que ele se vire.

**Solução:** dar a estrutura externa do processo.

1. Identifique o estado atual;
2. Levante hipóteses;
3. Determine evidências necessárias;
4. Use tools para verificá-las;
5. Descarte / confirme hipóteses.

… dessa forma, eu guio o modelo para raciocinar de forma "correta".

**Resumindo:** OOD é quando ocorre o risco do modelo generalizar demais e se perder, por não ter tido treinamento suficiente para lidar com certos problemas.

---

## 1.2 Introdução ao MCP

### 🟧 MCP server ≠ serviço público

Pode ser **local / privado / interno**, ou **remoto / público**.

→ O MCP padroniza **como o contexto é exposto e consumido**.
→ A interação é: o modelo recebe uma lista das tools disponíveis. O interessante é que o agent **não conhece nada da função**, apenas sua **interface** (params).

### 🟧 Por que MCP?

MCP cria um **protocolo padronizado** entre agents / LLMs e capabilities externas.

**Benefícios:**

1. **Standardization** — interface consistente;
2. **Decoupling** — tools separadas do código;
3. **Extensibility** — o server pode ser feito em outra linguagem;
4. **Abstração** — o cliente conhece o contrato.

### 🟩 Trade-off: MCP abstraction

**Ganhos:** padronização, desacoplamento, interoperabilidade, reutilização.
**Custos:** mais complexidade, potencial overhead de latência.

---

## 1.3 As 5 camadas funcionais dos agentes

As 5 primeiras camadas: **persona**, **tools / actions**, **reasoning / planning**, **knowledge & memory** e **evaluation / feedback**.

### 1. Persona
Frequentemente referenciada no system prompt.

### 2. Agent tools / actions
Ajudam os agentes a concluírem tasks.

### 3. Agent reasoning / planning

**Native reasoning vs. structured reasoning:**

| | Quando vale a pena |
|---|---|
| **Native** | casos de short horizon (1–3 passos), poucas tools, algo barato / reversível |
| **Structured** | long horizon / subgoals, muitas tools, ações caras, OOD → melhor para problemas complexos |

### 4. Single-path vs. multi-path reasoning

- **Single-path:** o agente se compromete com um caminho.
  ↳ frágil diante de incerteza.
- **Multi-path:** explora múltiplas estratégias e escolhe a mais promissora.
  ↳ 🟩 **trade-off:** mais tokens, mais latência, mais custo.

### 5. Knowledge & memory

- **Knowledge:** informação externa, relativamente estática e independente da conversa.
- **Memory:** informação dinâmica adquirida pela experiência / interação — histórico, preferências, decisões.

---

## 1.4 Sistemas multiagênticos

**Razões para ter:**

- **Specialization:** para cada task específica. Normalmente, ao invés de ter um agente que faz tudo, a tarefa é quebrada entre vários agentes.
- **Paralelismo:** ao invés de ter um sistema concorrente, é feito de uma vez, sem um depender do outro.
- **Context management:** para lidar com a janela de contexto. Ao distribuir o problema entre agentes, a chance de bater no limite diminui.
- **Inherently multi-agent problems:** problemas que exigem atores independentes para objetivos distintos.

> ⚠️ Multiagente deve resolver uma **limitação real** do single agent.

---

### 1.4.1 Agent Flow (assembly line)

#### 🟧 Pattern: Agent Flow
Cada agent é especializado, possui seus próprios tools e entrega o trabalho para o próximo.

Podem ser divididos em **3 formas**:

#### Thread
Todos os agentes leem / escrevem na mesma thread.
→ 🟩 **trade-off:** contexto completo, mas cresce rápido e gera ruído.

#### Blackboard
Vários agents compartilham um **workspace estruturado** → *shared structured state* entre agents.
↳ literalmente estruturado: cada agente retorna campos importantes.
↳ ou seja, cada agente tem acesso a **partes** do contexto, e não ao completo.

**Exemplo:**

| Agent | Reads | Writes |
|---|---|---|
| Research Agent | `goal`, `plan` | `research` |
| Risk Agent | `research` | `risks` |
| Writer Agent | `research`, `risks` | `final_report` |

↳ segue um contrato.
Usando essa arquitetura, é possível **desacoplar o sistema**.

```
goal → plan ─┬─→ researchAg. ──→ research ──┐
             │                              ├─→ blackboard → riskAgent
             └─→ dataAg. ──→ memória ───────┘
```

Dessa forma, dois agents trabalham **em paralelo**.

#### Conclusão sobre blackboard
🟧 É onde os agents publicam resultados intermediários e leem seletivamente apenas o contexto necessário.

🟩 **Trade-off:** mais controle e eficiência de contexto → mais complexidade de estado / coordenação.

→ A escolha correta depende de **quanto contexto o agente precisa**.

---

### 1.4.2 Agent Orchestration (hub-and-spoke)

→ Usado quando o single agent fica sobrecarregado ou precisa lidar com múltiplos objetivos.
↳ um **agente central** age como orquestrador, delegando tarefas para os agentes.

🟩 **Trade-off:** um drawback / desvantagem desse padrão é que os agents são bastante limitados.

**Fluxo do orquestrador:**

```
recebe o goal → planeja → quebra em subgoals → delega →
recebe resultados → decide próximos passos → determina quando terminou
```

Nesse pattern, os agents são como se fossem **workers** para o orchestrator usar.

**Quando funciona bem:** quando o goal pode ser decomposto em subgoals relativamente independentes e um agente central consegue coordená-los.

> Em outras palavras: quando **sabemos o que queremos** (goal), mas não necessariamente em runtime.

---

### 1.4.3 Agent Collaboration

Em contraponto ao orquestrador, pode existir **back-and-forth** entre os próprios agents.
↳ ou seja, há **feedback loops** entre agents, e isso resolve a limitação mencionada do orquestrador.

🟩 **Trade-off:**

**Ganhos**
- ↑ collaboration
- ↑ feedback / critique
- ↑ capacidade de resolver problemas complexos
- ↑ perspectivas diferentes

**Custos** — porém tudo isso tem preço:
- token cost;
- latência;
- repetição;
- complexidade operacional;
- dificuldade de controlar para parar.

→ Esse padrão é utilizado quando **não há nenhuma outra maneira de fazer**.

🟧 **Pattern: Collaboration**
> **Obs:** escolha o padrão **mais barato** que atenda aos requisitos do sistema.

---

## Takeaway do capítulo

Agents são **sistemas orientados a goals** que possuem **agency** para decidir, planejar e executar ações em **runtime**.

```
                        Agent
        ┌─────────────────┼─────────────────┐
        ↓                 ↓                 ↓
     Agency             Layers           Patterns
        ↓                 ↓                 ↓
      SPAL             Persona            Single
                        Tools          Assembly line
                      Reasoning        Hub-and-spoke
                      Knowledge         Collaboration
                      Evaluation
```
