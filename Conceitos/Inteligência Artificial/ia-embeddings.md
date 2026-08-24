---
type: concept
area: Conceitos
status: estavel
difficulty: intermediate
id: ia-embeddings
category: Inteligência Artificial
tags:
  - ia
  - llm
  - concept
  - mathematics
created: 2026-08-24
updated: 2026-08-24
---
# 🧬 Embeddings

> Representações vetoriais densas em que **proximidade geométrica corresponde a proximidade de significado**. É o VSM em que ninguém precisa escolher as dimensões.

---

## 📖 O que são

Um embedding é um vetor de números reais (tipicamente 384 a 3072 dimensões) que representa um texto, imagem ou item, **aprendido** por um modelo em vez de definido por um humano.

```
"curso de mecânica industrial"  → [0.021, -0.118, 0.435, ..., 0.077]
"formação em manutenção fabril" → [0.019, -0.121, 0.428, ..., 0.081]
                                   ↑ vetores próximos, palavras diferentes
```

A propriedade central: **similaridade semântica vira distância vetorial**, medida quase sempre por [[rec-similaridade-cosseno|cosseno]].

---

## 🆚 Embeddings versus vetores manuais

| | Vetor manual / TF-IDF | Embedding |
|---|---|---|
| Origem das dimensões | Definidas por humano ou por termo | Aprendidas pelo modelo |
| Cada dimensão tem nome | ✅ "mecânica", "elétrica" | ❌ sem significado isolado |
| Captura sinônimos | ❌ | ✅ |
| Interpretável | ✅ | ❌ |
| Custo de construção | Alto (N×D decisões) | Baixo (uma chamada) |
| Auditável | ✅ | 🟡 só por comportamento |
| Esparsidade | Alta | Densa |

> [!important] O trade-off é explicabilidade contra poder semântico
> Embeddings capturam relações que um vetor manual jamais capturaria. Em troca, você perde a capacidade de dizer *"foi recomendado porque pontuou 5 em mecânica"* — porque a dimensão 247 não significa nada nomeável. Em domínio que exige justificativa, isso é um custo real. Ver [[rec-explicabilidade|💡 Explicabilidade]].

---

## 🎯 Usos práticos

| Uso | Como |
|---|---|
| **Busca semântica** | Embedar consulta e documentos; ranquear por cosseno |
| **[[ia-rag\|RAG]]** | Recuperar trechos relevantes para injetar no prompt |
| **Recomendação por conteúdo** | Vetor do item derivado da descrição textual |
| **Clusterização** | Agrupar itens similares sem rótulo prévio |
| **Deduplicação** | Detectar itens quase idênticos |
| **Classificação** | Embedding como atributo de entrada de um classificador simples |

---

## 💻 Uso típico

```python
# Similaridade semântica entre textos
from sentence_transformers import SentenceTransformer

modelo = SentenceTransformer("paraphrase-multilingual-MiniLM-L12-v2")

textos = ["mecânica industrial", "manutenção de máquinas", "design gráfico"]
vetores = modelo.encode(textos, normalize_embeddings=True)

# Com vetores normalizados, o produto escalar JÁ É o cosseno
similaridade = vetores[0] @ vetores[1]   # alta
divergencia  = vetores[0] @ vetores[2]   # baixa
```

> [!tip] Para português, escolha modelo multilíngue
> Modelos treinados só em inglês degradam bastante em português. Opções que funcionam bem: `paraphrase-multilingual-MiniLM-L12-v2` (leve, roda em CPU), `multilingual-e5-base`, ou as APIs de embedding dos provedores comerciais. Rodar local elimina custo por chamada e dependência de rede.

---

## 🗄️ Bancos vetoriais

Para catálogos grandes, comparar a consulta contra todos os vetores é O(n). Bancos vetoriais usam índices aproximados (HNSW, IVF) para busca sublinear.

| Opção | Perfil |
|---|---|
| **pgvector** | Extensão do PostgreSQL — sem infra nova |
| **Qdrant** | Dedicado, open source, bom filtro por metadado |
| **FAISS** | Biblioteca (não servidor), da Meta |
| **Chroma** | Leve, ótimo para protótipo |

> [!note] Abaixo de ~10 mil itens, você não precisa de banco vetorial
> Um array NumPy em memória com produto de matrizes resolve em milissegundos. Introduzir infraestrutura antes de precisar é complexidade sem contrapartida — e uma escolha difícil de justificar em banca.

---

## ⚠️ Limitações

- **Dimensões sem significado** — impossível explicar o resultado por dimensão
- **Herdam vieses do treino** — associações estereotipadas ficam codificadas no espaço → [[rec-vieses-e-etica|⚖️ vieses]]
- **Dependem do modelo** — trocar de modelo invalida todos os vetores armazenados
- **Custo de recomputação** — mudou o modelo, reembeda o catálogo inteiro
- **Contexto limitado** — textos longos precisam ser fatiados, e o corte afeta o resultado
- **Não capturam negação com confiabilidade** — "não gosto de X" fica próximo de "gosto de X"

---

## 📚 Referências

- **Mikolov et al. (2013)** — *Efficient Estimation of Word Representations in Vector Space* (word2vec)
- **Devlin et al. (2019)** — *BERT: Pre-training of Deep Bidirectional Transformers*
- **Reimers & Gurevych (2019)** — *Sentence-BERT* — embeddings de sentença comparáveis por cosseno
- **Bolukbasi et al. (2016)** — *Man is to Computer Programmer as Woman is to Homemaker?* — viés em embeddings

---

## 🔗 Conceitos relacionados

- [[rec-modelo-espaco-vetorial|📐 Modelo de Espaço Vetorial]] · [[rec-similaridade-cosseno|📏 Similaridade de cosseno]]
- [[rec-tf-idf|📊 TF-IDF]] — a alternativa interpretável
- [[ia-rag|📚 RAG]] · [[ia-llm-fundamentos|🧠 LLM — Fundamentos]]

## Veja também

- [[Conceitos|🧠 Conceitos]]

---

**Status:** ✅ Completo
