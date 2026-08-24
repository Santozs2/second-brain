---
type: concept
area: Conceitos
status: estavel
difficulty: intermediate
id: rec-tf-idf
category: Recomendação
tags:
  - recomendacao
  - concept
  - mathematics
created: 2026-08-24
updated: 2026-08-24
---
# 📊 TF-IDF

> Pondera termos pela frequência local **e** pela raridade global. Um termo que aparece em todo documento não distingue nada — e o TF-IDF pune isso automaticamente.

---

## 📖 A intuição

Ao descrever documentos por palavras, dois efeitos competem:

- Um termo que aparece **muito** neste documento é provavelmente importante para ele → **TF**
- Um termo que aparece em **todos** os documentos não distingue nenhum → **IDF**

O TF-IDF multiplica os dois, premiando termos que são frequentes aqui e raros no resto.

---

## 🧮 A fórmula

$$\text{tf-idf}(t, d, D) = \text{tf}(t,d) \times \text{idf}(t,D)$$

### Term Frequency (TF)
$$\text{tf}(t,d) = \frac{\text{ocorrências de } t \text{ em } d}{\text{total de termos em } d}$$

### Inverse Document Frequency (IDF)
$$\text{idf}(t,D) = \log\frac{|D|}{|\{d \in D : t \in d\}|}$$

Onde `|D|` é o total de documentos e o denominador é quantos documentos contêm o termo.

```
100 documentos no total

"o"        aparece em 100 → idf = log(100/100) = log(1)  = 0     ← anulado
"sistema"  aparece em  50 → idf = log(100/50)  = log(2)  ≈ 0,69
"cosseno"  aparece em   2 → idf = log(100/2)   = log(50) ≈ 3,91  ← muito informativo
```

> [!note] O IDF anula sozinho as palavras vazias
> Termos como "o", "de", "para" aparecem em todo documento, logo `idf = log(1) = 0`, logo `tf-idf = 0`. **A remoção de *stopwords* acontece de graça pela matemática** — não precisa de lista manual.

---

## 💻 Implementação

```python
import math
from collections import Counter

def tf(termo: str, doc: list[str]) -> float:
    return doc.count(termo) / len(doc) if doc else 0.0

def idf(termo: str, corpus: list[list[str]]) -> float:
    n_com_termo = sum(1 for doc in corpus if termo in doc)
    if n_com_termo == 0:
        return 0.0
    return math.log(len(corpus) / n_com_termo)

def tf_idf_vector(doc: list[str], corpus: list[list[str]], vocab: list[str]):
    return [tf(t, doc) * idf(t, corpus) for t in vocab]
```

Na prática se usa `TfidfVectorizer` do **scikit-learn**, que aplica suavização no IDF (`+1` no denominador) para evitar divisão por zero com termos ausentes.

---

## 🎯 Onde entra em recomendação

TF-IDF é a forma clássica de **automatizar a construção dos vetores** de um [[rec-modelo-espaco-vetorial|modelo de espaço vetorial]] quando os itens têm descrição textual.

```
Ementa do curso → tokenizar → TF-IDF → vetor do curso
                                          ↓
                        cosseno com o vetor de perfil → ranking
```

> [!tip] Isto ataca diretamente o gargalo da atribuição manual de pesos
> Se atribuir pesos à mão custa N×D decisões humanas, derivar os vetores de textos que **já existem** (ementa, plano de curso, descrição de vaga) elimina o gargalo — ao preço de perder o controle fino sobre cada peso. O desenho mais forte costuma ser híbrido: TF-IDF gera a proposta inicial, o especialista revisa e ajusta.

---

## ⚖️ TF-IDF versus embeddings

| | TF-IDF | [[ia-embeddings\|Embeddings]] |
|---|---|---|
| Base | Frequência de termos | Significado aprendido |
| "carro" ≈ "automóvel" | ❌ termos distintos | ✅ vetores próximos |
| Interpretável | ✅ cada dimensão é um termo | ❌ dimensões sem nome |
| Custo | Baixo, roda em CPU | Alto ou dependente de API |
| Determinístico | ✅ | ✅ (mas depende do modelo) |
| Precisa de treino | ❌ | ✅ (ou modelo pronto) |

> [!important] TF-IDF continua competitivo e continua defensável
> Não é uma técnica ultrapassada — é uma técnica **transparente**. Em contexto onde a explicação importa mais que o último ponto percentual de acurácia, o fato de cada dimensão ter um nome legível vale mais que a superioridade semântica dos embeddings. Ver [[rec-explicabilidade|💡 Explicabilidade]].

---

## ⚠️ Limitações

- **Ignora ordem** — "não é bom" e "é bom, não" produzem o mesmo vetor (modelo *bag-of-words*)
- **Ignora sinonímia** — "docente" e "professor" são dimensões independentes
- **Ignora polissemia** — "manga" (fruta) e "manga" (roupa) são o mesmo termo
- **Vocabulário grande gera vetores esparsos** — milhares de dimensões, quase todas zero
- **Sensível ao corpus** — o IDF muda quando documentos entram ou saem

---

## 📚 Referências

- **Spärck Jones, K. (1972)** — *A Statistical Interpretation of Term Specificity and Its Application in Retrieval* — a origem do IDF
- **Salton & Buckley (1988)** — *Term-Weighting Approaches in Automatic Text Retrieval*
- **Manning, Raghavan & Schütze (2008)** — *Introduction to Information Retrieval*, cap. 6

---

## 🔗 Conceitos relacionados

- [[rec-modelo-espaco-vetorial|📐 Modelo de Espaço Vetorial]] · [[rec-similaridade-cosseno|📏 Similaridade de cosseno]]
- [[rec-normalizacao-vetorial|⚖️ Normalização vetorial]] · [[ia-embeddings|🧬 Embeddings]]

## Veja também

- [[Conceitos|🧠 Conceitos]]

---

**Status:** ✅ Completo
