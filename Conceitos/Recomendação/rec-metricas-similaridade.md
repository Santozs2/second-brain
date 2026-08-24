---
type: concept
area: Conceitos
status: estavel
difficulty: intermediate
id: rec-metricas-similaridade
category: Recomendação
tags:
  - recomendacao
  - concept
  - mathematics
created: 2026-08-24
updated: 2026-08-24
---
# 📏 Métricas de Similaridade

> Existem muitas formas de medir "parecido". Escolher a errada não gera erro — gera resultado ruim que parece certo.

---

## 📖 Por que a escolha importa

Toda métrica de similaridade responde a uma pergunta implícita. Trocar a métrica troca a pergunta sem avisar:

| Métrica | Pergunta que ela responde |
|---|---|
| Cosseno | "Apontam para a mesma direção?" |
| Euclidiana | "Estão perto no espaço?" |
| Pearson | "Variam juntos?" |
| Jaccard | "Compartilham os mesmos elementos?" |
| Manhattan | "Quanto custa ir de um ao outro em passos retos?" |

---

## 1️⃣ Similaridade de Cosseno

$$\cos(\theta) = \frac{A \cdot B}{\|A\| \times \|B\|}$$

Ignora magnitude, mede direção. **Faixa:** [-1, 1] — ou [0, 1] com atributos não-negativos.

```python
def cosine_similarity(a, b):
    na, nb = norm(a), norm(b)
    return dot(a, b) / (na * nb) if na and nb else 0.0
```

✅ Padrão em recuperação de informação e recomendação por conteúdo
❌ Descarta intensidade, que às vezes é sinal legítimo

Detalhe completo em [[rec-similaridade-cosseno|📏 nota dedicada]].

---

## 2️⃣ Distância Euclidiana

$$d(A,B) = \sqrt{\sum_{i=1}^{n}(A_i - B_i)^2}$$

A distância "em linha reta". **Faixa:** [0, ∞) — e é **distância**, não similaridade: quanto menor, mais parecido.

```python
def euclidean_distance(a, b):
    return math.sqrt(sum((x - y) ** 2 for x, y in zip(a, b)))

def euclidean_similarity(a, b):
    return 1 / (1 + euclidean_distance(a, b))   # converte para [0, 1]
```

✅ Quando a intensidade importa de verdade (coordenadas, medidas físicas)
❌ Sofre com a **maldição da dimensionalidade** — em muitas dimensões, todas as distâncias convergem e param de discriminar
❌ Sensível a escalas diferentes entre atributos → exige normalizar antes

> [!warning] O erro clássico: euclidiana em atributos de escalas diferentes
> Se uma dimensão vai de 0 a 5 e outra de 0 a 50.000, a segunda domina a distância inteira e a primeira vira decoração. Ou você normaliza todas as dimensões para a mesma faixa, ou usa cosseno.

---

## 3️⃣ Correlação de Pearson

$$r = \frac{\sum (A_i - \bar{A})(B_i - \bar{B})}{\sqrt{\sum (A_i - \bar{A})^2} \sqrt{\sum (B_i - \bar{B})^2}}$$

É **o cosseno aplicado aos vetores centrados na média**. **Faixa:** [-1, 1].

A diferença prática: Pearson remove o **viés individual**. Quem avalia tudo com nota alta e quem avalia tudo com nota baixa, mas na mesma ordem de preferência, têm correlação `1,0` — enquanto o cosseno os veria como diferentes.

✅ Padrão em [[rec-filtragem-colaborativa|filtragem colaborativa]] baseada em usuário
❌ Precisa de sobreposição suficiente para a média fazer sentido
❌ Instável com poucos pontos — dois itens em comum produzem correlação sem significado

---

## 4️⃣ Índice de Jaccard

$$J(A,B) = \frac{|A \cap B|}{|A \cup B|}$$

Para **conjuntos**, não vetores numéricos. **Faixa:** [0, 1].

```python
def jaccard(a: set, b: set) -> float:
    uniao = a | b
    return len(a & b) / len(uniao) if uniao else 0.0
```

✅ Dados binários: tags, categorias, "tem/não tem", listas de habilidades
❌ Descarta qualquer noção de grau — não distingue "muito" de "pouco"

---

## 5️⃣ Distância de Manhattan (L1)

$$d(A,B) = \sum_{i=1}^{n}|A_i - B_i|$$

Soma das diferenças absolutas. Mais robusta a valores extremos que a euclidiana, porque não eleva ao quadrado.

✅ Quando outliers existem e não devem dominar
❌ Menos intuitiva geometricamente

---

## 🧭 Tabela de decisão

| Seus dados são... | Use |
|---|---|
| Vetores de peso onde o **padrão** importa mais que a intensidade | **Cosseno** |
| Coordenadas ou medidas físicas na mesma escala | **Euclidiana** |
| Notas de usuários com viés pessoal de generosidade | **Pearson** |
| Conjuntos de tags, habilidades ou categorias | **Jaccard** |
| Vetores numéricos com outliers relevantes | **Manhattan** |
| Texto livre com significado semântico | **Cosseno sobre [[ia-embeddings\|embeddings]]** |

---

## 🔬 O teste que revela a diferença

Rodar a mesma base com duas métricas e comparar os rankings **é um experimento publicável**, não perda de tempo. Se os rankings coincidem, você ganhou um argumento de robustez ("o resultado não depende da métrica escolhida"). Se divergem, você ganhou uma discussão sobre por quê — e discussão é o que separa monografia de relatório.

```python
# Esqueleto do experimento comparativo
for metrica in (cosine_similarity, euclidean_similarity, pearson):
    ranking = sorted(itens, key=lambda i: metrica(perfil, vetor(i)), reverse=True)
    print(metrica.__name__, [i.nome for i in ranking[:5]])
```

Ver [[met-tipos-de-pesquisa|🔬 Tipos de pesquisa]] para enquadrar isso metodologicamente.

---

## 📚 Referências

- **Manning, Raghavan & Schütze (2008)** — *Introduction to Information Retrieval*, cap. 6
- **Cha, S. (2007)** — *Comprehensive Survey on Distance/Similarity Measures* — catálogo exaustivo de métricas
- **Aggarwal, C. C. (2016)** — *Recommender Systems: The Textbook*, cap. 2

---

## 🔗 Conceitos relacionados

- [[rec-similaridade-cosseno|📏 Similaridade de cosseno]] · [[rec-normalizacao-vetorial|⚖️ Normalização vetorial]]
- [[rec-modelo-espaco-vetorial|📐 Modelo de Espaço Vetorial]] · [[cs-linear-algebra|📐 Álgebra linear]]

## Veja também

- [[Conceitos|🧠 Conceitos]]

---

**Status:** ✅ Completo
