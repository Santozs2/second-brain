---
type: concept
area: Conceitos
status: estavel
difficulty: advanced
id: ia-rag
category: Inteligência Artificial
tags:
  - ia
  - llm
  - concept
created: 2026-08-24
updated: 2026-08-24
---
# 📚 RAG — Geração Aumentada por Recuperação

> Buscar a informação relevante e entregá-la ao modelo junto da pergunta. O modelo deixa de responder "de memória" e passa a responder "com a fonte na mão".

---

## 📖 O padrão

**Retrieval-Augmented Generation**, formalizado em *Lewis et al. (2020)*, separa **onde a informação vive** de **quem redige a resposta**.

```
Pergunta
   ↓
[1] Recuperação  → busca os K trechos mais relevantes na base
   ↓
[2] Aumento      → monta o prompt com pergunta + trechos
   ↓
[3] Geração      → o modelo responde usando apenas o que recebeu
   ↓
Resposta + citação da fonte
```

---

## 🎯 Que problemas resolve

| Problema do LLM puro | Como o RAG resolve |
|---|---|
| Corte de conhecimento | A base é atualizada sem retreinar nada |
| [[ia-alucinacao-e-grounding\|Alucinação]] | A resposta fica ancorada em texto fornecido |
| Falta de fonte | Cada resposta pode citar de onde veio |
| Dados privados | A base é sua; nada precisa entrar em treino |
| Custo de fine-tuning | Muito mais barato que treinar |

---

## 🏗️ As duas fases

### Fase 1 — Indexação (offline, roda uma vez)

```
Documentos → Chunking → Embedding → Banco vetorial
```

**Chunking** é a decisão mais subestimada do pipeline:

| Estratégia | Comentário |
|---|---|
| Tamanho fixo | Simples; corta frases no meio |
| Por parágrafo | Respeita a estrutura do texto |
| Com sobreposição | 10–20% de overlap evita perder contexto na borda |
| Semântico | Corta onde o assunto muda; melhor e mais caro |

> [!warning] Chunk ruim é a causa nº 1 de RAG ruim
> Chunk grande demais dilui o sinal e enche a janela de contexto. Pequeno demais perde o contexto necessário para a resposta fazer sentido. Antes de culpar o modelo por respostas fracas, **inspecione o que a recuperação está trazendo** — na maioria dos casos o problema está lá, não na geração.

### Fase 2 — Consulta (online, a cada pergunta)

```
Pergunta → Embedding → Busca top-K → Reranking → Prompt → Resposta
```

O **reranking** é uma segunda passada que reordena os K candidatos com um modelo mais preciso (*cross-encoder*). Custa mais, e costuma ser a melhoria com melhor retorno depois do chunking.

---

## 📐 Avaliação de RAG

RAG tem duas metades que falham por motivos diferentes — e precisam ser avaliadas **separadamente**.

| Métrica | Mede | Metade |
|---|---|---|
| **Context Recall** | Os trechos certos foram recuperados? | Recuperação |
| **Context Precision** | Os trechos trazidos são relevantes? | Recuperação |
| **Faithfulness** | A resposta se sustenta nos trechos? | Geração |
| **Answer Relevance** | A resposta responde à pergunta? | Geração |

> [!important] Diagnostique a metade certa antes de otimizar
> Se a recuperação não trouxe a informação, **nenhum prompt salva a resposta**. Se trouxe e o modelo ignorou, o problema é de prompt ou de modelo. Medir as duas metades separado é o que evita semanas otimizando o lado errado. Ferramenta usual: **RAGAS**.

---

## ⚖️ RAG × Fine-tuning × Prompt longo

| | RAG | Fine-tuning | Contexto longo |
|---|---|---|---|
| Atualizar informação | ✅ trivial | ❌ retreina | ✅ trivial |
| Custo inicial | Médio | Alto | Baixo |
| Custo por consulta | Médio | Baixo | Alto |
| Citar fonte | ✅ | ❌ | 🟡 |
| Ensinar formato/estilo | ❌ | ✅ | 🟡 |
| Volume de dados | Ilimitado | Médio | Limitado pela janela |

> [!tip] A regra prática
> **RAG** para conhecimento que muda. **Fine-tuning** para comportamento e formato que não mudam. **Contexto longo** quando a base cabe inteira no prompt — e nesse caso, RAG é complexidade desnecessária.

---

## 🚧 Quando RAG é exagero

Se a base inteira cabe no prompt, não monte pipeline de recuperação. Um catálogo de 18 itens com descrição curta ocupa poucos milhares de tokens: mandar tudo é mais simples, mais barato de manter e mais fácil de defender do que indexação, banco vetorial e reranking.

**RAG começa a valer quando a base não cabe no contexto** — ou quando o custo por chamada de mandar tudo supera o custo de manter o índice.

---

## 📚 Referências

- **Lewis et al. (2020)** — *Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks*, NeurIPS — o artigo original
- **Gao et al. (2023)** — *Retrieval-Augmented Generation for Large Language Models: A Survey*
- **Es et al. (2023)** — *RAGAS: Automated Evaluation of Retrieval Augmented Generation*

---

## 🔗 Conceitos relacionados

- [[ia-embeddings|🧬 Embeddings]] — a base da recuperação
- [[ia-alucinacao-e-grounding|🎭 Alucinação e grounding]] · [[ia-engenharia-de-prompt|📝 Engenharia de prompt]]
- [[rec-similaridade-cosseno|📏 Similaridade de cosseno]] · [[ia-avaliacao-de-llm|📐 Avaliação de LLM]]

## Veja também

- [[Conceitos|🧠 Conceitos]]

---

**Status:** ✅ Completo
