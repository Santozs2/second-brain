---
type: concept
area: Conceitos
status: estavel
difficulty: advanced
id: rec-metricas-avaliacao
category: Recomendação
tags:
  - recomendacao
  - concept
  - metricas
created: 2026-08-24
updated: 2026-08-24
---
# 📊 Métricas de Avaliação de Recomendação

> Como provar que a recomendação é boa. Acurácia sozinha não prova — e otimizar só por ela produz sistemas tecnicamente corretos e inúteis.

---

## 🎯 As três famílias de avaliação

| Tipo | Como se faz | Custo | O que mede |
|---|---|---|---|
| **Offline** | Dataset histórico, métricas automáticas | Baixo | Acurácia de predição |
| **Estudo com usuários** | Pessoas usam e respondem | Alto | Satisfação, confiança, utilidade |
| **Online (teste A/B)** | Duas versões em produção | Médio | Comportamento real |

> [!important] Avaliação offline mede o passado, não a utilidade
> Ela só consegue premiar o sistema quando ele acerta o que a pessoa **já escolheu**. Um sistema que recomenda algo excelente que a pessoa nunca viu é **penalizado** por isso. É a limitação estrutural da avaliação offline, e por isso ela nunca deve ser a única.

---

## 1️⃣ Métricas de ranking (as que importam em recomendação)

Recomendação entrega uma **lista ordenada**, e ninguém olha além das primeiras posições. Por isso as métricas são cortadas em `@k`.

### Precision@k
Dos `k` itens recomendados, quantos eram relevantes?

$$P@k = \frac{|\text{relevantes} \cap \text{recomendados}_k|}{k}$$

### Recall@k
Dos itens relevantes que existiam, quantos apareceram no top-k?

$$R@k = \frac{|\text{relevantes} \cap \text{recomendados}_k|}{|\text{relevantes}|}$$

### MRR — Mean Reciprocal Rank
Em que posição apareceu o primeiro acerto? Ideal quando existe **uma** resposta certa.

$$MRR = \frac{1}{|Q|}\sum_{i=1}^{|Q|} \frac{1}{\text{rank}_i}$$

Primeiro acerto na posição 1 → `1,0`. Na posição 5 → `0,2`.

### NDCG — Normalized Discounted Cumulative Gain
A métrica mais completa: considera **relevância graduada** (não só sim/não) e **desconta por posição** — acertar na posição 1 vale mais que na posição 10.

$$DCG@k = \sum_{i=1}^{k} \frac{2^{rel_i} - 1}{\log_2(i+1)}, \qquad NDCG@k = \frac{DCG@k}{IDCG@k}$$

Referência: *Järvelin & Kekäläinen (2002)*. É o padrão em publicações da área.

### MAP — Mean Average Precision
Média da precisão calculada em cada posição de acerto, promediada sobre todos os usuários.

---

## 2️⃣ Métricas "além da acurácia"

Estas são as que separam um sistema utilizável de um sistema apenas correto.

| Métrica | O que mede | Por que importa |
|---|---|---|
| **Cobertura de catálogo** | % de itens que o sistema alguma vez recomenda | Se só 5% do catálogo aparece, os outros 95% são invisíveis |
| **Diversidade** | Dissimilaridade média dentro da lista | 5 itens quase idênticos = 1 recomendação |
| **Novidade** | Quão desconhecidos são os itens | Recomendar o óbvio não ajuda ninguém |
| **Serendipidade** | Relevante **e** inesperado | O santo graal da área |
| **Viés de popularidade** | Concentração no topo da cauda | Mede injustiça na exposição |

> [!warning] Existe um trade-off matemático entre acurácia e diversidade
> Maximizar acurácia empurra para recomendar sempre os itens mais seguros e parecidos. Aumentar diversidade **necessariamente** reduz a acurácia medida. Não existe solução que otimize as duas; existe uma **escolha declarada** de onde ficar. Ver *Zhou et al. (2010)*.

---

## 3️⃣ Avaliação com usuários

Quando não existe base histórica, a avaliação com usuários **não é o plano B — é o método correto**.

### Instrumentos consagrados

- **ResQue** (*Pu, Chen & Hu, 2011*) — 15 construtos: qualidade percebida, adequação, transparência, confiança, satisfação, intenção de uso
- **Escala Likert de 5 ou 7 pontos** — padrão para atitude
- **SUS** (*System Usability Scale*) — usabilidade geral, 10 itens

### Perguntas que funcionam

```
1. As recomendações fizeram sentido para você?         (1–5)
2. A justificativa apresentada ficou clara?            (1–5)
3. Você consideraria seguir a primeira recomendação?   (1–5)
4. As opções apresentadas eram variadas o suficiente?  (1–5)
5. Você confiaria neste sistema para orientar alguém?  (1–5)
```

> [!tip] Reporte N, média, desvio-padrão — e o N mínimo honesto
> Com menos de 30 respondentes, evite testes de significância e trate os dados como **exploratórios**. Escrever *"estudo exploratório com N=22, sem pretensão de generalização estatística"* é metodologicamente correto. Escrever *"o sistema tem 89% de aprovação"* com N=9 não é. Ver [[met-validade-e-limitacoes|🎯 Validade e limitações]].

---

## 4️⃣ Avaliação comparativa (A × B)

O desenho mais forte quando existem duas versões do sistema:

```
Mesmos N perfis → Caminho A (baseline) → resultado A
                → Caminho B (proposto) → resultado B
                         ↓
        taxa de divergência · concordância · preferência humana
```

Métricas úteis:
- **Taxa de divergência** — em quantos % dos casos os dois caminhos discordam
- **Concordância de topo** — em quantos % o item nº 1 é o mesmo
- **Kendall's τ / Spearman ρ** — correlação entre os dois rankings completos
- **Preferência humana cega** — avaliadores escolhem sem saber qual é qual

> [!success] Divergência não é erro — é o dado
> Se dois caminhos concordam em 100% dos casos, um deles é desnecessário. Se divergem, **onde e por quê** divergem é o resultado do trabalho. Reportar a taxa e analisar os casos discordantes vale mais que qualquer número de acurácia.

---

## 📚 Referências fundamentais

- **Herlocker, Konstan, Terveen & Riedl (2004)** — *Evaluating Collaborative Filtering Recommender Systems*, ACM TOIS — o artigo fundador da avaliação na área
- **Järvelin & Kekäläinen (2002)** — *Cumulated Gain-Based Evaluation of IR Techniques* — origem do NDCG
- **Pu, Chen & Hu (2011)** — *A User-Centric Evaluation Framework* (ResQue), RecSys
- **Zhou et al. (2010)** — *Solving the Apparent Diversity-Accuracy Dilemma*, PNAS
- **Gunawardana & Shani (2009)** — *A Survey of Accuracy Evaluation Metrics*

---

## 🔗 Conceitos relacionados

- [[rec-sistemas-de-recomendacao|🎯 Sistemas de Recomendação]] · [[rec-explicabilidade|💡 Explicabilidade]]
- [[met-validade-e-limitacoes|🎯 Validade e limitações]] · [[met-tipos-de-pesquisa|🔬 Tipos de pesquisa]]
- [[ia-avaliacao-de-llm|📐 Avaliação de LLM]]

## Veja também

- [[Conceitos|🧠 Conceitos]]

---

**Status:** ✅ Completo
