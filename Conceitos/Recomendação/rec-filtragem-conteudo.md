---
type: concept
area: Conceitos
status: estavel
difficulty: intermediate
id: rec-filtragem-conteudo
category: Recomendação
tags:
  - recomendacao
  - concept
created: 2026-08-24
updated: 2026-08-24
---
# 📄 Filtragem Baseada em Conteúdo

> Recomenda comparando os **atributos** do item com o **perfil declarado** da pessoa. Não precisa de mais ninguém no sistema.

---

## 📖 Definição

A filtragem baseada em conteúdo (*content-based filtering*) descreve item e usuário no **mesmo espaço de atributos** e recomenda por proximidade.

```
Item    → vetor de atributos    [5, 2, 0, 1, ...]
Usuário → vetor de preferências [4, 1, 1, 0, ...]
                    ↓
        similaridade(usuário, item) → ranking
```

Toda a informação necessária está no catálogo e no perfil. **Nenhum outro usuário participa do cálculo.**

---

## 🔄 O ciclo completo

```
1. Representação do item     → extrair atributos (manual, TF-IDF ou embedding)
2. Construção do perfil      → quiz, histórico ou média dos itens curtidos
3. Cálculo de similaridade   → cosseno, euclidiana, Jaccard
4. Ranking                   → ordenar por similaridade decrescente
5. Filtragem                 → cortar em top-N, remover já consumidos
6. Apresentação              → entregar com justificativa
```

---

## ✅ Vantagens

| Vantagem | Por quê |
|---|---|
| **Imune a cold start de usuário** | Perfil vem de declaração, não de histórico → [[rec-cold-start\|🥶 cold start]] |
| **Imune a cold start de item** | Item novo já tem atributos no dia em que entra |
| **Independência entre usuários** | Não precisa de massa crítica; funciona com 1 usuário |
| **Explicável por construção** | A justificativa é literalmente a dimensão que mais contribuiu |
| **Sem privacidade compartilhada** | Não usa dados de terceiros para recomendar a você |
| **Determinístico e auditável** | Mesma entrada → mesma saída, sempre |

---

## ❌ Desvantagens

| Desvantagem | Consequência |
|---|---|
| **Superespecialização** | Só recomenda o que se parece com o que a pessoa já declarou |
| **Sem serendipidade** | Nunca sugere algo surpreendentemente bom fora do perfil |
| **Depende da qualidade dos atributos** | Atributo mal atribuído = recomendação ruim, sem bug nenhum |
| **Perfil estático** | Não capta mudança de interesse sem nova coleta |
| **Não capta qualidade** | Dois itens com atributos idênticos empatam, mesmo que um seja péssimo |

> [!warning] Superespecialização é o defeito assinatura desta abordagem
> Se a pessoa declara interesse em mecânica, ela recebe mecânica para sempre. O sistema nunca descobre que ela também gostaria de automação industrial — porque nenhum dado no modelo sugere isso. Mitigações: injetar diversidade forçada no top-N, ou adicionar uma camada colaborativa quando houver base. Ver [[rec-sistemas-hibridos|🔀 Sistemas híbridos]].

---

## 🧱 Como construir o perfil do usuário

### Explícito (declarado)
A pessoa responde perguntas. Cada resposta contribui com peso para uma ou mais dimensões.

```python
# Perfil por soma ponderada de respostas
perfil = [0.0] * len(areas)
for resposta in respostas:
    for i, peso in enumerate(resposta.pergunta.pesos_por_area):
        perfil[i] += peso * resposta.valor
```

✅ Interpretável, imediato
❌ Reflete o que a pessoa **diz**, não o que ela **faz**

### Implícito (inferido do comportamento)
Média dos vetores dos itens com que a pessoa interagiu.

✅ Reflete comportamento real
❌ Exige histórico — volta ao cold start

### Híbrido
Começa explícito, ajusta com comportamento ao longo do tempo. É o desenho mais robusto e o mais caro.

---

## 🎯 Onde esta abordagem é a escolha certa

- Catálogos **bem descritos** e relativamente estáveis (cursos, vagas, imóveis, seguros)
- Sistemas **novos**, sem base histórica
- Domínios que exigem **justificativa** da recomendação — educação, saúde, crédito, contratação
- Contextos onde recomendar por comportamento de terceiros seria **eticamente problemático**

> [!success] Em domínio institucional, explicabilidade não é bônus — é requisito
> Quando a recomendação afeta a trajetória de alguém (que curso fazer, que vaga tentar), "o algoritmo decidiu" é resposta insuficiente. A filtragem por conteúdo entrega o porquê de graça. Ver [[rec-explicabilidade|💡 Explicabilidade]].

---

## 📚 Referências

- **Pazzani & Billsus (2007)** — *Content-Based Recommendation Systems*, in *The Adaptive Web* — o capítulo de referência
- **Lops, de Gemmis & Semeraro (2011)** — *Content-based Recommender Systems: State of the Art and Trends*
- **Aggarwal, C. C. (2016)** — *Recommender Systems: The Textbook*, cap. 4

---

## 🔗 Conceitos relacionados

- [[rec-modelo-espaco-vetorial|📐 Modelo de Espaço Vetorial]] · [[rec-similaridade-cosseno|📏 Similaridade de cosseno]]
- [[rec-filtragem-colaborativa|👥 Filtragem colaborativa]] — a alternativa
- [[rec-tf-idf|📊 TF-IDF]] · [[rec-cold-start|🥶 Cold start]]

## Veja também

- [[Conceitos|🧠 Conceitos]]

---

**Status:** ✅ Completo
