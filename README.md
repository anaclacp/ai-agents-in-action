# 📓 Estudos sobre AI Agents e Engenharia de IA

Repositório com minhas anotações de estudo sobre **AI Agents, Responsible AI, avaliação de agentes, segurança, arquitetura e engenharia de sistemas de IA**.

Parte do conteúdo acompanha a leitura do livro [**AI Agents in Action (2ª ed.)**](https://www.oreilly.com/library/view/ai-agents-in/9781633434530), mas o repositório também reúne pesquisas independentes, documentação técnica, artigos, frameworks e conceitos que surgem durante meus estudos e no trabalho.

São notas pessoais escritas com minhas próprias palavras, incluindo:

* conceitos e definições;
* trade-offs;
* padrões de arquitetura;
* riscos e failure modes;
* dúvidas em aberto;
* conexões entre temas;
* ideias que quero aplicar em projetos reais.

O objetivo não é reproduzir nenhuma fonte, mas construir uma base de conhecimento própria, pesquisável e continuamente atualizada.

> ✍️ **Como estudo:** costumo ler livros, documentação e materiais técnicos, pesquisar pontos que quero aprofundar e anotar tudo à mão no meu caderno. Depois fotografo as páginas e uso IA para auxiliar na transcrição e organização em Markdown. Também reviso, complemento e reorganizo o conteúdo antes de publicar aqui.

---

## 📂 Estrutura

```text
.
├── README.md
├── chapters/
│   ├── chapter-1/
│   │   └── README.md
│   ├── chapter-2/
│   └── ...
└── others/
    ├── agent-evaluations.md
    ├── responsible-ai-risk-guardrails.md
    └── ...
```

A pasta `chapters/` contém as anotações diretamente relacionadas ao livro **AI Agents in Action**.

A pasta `others/` reúne estudos independentes e aprofundamentos sobre temas relacionados.

---

## 🏷️ Legenda das anotações

Uso um sistema de marcadores para separar o tipo de informação:

| Marca | Tipo          | Significado                 |
| ----- | ------------- | --------------------------- |
| 🟦    | **New**       | algo que eu não sabia       |
| 🟩    | **Trade-off** | decisão / prós + contras    |
| 🟪    | **Failure**   | como / por que pode falhar  |
| 🟧    | **Pattern**   | arquitetura / padrão útil   |
| ⬜     | **Apply**     | onde eu usaria isso         |
| ❓     | **Question**  | pesquisar / entender melhor |

---

## 📖 AI Agents in Action — Progresso

| Capítulo | Título                                           | Status      | Notas                                |
| -------- | ------------------------------------------------ | ----------- | ------------------------------------ |
| 1        | Entendendo Agentes, Assistentes e Padrões de LLM | ✅ Concluído | [`chapter-1`](./chapters/chapter-1/) |

---

## 🧠 Estudos complementares

Além do livro, o repositório inclui aprofundamentos em temas como:

* **Agent evaluations**
* **Responsible AI**
* **Risk Assessment**
* **Guardrails**
* **Human Oversight / HITL**
* **Prompt Injection**
* **Agent Security**
* **Least Privilege**
* **Excessive Agency**
* **NIST AI Risk Management Framework**
* **RAG e Groundedness**
* **Tool Calling**
* **Observabilidade e monitoramento de agentes**

---

## 🔑 Conceitos cobertos até aqui

### AI Agents

* **Agency, autonomia e níveis de autonomia**
* **SPAL** — Sense → Plan → Act → Learn
* **Tool chaining ≠ agent**
* **MCP**
* **Camadas funcionais:** persona, tools/actions, reasoning/planning, knowledge & memory, evaluation/feedback
* **Native vs. structured reasoning**
* **Single-path vs. multi-path reasoning**
* **Padrões multiagênticos:** Assembly Line, Blackboard, Hub-and-Spoke e Collaboration
* **Long-horizon tasks**
* **Out-of-distribution (OOD)**

### Evaluation

* deterministic evals;
* LLM-as-a-Judge;
* correctness;
* relevance;
* groundedness;
* retrieval relevance;
* tool-call accuracy;
* trajectory evaluation;
* outcome evaluation.

### Responsible AI e segurança

* Risk Assessment;
* likelihood e impact;
* inherent risk e residual risk;
* Guardrails;
* Human Oversight;
* HITL;
* Automation Bias;
* Least Privilege;
* Excessive Agency;
* Direct e Indirect Prompt Injection;
* Trust Boundaries;
* Defense in Depth;
* NIST AI RMF.

---

## 📚 Principais referências

### Livros

* [**AI Agents in Action — 2ª edição**](https://www.oreilly.com/library/view/ai-agents-in/9781633434530)

### Outras fontes

Também utilizo:

* documentação oficial;
* artigos técnicos;
* papers;
* frameworks de segurança e Responsible AI;
* materiais de empresas e organizações como NIST, OWASP, Anthropic, LangChain e outras fontes técnicas relevantes.

As referências específicas ficam documentadas dentro de cada anotação.
