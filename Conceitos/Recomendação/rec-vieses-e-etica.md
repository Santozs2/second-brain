---
type: concept
area: Conceitos
status: estavel
difficulty: advanced
id: rec-vieses-e-etica
category: Recomendação
tags:
  - recomendacao
  - concept
  - etica
created: 2026-08-24
updated: 2026-08-24
---
# ⚖️ Vieses e Ética em Recomendação

> Um sistema de recomendação distribui oportunidade. Quando ele erra sistematicamente contra um grupo, o erro deixa de ser técnico.

---

## 📖 Por que isto é capítulo obrigatório

Recomendar filme errado custa uma noite ruim. Recomendar **trajetória educacional ou profissional** errada custa anos de vida de alguém. Quanto maior a consequência da recomendação, maior a obrigação de auditar o viés.

Em contexto institucional — orientação de curso, triagem de vaga, alocação de bolsa — o sistema participa de uma **decisão sobre o futuro de uma pessoa**.

---

## 🔍 O catálogo de vieses

### 1. Viés de popularidade
Itens populares são recomendados mais, ficam mais populares e são recomendados ainda mais. Um ciclo que se auto-alimenta.

**Consequência:** a cauda longa fica invisível. Cursos pequenos, mas adequados, nunca aparecem.
**Como medir:** distribuição de exposição por item; índice de Gini sobre as recomendações.

### 2. Viés de dados históricos
O sistema aprende com o passado. Se historicamente poucas mulheres cursaram mecânica, um modelo treinado nesse histórico **aprende a não recomendar mecânica para mulheres**.

> [!warning] Este é o viés mais perigoso porque parece acurácia
> O modelo está "certo" estatisticamente: ele prediz corretamente o padrão histórico. Mas reproduzir fielmente uma desigualdade é **perpetuá-la**. Acurácia alta e justiça são objetivos diferentes, e otimizar o primeiro pode piorar o segundo.

### 3. Viés de apresentação (*position bias*)
O que aparece em primeiro lugar recebe mais cliques **porque** está em primeiro lugar. O feedback resultante confirma a decisão original, criando evidência circular.

### 4. Viés de seleção
Você só observa dados de quem interagiu. Quem desistiu na terceira pergunta do quiz não está na sua base — e pode ser exatamente o grupo mal atendido.

### 5. Bolha de filtro
O sistema mostra mais do que a pessoa já demonstrou gostar, estreitando o horizonte ao longo do tempo. Termo cunhado por *Eli Pariser (2011)*.

### 6. Viés do avaliador
Quando pesos são atribuídos por julgamento humano, os vieses de quem atribuiu entram no modelo — silenciosamente e sem registro.

---

## 🛠️ Como auditar

```python
def auditar_exposicao(recomendacoes_por_usuario, catalogo):
    """Quantos itens do catálogo alguma vez aparecem? Quão concentrado?"""
    from collections import Counter
    contagem = Counter(
        item for recs in recomendacoes_por_usuario for item in recs
    )
    cobertura = len(contagem) / len(catalogo)
    top10 = sum(c for _, c in contagem.most_common(10))
    total = sum(contagem.values())
    return {
        "cobertura_catalogo": cobertura,
        "concentracao_top10": top10 / total if total else 0,
        "itens_nunca_recomendados": len(catalogo) - len(contagem),
    }
```

### Auditoria por subgrupo
Repita as métricas de qualidade separadamente por grupo (gênero, faixa etária, região, escolaridade). **Diferença sistemática entre grupos é o achado.**

| Métrica | Grupo A | Grupo B | Diferença |
|---|---|---|---|
| Similaridade média do top-1 | 0,82 | 0,61 | 🚩 0,21 |
| Diversidade do top-5 | 0,45 | 0,44 | ok |
| Cobertura de catálogo | 38% | 12% | 🚩 26 p.p. |

---

## 🧭 Princípios de projeto

| Princípio | Na prática |
|---|---|
| **Não usar atributo sensível como preditor** | Gênero, raça e religião ficam fora do vetor |
| **Cuidado com proxies** | CEP correlaciona com renda e raça; "escola de origem" também |
| **Explicabilidade como requisito** | A pessoa precisa poder discordar → [[rec-explicabilidade\|💡 nota dedicada]] |
| **Recomendação não é prescrição** | Sugerir, com o catálogo completo sempre acessível |
| **Direito de recurso** | Refazer, ajustar e ignorar precisam ser possíveis |
| **Auditoria periódica** | Viés aparece com o tempo, não só no lançamento |

> [!important] Remover o atributo sensível não remove o viés
> Isto se chama *fairness through unawareness* e **não funciona**. Se outras variáveis correlacionam com o atributo removido, o modelo o reconstrói indiretamente. Auditar a saída por subgrupo é obrigatório mesmo quando o atributo não entra no modelo.

---

## 📜 Marco legal brasileiro

**LGPD — Lei 13.709/2018:**

| Artigo | O que garante |
|---|---|
| **Art. 6º** | Princípios: finalidade, adequação, necessidade, transparência, não discriminação |
| **Art. 9º** | Direito a informação clara sobre o tratamento |
| **Art. 18** | Direitos do titular: acesso, correção, eliminação, portabilidade |
| **Art. 20** | **Direito a revisão de decisões automatizadas** e a informação sobre os critérios |

Em sistema educacional, some-se o **ECA** (quando há menores) e as normas institucionais de proteção de dados de estudantes.

> [!success] Como isto vira uma seção forte de monografia
> Uma seção de "considerações éticas" que apenas cita a LGPD é decorativa. Uma que **mostra a auditoria rodada, os números por subgrupo e as decisões de projeto tomadas em resposta** é contribuição real — e é o tipo de coisa que distingue um trabalho na avaliação.

---

## 📚 Referências fundamentais

- **Barocas, Hardt & Narayanan** — *Fairness and Machine Learning: Limitations and Opportunities* — livre em fairmlbook.org
- **O'Neil, C. (2016)** — *Weapons of Math Destruction*
- **Pariser, E. (2011)** — *The Filter Bubble*
- **Abdollahpouri et al. (2019)** — *The Unfairness of Popularity Bias in Recommendation*
- **Ekstrand et al. (2018)** — *All The Cool Kids: Demographics of Recommender Effectiveness*

---

## 🔗 Conceitos relacionados

- [[rec-explicabilidade|💡 Recomendação explicável]] · [[rec-metricas-avaliacao|📊 Métricas de avaliação]]
- [[met-etica-em-pesquisa|🤝 Ética em pesquisa]] · [[rec-sistemas-de-recomendacao|🎯 Sistemas de Recomendação]]

## Veja também

- [[Conceitos|🧠 Conceitos]]

---

**Status:** ✅ Completo
