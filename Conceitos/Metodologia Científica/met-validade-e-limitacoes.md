---
type: concept
area: Conceitos
status: estavel
difficulty: intermediate
id: met-validade-e-limitacoes
category: Metodologia Científica
tags:
  - metodologia
  - concept
  - tcc
created: 2026-08-24
updated: 2026-08-24
---
# 🎯 Validade e Limitações

> A banca vai encontrar as fraquezas do trabalho. A única variável sob seu controle é se elas já estavam escritas quando isso acontecer.

---

## 📖 Os quatro tipos de validade

Taxonomia de *Wohlin et al. (2012)*, padrão em engenharia de software experimental:

| Tipo | Pergunta | Ameaça típica |
|---|---|---|
| **De construto** | Você mediu o que dizia medir? | A métrica não representa o conceito |
| **Interna** | A relação observada é causal? | Variável não controlada explica o efeito |
| **Externa** | O resultado generaliza? | Amostra pequena ou não representativa |
| **De conclusão** | A análise sustenta a conclusão? | Estatística mal aplicada, N insuficiente |

---

## 🔍 Ameaças comuns em trabalho de software

### Validade de construto

| Ameaça | Exemplo |
|---|---|
| Métrica desalinhada do conceito | Medir "satisfação" por tempo de uso |
| Instrumento não validado | Questionário inventado, sem base na literatura |
| Viés de expectativa | Quem avalia sabe qual sistema é o "novo" |

**Mitigação:** usar instrumentos consagrados (ResQue, SUS, Likert), aplicar avaliação cega.

### Validade interna

| Ameaça | Exemplo |
|---|---|
| Efeito de aprendizado | O participante melhora só por já ter feito antes |
| Efeito de ordem | O primeiro sistema testado leva vantagem |
| Seleção enviesada | Só participaram os já interessados no tema |

**Mitigação:** randomizar a ordem, contrabalancear, declarar como os participantes foram recrutados.

### Validade externa

| Ameaça | Exemplo |
|---|---|
| Amostra por conveniência | Só colegas de turma |
| Dados sintéticos | Perfis criados pelos autores, não coletados |
| Ambiente artificial | Teste em laboratório, não em uso real |
| Escala reduzida | 18 itens quando o real teria 180 |

**Mitigação:** honestamente, quase nenhuma dentro de um TCC. **Declare.**

### Validade de conclusão

| Ameaça | Exemplo |
|---|---|
| N pequeno | Afirmar tendência com 9 respostas |
| Teste estatístico inadequado | Aplicar teste paramétrico em dado ordinal |
| Comparação sem baseline | "Foi bom" sem referência de comparação |

---

## ✍️ Como escrever a seção de limitações

**A regra:** cada limitação vem com (1) o que é, (2) por que existe, (3) o que ela impede de afirmar, (4) o que faria para resolver.

> **Escala do catálogo.** A base foi recortada em 18 cursos representativos das 7 áreas, enquanto o catálogo institucional completo contém aproximadamente 180. O recorte decorre do custo de atribuição manual de pesos, que cresce com o produto entre itens e dimensões. Em consequência, os resultados **não permitem afirmar** o comportamento do sistema em escala completa, particularmente quanto à ocorrência de empates no topo do ranking. A superação desta limitação exige um método automatizado de derivação de pesos a partir das ementas, o que se propõe como trabalho futuro.

> [!success] Uma limitação bem escrita é uma demonstração de domínio
> O parágrafo acima mostra que o autor entende o custo computacional do método, a diferença entre o recorte e o universo, e a consequência específica sobre os resultados. **Isso vale mais do que ter usado 180 cursos sem entender por que 18 seriam suficientes.**

---

## 🚫 O que não fazer

| Antipadrão | Por que falha |
|---|---|
| Não declarar limitação nenhuma | Sinaliza que o autor não enxerga o próprio trabalho |
| Limitação genérica ("faltou tempo") | Não informa nada |
| Limitação disfarçada de qualidade | "Nossa única limitação é sermos muito rigorosos" |
| Esconder resultado desfavorável | Se descoberto, compromete o trabalho inteiro |
| Confundir limitação com trabalho futuro | Limitação restringe o que **este** trabalho afirma |

---

## 📊 Quando você pode e quando não pode usar estatística

| N | O que é honesto afirmar |
|---|---|
| < 10 | Relato qualitativo; sem números percentuais |
| 10–29 | Exploratório; média e desvio, **sem** teste de significância |
| 30–99 | Testes não-paramétricos; generalização limitada e declarada |
| 100+ | Testes paramétricos, conforme a distribuição |

> [!warning] Percentual com N pequeno mente por construção
> "78% aprovaram" com N=9 significa que 7 pessoas gostaram. **Reporte sempre o N junto do percentual** — e com menos de 30, prefira o número absoluto. Escrever "7 dos 9 participantes" é mais honesto e mais difícil de atacar.

---

## 🎤 Como isto vira defesa

Quando a banca aponta uma limitação que **já está escrita**, a resposta é curta e forte:

> *"Sim, essa limitação está declarada na seção 6.2. Ela decorre de [causa], impede que se afirme [o quê], e a proposta de superação é [o quê]."*

A pergunta deixa de ser um ataque e vira a confirmação de que o autor conhece o próprio trabalho. Ver [[met-defesa-banca|🎤 Defesa de banca]].

---

## 📚 Referências

- **Wohlin et al. (2012)** — *Experimentation in Software Engineering*, Springer — cap. 8, a referência sobre ameaças à validade
- **Campbell & Stanley (1963)** — *Experimental and Quasi-Experimental Designs for Research*
- **Runeson & Höst (2009)** — *Guidelines for Conducting and Reporting Case Study Research in Software Engineering*

---

## 🔗 Conceitos relacionados

- [[met-tipos-de-pesquisa|🔬 Tipos de pesquisa]] · [[met-defesa-banca|🎤 Defesa de banca]]
- [[met-estrutura-monografia|📄 Estrutura da monografia]] · [[rec-metricas-avaliacao|📊 Métricas de avaliação]]

## Veja também

- [[Conceitos|🧠 Conceitos]]

---

**Status:** ✅ Completo
