---
type: concept
area: Conceitos
status: estavel
difficulty: intermediate
id: rec-filtragem-colaborativa
category: Recomendação
tags:
  - recomendacao
  - concept
created: 2026-08-24
updated: 2026-08-24
---
# 👥 Filtragem Colaborativa

> Recomenda pelo comportamento de pessoas parecidas com você. Não sabe — nem precisa saber — o que o item é.

---

## 📖 Definição

A filtragem colaborativa (*collaborative filtering*) parte de uma matriz de interações usuário × item e prevê as células vazias.

```
          Curso A  Curso B  Curso C  Curso D
Ana          5        3        ?        1
Bruno        4        3        4        1
Carla        1        ?        5        4
Você         5        3        ?        ?     ← prever aqui
```

O sistema nota que **você e Bruno se comportam parecido**, e Bruno gostou do Curso C. Logo, Curso C para você.

> [!note] O item é uma caixa-preta e isso é uma vantagem
> A filtragem colaborativa **não sabe** que o Curso C é de mecânica. Ela só sabe que quem gosta de A e B tende a gostar de C. Isso permite capturar padrões que nenhum atributo descreveria — combinações que ninguém pensaria em modelar.

---

## 🗂️ As duas abordagens de vizinhança

### Baseada em usuário (*user-based*)
Encontra usuários similares a você e recomenda o que eles consumiram.

```
similaridade(você, outros) → top-K vizinhos → itens que eles gostaram
```

Métrica usual: [[rec-metricas-similaridade|correlação de Pearson]], porque corrige o viés de quem dá nota alta para tudo.

### Baseada em item (*item-based*)
Encontra itens similares aos que você já consumiu, medindo similaridade **pelos padrões de consumo**, não pelos atributos.

```
"Quem consumiu A também consumiu C" → A e C são similares
```

✅ Mais estável — a relação entre itens muda menos que o gosto das pessoas
✅ Pré-computável offline, o que resolve escalabilidade
✅ É a abordagem que a Amazon popularizou (*Linden et al., 2003*)

---

## 🧮 Fatoração de matrizes

A evolução moderna. Em vez de comparar vizinhos, decompõe a matriz esparsa em **fatores latentes**:

```
Matriz(usuários × itens) ≈ U(usuários × k) × V(k × itens)
```

Os `k` fatores latentes são dimensões descobertas automaticamente — o modelo aprende que existe algo como "gosto por trabalho manual" sem ninguém ter nomeado isso.

Ficou famosa no **Prêmio Netflix (2006–2009)**, onde SVD e variantes dominaram. Referência: *Koren, Bell & Volinsky (2009)*.

✅ Lida bem com esparsidade
✅ Captura estrutura que vizinhança não vê
❌ Fatores latentes **não são interpretáveis** — adeus explicabilidade
❌ Exige retreino periódico

---

## ❌ As limitações que decidem arquitetura

| Limitação | Gravidade |
|---|---|
| **Cold start** | 🔴 Fatal — sem histórico, não há vizinhos → [[rec-cold-start\|🥶 nota dedicada]] |
| **Esparsidade** | 🔴 Matrizes reais são >99% vazias |
| **Escalabilidade** | 🟡 Comparar todos com todos é O(n²) |
| **Viés de popularidade** | 🟡 Itens populares dominam; cauda longa some |
| **Ovelha negra** | 🟡 Quem tem gosto único não tem vizinhos |
| **Não é explicável** | 🟡 "Pessoas como você gostaram" é justificativa fraca |
| **Vulnerável a ataque** | 🟡 *Shilling*: contas falsas manipulam recomendações |

> [!important] Filtragem colaborativa exige uma base que a maioria dos projetos não tem
> Ela é a técnica dominante em Netflix, Amazon e Spotify porque essas empresas têm milhões de interações. **Um sistema novo, institucional, com dezenas de usuários, não tem o insumo.** Reconhecer isso não é limitação do projeto — é diagnóstico correto do regime de dados.

---

## 🆚 Colaborativa versus conteúdo

| Critério | Colaborativa | Conteúdo |
|---|:---:|:---:|
| Precisa de histórico | ✅ sim | ❌ não |
| Precisa de atributos | ❌ não | ✅ sim |
| Funciona no dia zero | ❌ | ✅ |
| Descobre o inesperado | ✅ | ❌ |
| Explicável | ❌ | ✅ |
| Determinístico | ❌ | ✅ |

A combinação das duas é [[rec-sistemas-hibridos|🔀 sistema híbrido]].

---

## 📚 Referências fundamentais

- **Linden, Smith & York (2003)** — *Amazon.com Recommendations: Item-to-Item Collaborative Filtering*, IEEE Internet Computing
- **Koren, Bell & Volinsky (2009)** — *Matrix Factorization Techniques for Recommender Systems*, IEEE Computer — o artigo do Prêmio Netflix
- **Herlocker et al. (2004)** — *Evaluating Collaborative Filtering Recommender Systems*, ACM TOIS
- **Sarwar et al. (2001)** — *Item-Based Collaborative Filtering Recommendation Algorithms*, WWW

---

## 🔗 Conceitos relacionados

- [[rec-filtragem-conteudo|📄 Filtragem baseada em conteúdo]] · [[rec-cold-start|🥶 Cold start]]
- [[rec-sistemas-hibridos|🔀 Sistemas híbridos]] · [[rec-metricas-similaridade|📏 Métricas de similaridade]]

## Veja também

- [[Conceitos|🧠 Conceitos]]

---

**Status:** ✅ Completo
