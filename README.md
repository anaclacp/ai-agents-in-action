# 📓 Anotações de AI Agents in Action (2ª edição)

Repositório com minhas anotações de leitura do livro [**AI Agents in Action (2ª ed.)**](https://www.oreilly.com/library/view/ai-agents-in/9781633434530).

São notas de estudo pessoais: resumo dos conceitos com minhas próprias palavras, trade-offs que achei relevantes, dúvidas em aberto e pontos que quero aplicar em projetos reais. Não substituem o livro. Além de ler o livro, estou forçando meu senso crítico: questionando decisões de arquitetura e mapeando onde cada padrão faz sentido.

> ✍️ **Como isso é feito:** leio o capítulo, pesquiso algumas coisas e anoto à mão no meu caderno. Depois fotografo as páginas e passo pro Claude transcrever e organizar em Markdown, pra ficar documentado e pesquisável aqui. O conteúdo é meu, a IA só estrutura.

---

## 📂 Estrutura

```
.
├── README.md
└── chapters/
    ├── chapter-1/
    │   └── README.md
    ├── chapter-2/
    └── ...
```

Cada capítulo vira uma pasta em `chapters/` com suas anotações.

---

## 🏷️ Legenda das anotações

Uso um sistema de marcadores para separar o tipo de informação:

| Marca | Tipo | Significado |
|---|---|---|
| 🟦 | **New** | algo que eu não sabia |
| 🟩 | **Trade-off** | decisão / prós + contras |
| 🟪 | **Failure** | como / porque pode falhar |
| 🟧 | **Pattern** | arquitetura / padrão útil |
| ⬜ | **Apply** | onde eu usaria isso |
| ❓ | **Question** | pesquisar / entender melhor |

---

## 📖 Progresso

| Capítulo | Título | Status | Notas |
|---|---|---|---|
| 1 | Entendendo Agentes, Assistentes e Padrões de LLM | ✅ Concluído | [`chapter-1`](./chapters/chapter-1/) |

---

## 🔑 Conceitos cobertos até aqui

- **Agency, autonomia e níveis de autonomia** (Direct LLM → Tool augmented → Assistant → Agent)
- **Graduated HITL** — autonomia por nível de risco da ação, não por agente
- **SPAL** (Sense → Plan → Act → Learn)
- **Tool chaining ≠ agent** — decisão em runtime é o que define agência
- **MCP** — protocolo padronizado entre agents/LLMs e capabilities externas
- **Camadas funcionais:** persona, tools/actions, reasoning/planning, knowledge & memory, evaluation/feedback
- **Native vs. structured reasoning** e **single-path vs. multi-path**
- **Padrões multiagênticos:** Assembly line (thread / blackboard), Hub-and-spoke, Collaboration
- **Long-horizon** e **out-of-distribution (OOD)**

---

## 📚 Referência

> [**AI Agents in Action**, 2ª edição](https://www.oreilly.com/library/view/ai-agents-in/9781633434530) — Manning Publications.
