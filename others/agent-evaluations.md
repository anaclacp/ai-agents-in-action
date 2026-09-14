# AI Agent Evaluations

![Agent Evals](https://img.shields.io/badge/AI_AGENT-EVALUATIONS-6D28D9?style=for-the-badge)

## Visão geral

Uma avaliação, ou `eval`, executa uma tarefa contra um sistema de IA e aplica critérios para medir seu desempenho.

No caso de agentes, avaliar apenas a resposta final é insuficiente. Também é necessário observar:

* ferramentas selecionadas;
* argumentos enviados;
* ordem e qualidade das ações;
* resultados retornados pelas ferramentas;
* estado final do ambiente;
* qualidade da resposta comunicada ao usuário;
* consistência entre múltiplas execuções.

A Anthropic diferencia os seguintes componentes:

| Componente               | Definição                                                   |
| ------------------------ | ----------------------------------------------------------- |
| Task                     | Caso de teste com entrada e critérios de sucesso            |
| Trial                    | Uma tentativa de executar a task                            |
| Grader                   | Lógica que avalia uma parte do desempenho                   |
| Assertion                | Verificação individual dentro de um grader                  |
| Transcript ou trajectory | Registro completo da execução                               |
| Outcome                  | Estado final real do ambiente                               |
| Eval harness             | Infraestrutura responsável por executar e avaliar os testes |
| Agent harness            | Sistema que permite ao modelo atuar como agente             |

Fonte: [Anthropic, Demystifying evals for AI agents](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents)

## 1. Deterministic eval vs LLM-as-a-Judge

![Evaluators](https://img.shields.io/badge/1-EVALUATORS-6D28D9?style=for-the-badge)

### Deterministic eval

Utiliza código e regras objetivas.

Exemplos:

* `exact match`;
* regex;
* validação de JSON Schema;
* validação com Pydantic;
* unit tests;
* verificação de tool calls;
* comparação de argumentos;
* consulta ao banco de dados;
* verificação de estado;
* latência, custo, tokens e número de passos.

```python
assert call.name == "cancel_order"
assert call.args["order_id"] == "123"
assert database.order("123").status == "cancelled"
```

Características:

| Critério             | Deterministic eval                        |
| -------------------- | ----------------------------------------- |
| Custo                | Baixo                                     |
| Velocidade           | Alta                                      |
| Reprodutibilidade    | Alta                                      |
| Debug                | Simples                                   |
| Julgamento semântico | Limitado                                  |
| Melhor aplicação     | Comportamentos objetivamente verificáveis |

A Anthropic classifica verificações de tool calls, parâmetros, estado do ambiente, testes binários e análise do transcript como exemplos de graders baseados em código. Eles são rápidos, baratos, objetivos e reproduzíveis, mas podem ser rígidos diante de variações válidas.

Fonte: [Anthropic, Types of graders for agents](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents)

### LLM-as-a-Judge

Usa outro modelo para avaliar propriedades semânticas ou qualitativas.

Aplicações comuns:

* correctness semântico;
* groundedness;
* relevance;
* helpfulness;
* clareza;
* tom;
* coerência;
* completude;
* qualidade ou razoabilidade de uma trajectory.

```text
INPUT
+ RESPONSE
+ CONTEXT OU REFERENCE
+ RUBRIC
        |
        v
    LLM JUDGE
        |
        v
PASS / FAIL / SCORE / EXPLANATION
```

Características:

| Critério             | LLM-as-a-Judge                                   |
| -------------------- | ------------------------------------------------ |
| Custo                | Maior                                            |
| Velocidade           | Menor                                            |
| Reprodutibilidade    | Menor                                            |
| Flexibilidade        | Alta                                             |
| Julgamento semântico | Alto                                             |
| Melhor aplicação     | Critérios subjetivos ou semanticamente complexos |

Um judge é um instrumento de medição, não uma fonte automática de verdade. A Anthropic recomenda calibrar graders baseados em modelos contra avaliações humanas e usar rubricas claras e estruturadas.

### Reference-based e reference-free

#### Reference-based

Compara a resposta com uma referência conhecida:

```text
response <-> reference answer
```

Exemplo:

```text
Reference:
O prazo de devolução é de 30 dias.

Response:
O produto pode ser devolvido dentro de um mês.
```

Embora os textos sejam diferentes, um judge pode reconhecer equivalência semântica.

#### Reference-free

Não utiliza uma resposta esperada. O judge avalia a saída diretamente contra critérios definidos.

Exemplos:

* a resposta é relevante para a pergunta?
* as afirmações estão sustentadas pelo contexto?
* o tom respeita a política?
* a trajetória foi eficiente e segura?

Reference-based costuma ser mais apropriado para evals offline com golden datasets. Reference-free é útil no monitoramento de produção, onde normalmente não existe uma resposta esperada para cada solicitação.

Fonte: [LangSmith, Application-specific evaluation approaches](https://docs.langchain.com/langsmith/evaluation-approaches)

### Principais riscos do LLM-as-a-Judge

#### Position bias

A ordem das respostas pode influenciar uma comparação pairwise.

```text
Run 1: A vs B
Run 2: B vs A
```

Um estudo com mais de 150 mil avaliações e 15 judges encontrou position bias sistemático, variando conforme o judge, a tarefa e a diferença de qualidade entre as respostas.

Mitigação:

* inverter a ordem das respostas;
* repetir avaliações;
* medir consistência;
* tratar empates e divergências explicitamente.

Fonte: [Judging the Judges: A Systematic Study of Position Bias](https://aclanthology.org/2025.ijcnlp-long.18/)

#### Self-preference bias

O judge pode favorecer respostas próprias ou semelhantes à sua família de modelos. Um estudo com 20 LLMs encontrou que maior capacidade do modelo não implica menor self-preference bias.

Mitigação:

* separar generator e judge quando apropriado;
* decompor a avaliação em dimensões;
* utilizar múltiplos judges em casos críticos;
* validar decisões contra humanos.

Fonte: [Quantifying and Mitigating Self-Preference Bias of LLM Judges](https://arxiv.org/abs/2604.22891)

#### Score-range bias

A escala escolhida pode modificar a distribuição e até a avaliação relativa das respostas.

```text
Score from 1 to 5
```

pode produzir comportamento diferente de:

```text
Score from 1 to 10
```

Um trabalho publicado no ACL 2026 demonstrou sensibilidade dos judges à escala definida no prompt.

Mitigação:

* preferir rubricas discretas;
* evitar precisão numérica artificial;
* definir claramente o significado de cada nível.

Fonte: [Contrastive Decoding Mitigates Score Range Bias](https://aclanthology.org/2026.findings-acl.657/)

### Rubricas recomendadas

Evitar:

```text
Evaluate whether this is a good answer.
Score from 1 to 10.
```

Preferir dimensões separadas:

```yaml
correctness: PASS
relevance: PASS
groundedness: FAIL
instruction_following: PASS
```

Princípio central:

> Use o evaluator mais barato e determinístico que consiga medir o comportamento de forma confiável. Use LLM judges quando julgamento semântico for realmente necessário.

---

## 2. Correctness, relevance e groundedness

![RAG Metrics](https://img.shields.io/badge/2-RAG_METRICS-7C3AED?style=for-the-badge)

| Métrica             | Comparação          | Pergunta                                      |
| ------------------- | ------------------- | --------------------------------------------- |
| Correctness         | answer vs reference | A resposta está correta?                      |
| Relevance           | answer vs query     | A resposta atende à pergunta?                 |
| Groundedness        | answer vs context   | As afirmações são sustentadas pelo contexto?  |
| Retrieval relevance | context vs query    | O retriever encontrou informações relevantes? |

Fonte: [LangSmith, Evaluate a RAG application](https://docs.langchain.com/langsmith/evaluate-rag-tutorial)

### Correctness

Mede a resposta em relação a uma referência confiável.

```text
Question:
Qual é o prazo para devolução?

Reference:
30 dias.

Response:
O prazo é de 30 dias.

Correctness: PASS
```

Normalmente depende de:

* reference answer;
* expected label;
* valor conhecido no banco;
* golden dataset;
* resultado esperado.

Pode ser avaliada deterministicamente quando a saída é estruturada. Respostas textuais semanticamente equivalentes podem exigir um LLM judge.

### Relevance

Mede se a resposta atende diretamente à solicitação do usuário.

```text
Question:
Qual é o prazo para devolução?

Response:
Nossa empresa possui uma política flexível e foi fundada em 2012.

Relevance: FAIL
```

A resposta pode ser verdadeira e ainda assim irrelevante.

```text
Relevant != Correct
```

Uma resposta também pode ser relevante e incorreta:

```text
Question:
Qual é o prazo?

Response:
O prazo é de 15 dias.

Relevance: PASS
Correctness: FAIL
```

### Groundedness

Mede se as afirmações da resposta são sustentadas pelas evidências disponibilizadas ao modelo.

```text
Context:
O prazo de devolução é de 30 dias.

Response:
O prazo de devolução é de 30 dias.

Groundedness: PASS
```

Caso parcialmente sustentado:

```text
Context:
O prazo de devolução é de 30 dias.

Response:
O prazo é de 30 dias e o reembolso acontece em 48 horas.
```

Resultado:

```text
Prazo de 30 dias: supported
Reembolso em 48 horas: unsupported
Groundedness: FAIL
```

### Groundedness não garante correctness

```text
Retrieved context:
O prazo é de 15 dias.

Política real:
O prazo é de 30 dias.

Response:
O prazo é de 15 dias.
```

Resultado:

```text
Groundedness: PASS
Correctness: FAIL
```

O modelo representou corretamente uma fonte incorreta.

### Retrieval relevance

Avalia o retriever, não a resposta final.

```text
query <-> retrieved documents
```

Exemplo:

```text
Query:
Qual é a política de férias?

Retrieved:
1. Política de férias
2. Configuração do Outlook
3. Regras de folha de pagamento
```

Mesmo que o LLM produza uma resposta razoável, o retrieval apresenta baixa precisão e adiciona ruído ao contexto.

### Offline vs online

#### Offline evaluation

Pode utilizar:

* query;
* reference answer;
* retrieved context;
* response.

Métricas possíveis:

* correctness;
* relevance;
* groundedness;
* retrieval relevance.

#### Online evaluation

Normalmente possui:

* query;
* retrieved context;
* response.

Como não existe uma golden answer para cada interação de produção, é mais viável monitorar:

* relevance;
* groundedness;
* retrieval relevance;
* comportamento e custo;
* feedback do usuário.

---

## 3. Tool-call accuracy e agent evals

![Agent Evaluation](https://img.shields.io/badge/3-AGENT_EVALUATION-8B5CF6?style=for-the-badge)

A avaliação de tool calling deve ser decomposta em:

1. tool relevance;
2. tool selection;
3. argument accuracy;
4. trajectory;
5. outcome;
6. final response;
7. task success e consistência.

O BFCL avalia function calling em cenários simples, múltiplos, paralelos, multi-turn e em casos nos quais nenhuma função é adequada.

Fonte: [Berkeley Function Calling Leaderboard](https://gorilla.cs.berkeley.edu/leaderboard.html)

### 3.1 Tool relevance

Pergunta principal:

> Uma ferramenta deveria ter sido chamada?

```text
User:
Oi, tudo bem?

Agent:
get_customer(...)
```

Resultado:

```text
Tool relevance: FAIL
```

A ausência de tool call também é uma decisão válida. A eval deve conter exemplos positivos e negativos:

* casos nos quais o agente deve chamar uma tool;
* casos nos quais não deve chamar;
* casos nos quais deve pedir informações adicionais.

Isso evita `overtriggering`, quando o agente usa ferramentas excessivamente, e `undertriggering`, quando não as utiliza quando necessário.

A Anthropic recomenda conjuntos de testes balanceados exatamente para evitar otimizar apenas um lado do comportamento.

### 3.2 Tool selection

Pergunta principal:

> O agente selecionou a ferramenta correta?

```python
tools = [
    search_order,
    cancel_order,
    refund_order,
    update_address,
]
```

```text
User:
Cancele o pedido 123.
```

Esperado:

```text
search_order
cancel_order
```

Se o agente selecionar `refund_order`, a seleção falhou mesmo que o JSON seja válido.

```python
assert actual_call.name == expected_call.name
```

LangSmith classifica a seleção de uma ferramenta em um passo específico como `single-step evaluation`.

Fonte: [LangSmith, Evaluate a complex agent](https://docs.langchain.com/langsmith/evaluate-complex-agent)

### 3.3 Argument accuracy

Pergunta principal:

> Os parâmetros estão completos, válidos e semanticamente corretos?

```json
{
  "order_id": "124"
}
```

quando o usuário pediu:

```json
{
  "order_id": "123"
}
```

Resultado:

| Check                   | Resultado |
| ----------------------- | --------- |
| Tool correta            | PASS      |
| Schema válido           | PASS      |
| Argumentos obrigatórios | PASS      |
| Valores corretos        | FAIL      |

A avaliação deve separar:

* presença dos argumentos obrigatórios;
* tipos corretos;
* conformidade com o schema;
* valores extraídos da solicitação;
* restrições de negócio;
* argumentos não autorizados;
* dados inventados.

```python
CancelOrderArgs.model_validate(call.args)
assert call.args["order_id"] == "123"
```

Schema válido não significa argumento correto.

### 3.4 Trajectory evaluation

Trajectory é o registro do caminho executado pelo agente, incluindo mensagens, tool calls e resultados intermediários.

```text
search_order(123)
        |
        v
check_policy(123)
        |
        v
cancel_order(123)
```

A trajectory permite avaliar:

* ordem das ações;
* dependências;
* verificações obrigatórias;
* ações proibidas;
* excesso de chamadas;
* retries;
* fallback;
* eficiência;
* comportamento diante de falhas.

#### Modos determinísticos de trajectory matching

| Modo      | Comportamento                                                     |
| --------- | ----------------------------------------------------------------- |
| Strict    | Mesmas chamadas na mesma ordem                                    |
| Unordered | Mesmas chamadas, independentemente da ordem                       |
| Subset    | O agente não pode usar tools fora do conjunto permitido           |
| Superset  | As chamadas obrigatórias devem existir, mas extras são permitidas |

Fonte: [LangSmith, Trajectory evaluations](https://docs.langchain.com/langsmith/trajectory-evals)

#### Quando usar strict matching

Use quando a ordem representa uma regra obrigatória:

```text
verify_identity
        |
        v
confirm_transfer
        |
        v
transfer_money
```

Executar `transfer_money` antes da confirmação deve falhar, mesmo que o resultado final coincidentemente esteja correto.

#### Limitação de trajetórias rígidas

Podem existir vários caminhos corretos:

```text
Path A:
get_customer
get_orders
cancel_order
```

```text
Path B:
get_order_by_id
cancel_order
```

Se apenas `Path A` estiver registrado como referência, um strict match reprovará `Path B`, mesmo que ele seja válido.

A Anthropic relata um caso no `τ²-bench` em que um modelo encontrou uma solução válida e melhor para o usuário, mas falhou porque a avaliação esperava outro caminho. Por isso, a recomendação é avaliar principalmente o que o agente produziu e o estado que alcançou, restringindo a trajetória apenas quando a sequência for parte real da política ou da segurança.

Fonte: [Anthropic, Demystifying evals for AI agents](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents)

### 3.5 Outcome evaluation

Pergunta principal:

> O estado real do ambiente confirma que a tarefa foi concluída?

```text
Agent response:
Seu pedido foi cancelado.
```

Isso não prova que o cancelamento ocorreu.

A eval deve consultar o ambiente:

```sql
SELECT status
FROM orders
WHERE order_id = '123';
```

Esperado:

```text
status = cancelled
```

A Anthropic diferencia:

```text
Trajectory:
o que o agente fez

Outcome:
o que realmente mudou no ambiente
```

Exemplo de falha:

```text
search_order(123): SUCCESS
cancel_order(123): 500 Internal Server Error

Agent:
Pedido cancelado com sucesso.
```

Resultado:

| Dimensão                 | Resultado |
| ------------------------ | --------- |
| Tool selection           | PASS      |
| Arguments                | PASS      |
| Trajectory               | PASS      |
| Outcome                  | FAIL      |
| Groundedness da resposta | FAIL      |
| Task success             | FAIL      |

Portanto:

```text
tool-call accuracy != task success
```

### 3.6 Final response

Depois de validar ferramentas, trajetória e outcome, avalie a resposta enviada ao usuário.

Critérios:

* groundedness em relação aos tool results;
* relevance;
* clareza;
* instruction following;
* ausência de confirmação falsa;
* explicação adequada de falhas;
* próximos passos quando a ação não puder ser concluída.

```text
Tool result:
status = failed

Response:
Não consegui concluir o cancelamento. Posso tentar novamente ou encaminhar ao suporte.

Groundedness: PASS
```

---

## 4. Task completion, success rate e confiabilidade

![Task Success](https://img.shields.io/badge/4-TASK_SUCCESS-A855F7?style=for-the-badge)

### Task completion

Task success mede se o objetivo do usuário foi realmente alcançado.

```text
task_success_rate =
successful_trials / total_trials
```

Para uma transferência bancária:

```yaml
task_success:
  transaction_created: true
  amount_correct: true
  recipient_correct: true
  confirmation_collected: true
  unauthorized_actions: false
```

Pode ser:

* binário, quando todos os requisitos são obrigatórios;
* ponderado, quando critérios têm pesos diferentes;
* parcial, quando partes independentes da tarefa podem ser concluídas.

A Anthropic recomenda partial credit para tarefas multidimensionais. Um agente que identifica o problema e valida o cliente, mas falha no reembolso, teve desempenho melhor do que um agente que falhou antes de qualquer etapa.

### Múltiplos trials

Agentes são não determinísticos. Uma task pode passar em uma execução e falhar em outra. Por isso, um único trial não representa necessariamente a confiabilidade real.

### pass@k

Mede a probabilidade de pelo menos uma tentativa ter sucesso em `k` trials.

Sob uma hipótese simplificada de trials independentes com probabilidade de sucesso `p`:

```text
pass@k = 1 - (1 - p)^k
```

É útil quando várias tentativas são aceitáveis e basta uma solução funcionar.

Exemplo:

```text
geração de múltiplas soluções de código
```

### pass^k

Mede a probabilidade de todas as `k` tentativas terem sucesso.

```text
pass^k = p^k
```

Se o agente possui sucesso por trial de `75%`:

```text
pass^3 = 0.75^3
pass^3 = 42.2%
```

É especialmente relevante para agentes customer-facing, nos quais o comportamento precisa funcionar consistentemente.

Resumo:

| Métrica      | Pergunta                                 |
| ------------ | ---------------------------------------- |
| Success rate | Com que frequência a tarefa funciona?    |
| pass@k       | Pelo menos uma de k tentativas funciona? |
| pass^k       | Todas as k tentativas funcionam?         |

Fonte: [Anthropic, Non-determinism in agent evaluations](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents)

---

## 5. Estrutura recomendada para uma eval de agent

![Implementation](https://img.shields.io/badge/5-EVAL_DESIGN-9333EA?style=for-the-badge)

```yaml
task:
  id: cancel-order-001
  input:
    user_message: "Cancele o pedido 123"

  success_criteria:
    order_id: "123"
    final_status: "cancelled"

  graders:
    tool_relevance:
      expected_tool_usage: true

    tool_selection:
      required:
        - search_order
        - cancel_order

    arguments:
      cancel_order:
        order_id: "123"

    trajectory:
      requirements:
        - search_before_cancel
        - no_refund_tool
        - no_duplicate_cancellation
      max_tool_calls: 4

    outcome:
      database:
        orders:
          order_id: "123"
          status: "cancelled"

    response:
      relevance: true
      groundedness: true
      must_not_claim_success_on_failure: true

  tracked_metrics:
    - task_success
    - n_turns
    - n_tool_calls
    - latency
    - token_usage
    - estimated_cost
```

### Separação dos evaluators

| Dimensão                     | Evaluator recomendado |
| ---------------------------- | --------------------- |
| Tool chamada ou não          | Deterministic         |
| Tool selecionada             | Deterministic         |
| JSON Schema                  | Deterministic         |
| Argumentos                   | Deterministic         |
| Estado no banco              | Deterministic         |
| Ações proibidas              | Deterministic         |
| Ordem obrigatória            | Deterministic         |
| Eficiência de caminho aberto | LLM judge com rubric  |
| Clareza e tom                | LLM judge com rubric  |
| Groundedness semântico       | LLM judge             |
| Casos críticos e calibração  | Human evaluation      |

---

## 6. Processo recomendado pela Anthropic

![Anthropic](https://img.shields.io/badge/6-ANTHROPIC_GUIDANCE-581C87?style=for-the-badge)

### 1. Começar cedo

Um conjunto inicial de 20 a 50 tasks realistas já pode revelar melhorias grandes durante o desenvolvimento inicial.

### 2. Utilizar falhas reais

Transformar bugs, feedbacks e incidentes de produção em casos de regressão.

### 3. Escrever tasks sem ambiguidade

Dois especialistas de domínio deveriam chegar independentemente ao mesmo veredito de aprovação ou reprovação.

### 4. Criar uma reference solution

A referência comprova que:

* a tarefa é solucionável;
* os critérios estão corretos;
* o grader não está quebrado.

### 5. Balancear casos positivos e negativos

Testar:

* quando uma tool deve ser chamada;
* quando não deve;
* quando o agente deve pedir confirmação;
* quando deve recusar ou escalar.

### 6. Isolar os trials

Cada trial deve iniciar em um ambiente limpo. Estado compartilhado, cache, arquivos antigos ou recursos insuficientes podem contaminar os resultados.

### 7. Escolher graders de forma consciente

* deterministic onde possível;
* LLM judge onde necessário;
* humanos para calibração, auditoria e casos subjetivos importantes.

### 8. Evitar trajetórias desnecessariamente rígidas

Validar o outcome como prioridade. Restringir a trajectory quando a ordem estiver ligada a segurança, autorização, compliance ou política de negócio.

### 9. Conceder partial credit

Separar os componentes da tarefa para identificar exatamente onde ocorreu a falha.

### 10. Ler transcripts

A Anthropic afirma que não considera scores confiáveis até que alguém investigue a eval e leia amostras das trajectories. Isso ajuda a distinguir:

* falha real do agente;
* grader incorreto;
* tarefa ambígua;
* solução válida não prevista;
* limitação do harness;
* problema de infraestrutura.

### 11. Separar capability e regression evals

```text
Capability eval:
O que o agente consegue fazer atualmente?

Regression eval:
O agente continua executando corretamente o que já conseguia?
```

Capability evals precisam oferecer espaço para melhoria. Regression evals devem permanecer próximas de 100%.

### 12. Combinar camadas

Nenhuma camada detecta todos os problemas. Uma estratégia madura combina:

* automated evals;
* produção e observabilidade;
* feedback dos usuários;
* A/B tests;
* revisão manual de transcripts;
* avaliação humana periódica.

Fonte principal: [Anthropic, Demystifying evals for AI agents](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents)

---

## 7. Modelo mental final

```text
Can I verify it with code?
        |
       YES
        |
Deterministic evaluator
        |
tool, args, schema, state,
latency, cost, constraints


Can I verify it with code?
        |
        NO
        |
Does it require semantic judgment?
        |
       YES
        |
LLM-as-a-Judge
        |
clear rubric
separate dimensions
human calibration
```

Para agents:

```text
AGENT RUN
    |
    +-- Tool relevance
    +-- Tool selection
    +-- Argument accuracy
    +-- Trajectory
    +-- Outcome
    +-- Final response
    +-- Task success
    +-- Reliability across trials
```

## Referências

1. [Anthropic: Demystifying evals for AI agents](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents)
2. [LangSmith: Evaluation approaches](https://docs.langchain.com/langsmith/evaluation-approaches)
3. [LangSmith: Evaluate a RAG application](https://docs.langchain.com/langsmith/evaluate-rag-tutorial)
4. [LangSmith: Trajectory evaluations](https://docs.langchain.com/langsmith/trajectory-evals)
5. [Berkeley Function Calling Leaderboard](https://gorilla.cs.berkeley.edu/leaderboard.html)
6. [Judging the Judges: Position Bias](https://aclanthology.org/2025.ijcnlp-long.18/)
7. [Quantifying and Mitigating Self-Preference Bias](https://arxiv.org/abs/2604.22891)
8. [Contrastive Decoding Mitigates Score Range Bias](https://aclanthology.org/2026.findings-acl.657/)
