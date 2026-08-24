---
type: concept
area: Conceitos
status: estavel
difficulty: advanced
id: rec-sistemas-hibridos
category: Recomendação
tags:
  - recomendacao
  - concept
created: 2026-08-24
updated: 2026-08-24
---
# 🔀 Sistemas Híbridos de Recomendação

> Combinar duas ou mais técnicas para que a força de uma cubra a fraqueza da outra. Praticamente todo sistema real em produção é híbrido.

---

## 📖 Por que hibridizar

As duas famílias falham em pontos opostos:

| | Conteúdo | Colaborativa |
|---|:---:|:---:|
| Cold start | ✅ resolve | ❌ falha |
| Serendipidade | ❌ falha | ✅ resolve |
| Explicabilidade | ✅ resolve | ❌ falha |
| Captura padrão não-modelado | ❌ falha | ✅ resolve |

A complementaridade é quase perfeita — e é exatamente isso que torna a hibridização atraente.

---

## 🧱 As sete estratégias de Burke (2002)

*Robin Burke* catalogou os desenhos possíveis em um artigo que continua sendo a referência da área:

| Estratégia | Como funciona |
|---|---|
| **Ponderada** (*weighted*) | Combina os escores: `score = α·conteúdo + (1-α)·colaborativa` |
| **Alternada** (*switching*) | Escolhe uma técnica conforme o contexto (ex.: conteúdo se usuário é novo) |
| **Misturada** (*mixed*) | Apresenta resultados das duas lado a lado |
| **Combinação de atributos** | Injeta dados colaborativos como atributos de um modelo de conteúdo |
| **Cascata** (*cascade*) | Uma técnica gera candidatos, a outra refina o ranking |
| **Aumento de atributos** | A saída de uma técnica vira entrada da outra |
| **Meta-nível** | O modelo aprendido por uma técnica é o insumo da outra |

---

## ⭐ Cascata: o desenho mais defensável

```
[Etapa 1 — Recuperação]        [Etapa 2 — Refinamento]
Técnica barata e determinística   Técnica cara e sofisticada
    ↓                                 ↓
Reduz 180 itens → top-N         Reordena os N candidatos
```

Por que este desenho é forte:

- **Custo controlado** — a técnica cara só roda sobre N itens, não sobre o catálogo inteiro
- **Garantia de sanidade** — a etapa 2 **não pode** introduzir um item que a etapa 1 não aprovou
- **Degradação graciosa** — se a etapa 2 falha, a saída da etapa 1 ainda é uma resposta válida
- **Auditável** — dá para inspecionar as duas etapas separadamente

> [!important] Em cascata, quem define o conjunto de candidatos define o limite de erro
> A segunda etapa só pode reordenar, nunca inventar. Isso transforma um componente imprevisível em um componente **contido**: o pior caso da etapa 2 é uma ordenação ruim de itens que já eram aceitáveis. É a diferença entre um sistema que pode errar de leve e um que pode errar de forma catastrófica.

---

## 📐 Hibridização ponderada

```python
def score_hibrido(perfil, item, alpha=0.7):
    """alpha=1.0 → só conteúdo; alpha=0.0 → só colaborativa."""
    s_conteudo = cosine_similarity(perfil.vetor, item.vetor)
    s_colab = predicao_colaborativa(perfil.usuario, item)
    return alpha * s_conteudo + (1 - alpha) * s_colab
```

O `alpha` pode ser **adaptativo**: alto quando o usuário é novo, decaindo conforme o histórico cresce.

```python
def alpha_adaptativo(n_interacoes, limiar=20):
    """Migra de conteúdo para colaborativa conforme os dados chegam."""
    return max(0.3, 1.0 - (n_interacoes / limiar))
```

Este é o desenho que resolve [[rec-cold-start|🥶 cold start]] de forma elegante: o sistema **começa** explicável e determinístico e **evolui** para capturar padrões, sem nunca ter uma tela vazia.

---

## ⚠️ O custo da hibridização

| Custo | Detalhe |
|---|---|
| **Complexidade** | Dois sistemas para manter, testar e depurar |
| **Explicabilidade degradada** | Combinar escores dilui a justificativa |
| **Calibração** | O `alpha` precisa de justificativa — chutar é indefensável |
| **Avaliação mais difícil** | Qual componente causou a melhora? |

> [!tip] Comece simples e hibridize com motivo
> Um sistema de conteúdo bem calibrado supera um híbrido mal ajustado. A hibridização deve ser resposta a um problema **medido**, não a uma vontade de sofisticação. Se você não consegue nomear a falha que o híbrido corrige, não hibridize ainda.

---

## 📚 Referências fundamentais

- **Burke, R. (2002)** — *Hybrid Recommender Systems: Survey and Experiments*, User Modeling and User-Adapted Interaction — a taxonomia das sete estratégias
- **Burke, R. (2007)** — *Hybrid Web Recommender Systems*, in *The Adaptive Web*
- **Çano & Morisio (2017)** — *Hybrid Recommender Systems: A Systematic Literature Review*

---

## 🔗 Conceitos relacionados

- [[rec-filtragem-conteudo|📄 Filtragem baseada em conteúdo]] · [[rec-filtragem-colaborativa|👥 Filtragem colaborativa]]
- [[rec-cold-start|🥶 Cold start]] · [[rec-metricas-avaliacao|📊 Métricas de avaliação]]

## Veja também

- [[Conceitos|🧠 Conceitos]]

---

**Status:** ✅ Completo
