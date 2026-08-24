---
type: concept
area: Conceitos
status: estavel
difficulty: beginner
id: rec-sistemas-de-recomendacao
category: Recomendação
tags:
  - recomendacao
  - concept
created: 2026-08-24
updated: 2026-08-24
---
# 🎯 Sistemas de Recomendação

> Sistemas que filtram um catálogo grande demais para ser lido inteiro e devolvem o subconjunto mais relevante para uma pessoa específica.

---

## 📖 Definição

Um sistema de recomendação resolve o **problema da sobrecarga de informação**: existem 180 cursos, 10 mil produtos ou 50 mil filmes, e a pessoa consegue avaliar cinco. O sistema estima uma **função de utilidade** e devolve os itens de maior valor.

```
u : Usuários × Itens → Relevância

recomendação(user) = argmax_{item ∈ Catálogo} u(user, item)
```

O trabalho inteiro está em estimar `u` sem nunca ter observado a maior parte dos pares.

---

## 🗂️ As três famílias clássicas

### 1. Baseado em conteúdo (*content-based*)

Descreve **item e usuário no mesmo espaço de atributos** e mede a distância entre eles. "Você gostou de coisas com estas características; aqui está outra coisa com estas características."

- **Precisa de:** atributos dos itens + um perfil do usuário
- **Não precisa de:** outros usuários
- Ver [[rec-filtragem-conteudo|📄 Filtragem baseada em conteúdo]]

### 2. Filtragem colaborativa (*collaborative filtering*)

Ignora os atributos e olha só o **padrão de comportamento**. "Pessoas que se comportaram como você gostaram disto."

- **Precisa de:** histórico de interações de muitos usuários
- **Não precisa de:** saber nada sobre o item
- Ver [[rec-filtragem-colaborativa|👥 Filtragem colaborativa]]

### 3. Híbrido

Combina as duas para cobrir a fraqueza de cada uma.

- Ver [[rec-sistemas-hibridos|🔀 Sistemas híbridos]]

---

## ⚖️ Como escolher a família

| Situação | Família indicada | Por quê |
|---|---|---|
| Não existe histórico de usuários | **Conteúdo** | Colaborativa exige dados que você não tem |
| Catálogo pequeno e bem descrito | **Conteúdo** | Os atributos carregam informação suficiente |
| Milhões de interações registradas | **Colaborativa** | Captura padrões que atributo nenhum descreve |
| Descoberta de itens inesperados | **Colaborativa** | Conteúdo só recomenda "mais do mesmo" |
| O sistema precisa explicar a escolha | **Conteúdo** | A justificativa sai direto dos atributos |
| Produto maduro em produção | **Híbrido** | Cobre os dois regimes |

> [!important] A decisão é ditada pelos dados que existem, não pela sofisticação da técnica
> A pergunta correta não é "qual técnica é melhor?", e sim **"que dados eu tenho no dia zero?"**. Um sistema sem base de usuários não tem escolha: é conteúdo, ou é nada. Ver [[rec-cold-start|🥶 Cold start]].

---

## 🚧 Os problemas estruturais da área

| Problema | O que é |
|---|---|
| **Cold start** | Usuário ou item novo, sem histórico → [[rec-cold-start\|🥶 nota dedicada]] |
| **Esparsidade** | A matriz usuário × item é quase toda vazia |
| **Escalabilidade** | Comparar todos contra todos custa O(n²) |
| **Bolha de filtro** | O sistema reforça o que a pessoa já é |
| **Viés de popularidade** | Itens populares afundam a cauda longa → [[rec-vieses-e-etica\|⚖️ vieses e ética]] |
| **Caixa-preta** | A pessoa não sabe por que recebeu aquilo → [[rec-explicabilidade\|💡 explicabilidade]] |

---

## 🧪 Como se avalia

Recomendação **não se avalia por acurácia sozinha**. As dimensões que importam incluem cobertura, diversidade, novidade e serendipidade, além da precisão no topo da lista.

Ver [[rec-metricas-avaliacao|📊 Métricas de avaliação]].

---

## 📚 Referências fundamentais

- **Ricci, Rokach & Shapira** — *Recommender Systems Handbook* (Springer) — a referência canônica da área
- **Aggarwal, C. C. (2016)** — *Recommender Systems: The Textbook* — o mais didático para fundamentação teórica
- **Adomavicius & Tuzhilin (2005)** — *Toward the Next Generation of Recommender Systems*, IEEE TKDE — consolidou a taxonomia das três famílias

---

## 🔗 Conceitos relacionados

- [[rec-modelo-espaco-vetorial|📐 Modelo de Espaço Vetorial]] — como itens viram vetores
- [[rec-similaridade-cosseno|📏 Similaridade de cosseno]] — a medida mais usada
- [[cs-linear-algebra|📐 Álgebra linear]] — a matemática por baixo

## Veja também

- [[Conceitos|🧠 Conceitos]]
- [[Documentações|Documentações]]

---

**Status:** ✅ Completo
