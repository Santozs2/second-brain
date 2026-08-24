---
type: concept
area: Conceitos
status: estavel
difficulty: intermediate
id: rec-modelo-espaco-vetorial
category: Recomendação
tags:
  - recomendacao
  - concept
  - mathematics
created: 2026-08-24
updated: 2026-08-24
---
# 📐 Modelo de Espaço Vetorial (VSM)

> Representar objetos heterogêneos como pontos em um mesmo espaço de N dimensões, para que "parecido" vire uma operação aritmética.

---

## 📖 Definição

O **Vector Space Model** foi formulado por *Salton, Wong & Yang (1975)* para recuperação de documentos. A ideia sobreviveu a tudo que veio depois porque é simples e geral:

> Escolha N atributos. Todo objeto vira um vetor de N números. A relação entre dois objetos vira uma operação entre dois vetores.

```
Espaço de 3 dimensões: [mecânica, elétrica, informática]

Curso A = [5, 2, 0]     ← forte em mecânica
Curso B = [0, 1, 5]     ← forte em informática
Perfil  = [4, 1, 1]     ← pessoa com afinidade mecânica
```

A partir daí, "qual item combina com esta pessoa" deixa de ser opinião e vira **geometria**.

---

## 🧱 As três decisões que definem um VSM

### 1. Quais são as dimensões

As dimensões precisam ser **exaustivas** (cobrem o domínio) e **pouco redundantes** (não medem a mesma coisa duas vezes). Duas dimensões que sempre andam juntas são, na prática, uma dimensão só — e distorcem o resultado ao contar o mesmo fator duas vezes.

### 2. Como cada objeto recebe seus valores

Este é o ponto mais frágil do modelo e o mais atacado em banca.

| Método | Como funciona | Custo | Defensabilidade |
|---|---|---|---|
| **Julgamento humano** | Especialista atribui nota 0–5 | Alto (N×D decisões) | Alta **se** o critério for declarado |
| **Derivado de dados** | Extraído de ementa, carga horária, texto | Médio | Alta — reproduzível |
| **TF-IDF** | Frequência de termos ponderada | Baixo | Alta → [[rec-tf-idf\|📊 TF-IDF]] |
| **Embeddings** | Vetor aprendido por modelo | Baixo | Média — não é interpretável → [[ia-embeddings\|🧬 Embeddings]] |

> [!warning] O gargalo nunca é o algoritmo — é a atribuição de pesos
> Um espaço com D dimensões e N itens exige **N × D decisões**. Para 18 itens e 7 dimensões, são 126 julgamentos; para 180 itens, 1.260. O algoritmo roda igual com qualquer número. **Quem não escala é o humano que preenche a matriz.**

### 3. Como se mede a proximidade

Distância euclidiana, cosseno, Jaccard, Pearson — cada uma responde a uma pergunta diferente. Ver [[rec-metricas-similaridade|📏 Métricas de similaridade]].

---

## ✅ Por que o VSM continua sendo escolhido

- **Interpretável** — dá para apontar qual dimensão puxou o resultado, o que sustenta [[rec-explicabilidade|💡 recomendação explicável]]
- **Independente de histórico** — funciona no dia zero, contornando [[rec-cold-start|🥶 cold start]]
- **Determinístico** — mesma entrada, mesma saída, sempre; auditável e testável
- **Barato** — não exige GPU, treino nem serviço externo
- **Incremental** — item novo entra sem recalcular nada dos outros

---

## ⚠️ Limitações honestas

| Limitação | Consequência |
|---|---|
| Assume independência entre dimensões | Dimensões correlacionadas contam duas vezes |
| Ignora ordem e contexto | "não gosto de mecânica" e "gosto de mecânica" podem virar o mesmo vetor |
| Qualidade limitada pelos pesos | Peso mal atribuído = recomendação ruim, sem o algoritmo errar nada |
| Não aprende com o uso | Sem laço de feedback, o sistema de hoje é igual ao de um ano atrás |

> [!tip] Declarar a limitação é mais forte do que escondê-la
> Em trabalho acadêmico, a frase *"o VSM assume independência entre dimensões, o que não se sustenta plenamente porque as áreas X e Y compartilham fundamentos"* **soma** pontos. A banca vai encontrar a limitação de qualquer jeito; é melhor que a encontre já mapeada. Ver [[met-validade-e-limitacoes|🎯 Validade e limitações]].

---

## 📚 Referências fundamentais

- **Salton, Wong & Yang (1975)** — *A Vector Space Model for Automatic Indexing*, Communications of the ACM 18(11) — o artigo original
- **Salton & McGill (1983)** — *Introduction to Modern Information Retrieval*
- **Manning, Raghavan & Schütze (2008)** — *Introduction to Information Retrieval*, cap. 6 — disponível livre online

---

## 🔗 Conceitos relacionados

- [[rec-similaridade-cosseno|📏 Similaridade de cosseno]] — a medida que o VSM normalmente usa
- [[rec-normalizacao-vetorial|⚖️ Normalização vetorial]] — por que o tamanho do vetor atrapalha
- [[cs-linear-algebra|📐 Álgebra linear]] · [[rec-sistemas-de-recomendacao|🎯 Sistemas de Recomendação]]

## Veja também

- [[Conceitos|🧠 Conceitos]]

---

**Status:** ✅ Completo
