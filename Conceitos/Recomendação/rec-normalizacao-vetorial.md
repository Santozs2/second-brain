---
type: concept
area: Conceitos
status: estavel
difficulty: intermediate
id: rec-normalizacao-vetorial
category: Recomendação
tags:
  - recomendacao
  - concept
  - mathematics
created: 2026-08-24
updated: 2026-08-24
---
# ⚖️ Normalização Vetorial

> Colocar vetores de tamanhos diferentes na mesma escala, para que a comparação meça **padrão** e não **volume**.

---

## 📖 O problema

Sem normalização, quem tem números maiores vence — independentemente de fazer sentido.

```
Perfil = [3, 3, 3]

Item A = [1, 1, 1]      soma dos produtos = 9
Item B = [10, 10, 10]   soma dos produtos = 90   ← "vence" por ser grande
```

Os dois itens têm **exatamente o mesmo padrão** (equilibrados nas três dimensões). B só tem números maiores. Se a magnitude não é informação relevante, ela é **ruído com poder de decisão**.

---

## 🧮 Normalização L2 (euclidiana) — a mais usada

Divide cada componente pela norma do vetor, produzindo um **vetor unitário** (comprimento 1) que preserva a direção.

$$\hat{v} = \frac{v}{\|v\|} = \frac{v}{\sqrt{\sum v_i^2}}$$

```python
def normalize_l2(v: list[float]) -> list[float]:
    n = math.sqrt(sum(x * x for x in v))
    return [x / n for x in v] if n else [0.0] * len(v)
```

```
[10, 10, 10] → norma = 17,32 → [0,577, 0,577, 0,577]
[1, 1, 1]    → norma =  1,73 → [0,577, 0,577, 0,577]   ← idênticos
```

> [!important] O cosseno já faz normalização L2 embutida
> A fórmula `(A·B) / (‖A‖ × ‖B‖)` **é** o produto escalar de dois vetores normalizados. Normalizar antes e depois aplicar cosseno é redundante — o resultado é o mesmo. Ver [[rec-similaridade-cosseno|📏 Similaridade de cosseno]].

---

## 📊 Outras normalizações

### L1 (Manhattan)
Divide pela soma dos valores absolutos. Resultado soma 1 — útil quando o vetor representa **distribuição de probabilidade**.

$$\hat{v} = \frac{v}{\sum |v_i|}$$

### Min-Max (escalonamento para [0,1])
Coloca cada **dimensão** na faixa [0,1], usando o mínimo e o máximo observados naquela dimensão.

$$x' = \frac{x - \min}{\max - \min}$$

✅ Resolve o problema de dimensões com escalas incompatíveis (nota 0–5 vs. salário 0–50.000)
❌ Sensível a outliers — um valor extremo comprime todo o resto

### Z-score (padronização)
Centra na média e divide pelo desvio-padrão. Resultado tem média 0 e desvio 1.

$$z = \frac{x - \mu}{\sigma}$$

✅ Robusta a escalas diferentes, tolera outliers melhor que min-max
❌ Produz valores negativos — quebra a leitura de cosseno como percentual

---

## 🧭 Qual usar

| Situação | Normalização |
|---|---|
| Comparar padrões, ignorando intensidade | **L2** (ou cosseno direto) |
| Vetor representa distribuição/proporção | **L1** |
| Dimensões com unidades muito diferentes | **Min-Max** ou **Z-score** por dimensão |
| Presença de outliers relevantes | **Z-score** |
| Já vai usar cosseno | **Nenhuma** — está embutida |

---

## 🚨 Normalização por vetor × normalização por dimensão

Confundir as duas é erro comum e silencioso:

```
Matriz de itens (linhas) × atributos (colunas)

Por VETOR (linha)     → cada item vira unitário
                      → remove o efeito "item gordo"

Por DIMENSÃO (coluna) → cada atributo entra na mesma escala
                      → remove o efeito "atributo com números grandes"
```

São problemas diferentes e podem ser necessários **ao mesmo tempo**: primeiro normalizar as colunas para equalizar escalas de atributos, depois normalizar as linhas para equalizar magnitudes de itens.

> [!warning] A ordem das operações muda o resultado
> Normalizar linha e depois coluna **não** é o mesmo que coluna e depois linha. Se o seu pipeline faz as duas, a ordem precisa estar documentada e ser sempre a mesma — inclusive entre treino e produção.

---

## 🔗 Conceitos relacionados

- [[rec-similaridade-cosseno|📏 Similaridade de cosseno]] · [[rec-metricas-similaridade|📏 Métricas de similaridade]]
- [[rec-modelo-espaco-vetorial|📐 Modelo de Espaço Vetorial]] · [[rec-tf-idf|📊 TF-IDF]]
- [[cs-linear-algebra|📐 Álgebra linear]]

## Veja também

- [[Conceitos|🧠 Conceitos]]

---

**Status:** ✅ Completo
