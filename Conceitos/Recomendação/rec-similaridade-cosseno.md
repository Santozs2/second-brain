---
type: concept
area: Conceitos
status: estavel
difficulty: intermediate
id: rec-similaridade-cosseno
category: Recomendação
tags:
  - recomendacao
  - concept
  - mathematics
created: 2026-08-24
updated: 2026-08-24
---
# 📏 Similaridade de Cosseno

> Mede o **ângulo** entre dois vetores, ignorando o tamanho deles. Responde "estes dois apontam para a mesma direção?" em vez de "estes dois têm o mesmo tamanho?".

---

## 🧮 A fórmula

$$
\text{sim}(A, B) = \cos(\theta) = \frac{A \cdot B}{\|A\| \times \|B\|} = \frac{\sum_{i=1}^{n} A_i B_i}{\sqrt{\sum_{i=1}^{n} A_i^2} \times \sqrt{\sum_{i=1}^{n} B_i^2}}
$$

Em três partes independentes:

```
1. Produto escalar (numerador)  →  A · B   = Σ (Aᵢ × Bᵢ)
2. Norma de cada vetor          →  ‖A‖     = √(Σ Aᵢ²)
3. Divisão                      →  A·B / (‖A‖ × ‖B‖)
```

O numerador mede **o quanto os dois concordam**. O denominador **remove a influência da magnitude**. É essa divisão que faz todo o trabalho conceitual.

---

## 📊 Faixa de valores

| Valor | Ângulo | Significado |
|---:|---:|---|
| `1.0` | 0° | Direções idênticas — proporcionais |
| `0.8` | ~37° | Muito parecidos |
| `0.5` | 60° | Parcialmente alinhados |
| `0.0` | 90° | Ortogonais — nada em comum |
| `-1.0` | 180° | Opostos |

> [!note] Com vetores não-negativos, o resultado vive em [0, 1]
> Se todos os atributos são contagens ou notas de 0 a 5, nenhum componente é negativo e o cosseno **nunca fica abaixo de zero**. Isso permite ler o valor diretamente como percentual de afinidade — `0,87 → 87%`. Valores negativos só aparecem quando o modelo admite atributos negativos (ex.: escalas de -5 a +5).

---

## 🎯 O problema que o cosseno resolve

Considere pontuar cursos por soma simples de pontos:

```
Perfil da pessoa      = [5, 0, 0]   ← só quer mecânica

Curso Especialista    = [5, 0, 0]   soma dos produtos = 25
Curso Generalista     = [5, 5, 5]   soma dos produtos = 25   ← EMPATE
```

Por soma de pontos, os dois empatam. Mas o Generalista **não é sobre mecânica** — ele é sobre tudo, e só por isso acumulou pontos. Com cosseno:

```
Especialista:  25 / (5 × 5)     = 25 / 25    = 1,00  ✅
Generalista:   25 / (5 × 8,66)  = 25 / 43,3  = 0,58  ✅
```

> [!important] Isto é chamado de "problema do item gordo"
> Um item que pesa muito em **todas** as dimensões acumula pontos por volume, não por afinidade. A **normalização pela norma** neutraliza isso: quanto mais espalhado o item, maior o denominador, menor a similaridade. Sem essa divisão, o sistema recomenda sempre o item mais genérico do catálogo — que é exatamente o menos útil.

---

## 💻 Implementação em funções puras

```python
import math

def dot(a: list[float], b: list[float]) -> float:
    """Produto escalar: soma dos produtos componente a componente."""
    return sum(x * y for x, y in zip(a, b))

def norm(v: list[float]) -> float:
    """Norma euclidiana (comprimento) do vetor."""
    return math.sqrt(sum(x * x for x in v))

def cosine_similarity(a: list[float], b: list[float]) -> float:
    """Similaridade de cosseno entre dois vetores. Retorna 0.0 se algum for nulo."""
    na, nb = norm(a), norm(b)
    if na == 0 or nb == 0:      # guarda contra divisão por zero
        return 0.0
    return dot(a, b) / (na * nb)
```

> [!tip] Funções puras são a decisão de arquitetura que rende defesa
> Nenhuma dessas funções sabe o que é um curso, um banco de dados ou uma requisição HTTP. Elas recebem listas de números e devolvem um número. Consequências: são **testáveis sem banco**, **reutilizáveis em qualquer domínio** e **verificáveis à mão** com papel e caneta — o que permite provar a corretude em uma monografia. Ver [[tst-piramide-de-testes|🔺 Pirâmide de testes]].

---

## 🚨 Os quatro casos-limite que precisam de teste

| Caso | Entrada | Comportamento esperado |
|---|---|---|
| **Vetor nulo** | `[0,0,0]` | Retornar `0.0`, não estourar `ZeroDivisionError` |
| **Vetores idênticos** | `A = B` | Exatamente `1.0` |
| **Proporcionais** | `[1,2]` e `[2,4]` | `1.0` — o cosseno ignora escala |
| **Ortogonais** | `[1,0]` e `[0,1]` | `0.0` |

> [!warning] Ponto flutuante quase nunca devolve 1.0 cravado
> `cosine_similarity([1,2,3], [1,2,3])` pode retornar `0.9999999999999998`. **Nunca compare com `==` em teste.** Use `assertAlmostEqual(resultado, 1.0, places=6)` ou `math.isclose(resultado, 1.0)`. Este é o bug mais comum em suíte de teste de engine vetorial.

---

## ⚖️ Cosseno versus as alternativas

| Medida | O que mede | Quando prefere |
|---|---|---|
| **Cosseno** | Ângulo (direção) | Magnitude é ruído — o padrão importa mais que a intensidade |
| **Euclidiana** | Distância absoluta | Magnitude é sinal legítimo |
| **Pearson** | Correlação linear | Precisa corrigir viés individual (quem dá nota alta pra tudo) |
| **Jaccard** | Sobreposição de conjuntos | Dados binários (tem/não tem) |

Comparação completa em [[rec-metricas-similaridade|📏 Métricas de similaridade]].

---

## ⚠️ Limitações

- **Ignora magnitude por definição** — se a intensidade importa (alguém que respondeu "5" em tudo é diferente de quem respondeu "1" em tudo), o cosseno apaga essa diferença. Os dois têm a mesma direção.
- **Sensível à escolha das dimensões** — dimensões redundantes inflam o peso do fator que elas compartilham
- **Perfil todo-zero é indefinido** — quem não respondeu nada não tem direção; o retorno `0.0` é convenção, não matemática
- **Empates são frequentes e reais** — dois itens podem ficar a `0,001` de distância; nesse caso, o ranking é ruído. Isso pede um **indicador de confiança**, não um desempate arbitrário.

---

## 📚 Referências fundamentais

- **Salton & McGill (1983)** — *Introduction to Modern Information Retrieval* — a formulação clássica
- **Manning, Raghavan & Schütze (2008)** — *Introduction to Information Retrieval*, seção 6.3 — dedução completa
- **Singhal, A. (2001)** — *Modern Information Retrieval: A Brief Overview* — contexto histórico da normalização

---

## 🔗 Conceitos relacionados

- [[rec-modelo-espaco-vetorial|📐 Modelo de Espaço Vetorial]] — o modelo que o cosseno serve
- [[rec-normalizacao-vetorial|⚖️ Normalização vetorial]] — o que a divisão pela norma faz
- [[rec-metricas-similaridade|📏 Métricas de similaridade]] · [[cs-linear-algebra|📐 Álgebra linear]]

## Veja também

- [[Conceitos|🧠 Conceitos]]

---

**Status:** ✅ Completo
