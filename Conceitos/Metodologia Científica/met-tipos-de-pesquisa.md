---
type: concept
area: Conceitos
status: estavel
difficulty: beginner
id: met-tipos-de-pesquisa
category: Metodologia Científica
tags:
  - metodologia
  - concept
  - tcc
created: 2026-08-24
updated: 2026-08-24
---
# 🔬 Tipos de Pesquisa

> Classificar a pesquisa não é burocracia de formulário: é declarar as regras pelas quais o trabalho quer ser julgado.

---

## 🗂️ As quatro dimensões de classificação

Todo trabalho é classificado simultaneamente em quatro eixos independentes. A taxonomia mais usada no Brasil é a de **Gil (2002)** e **Silva & Menezes (2005)**.

### 1. Quanto à natureza

| Tipo | Descrição |
|---|---|
| **Básica** | Gera conhecimento novo sem aplicação prevista |
| **Aplicada** | Gera conhecimento para resolver um problema concreto |

> TCC de curso técnico ou tecnológico é quase sempre **aplicada**.

### 2. Quanto aos objetivos

| Tipo | Descrição | Pergunta típica |
|---|---|---|
| **Exploratória** | Familiarizar-se com um problema pouco estudado | "O que existe sobre isto?" |
| **Descritiva** | Descrever características de um fenômeno | "Como isto se comporta?" |
| **Explicativa** | Identificar causas | "Por que isto acontece?" |

### 3. Quanto à abordagem

| Tipo | Descrição |
|---|---|
| **Qualitativa** | Interpreta significados; não quantifica |
| **Quantitativa** | Mede, conta, aplica estatística |
| **Quali-quantitativa (mista)** | Combina as duas |

> [!tip] Trabalho de sistema com avaliação de usuário costuma ser misto
> A métrica do algoritmo é quantitativa; a percepção de quem usou é qualitativa. Declarar **mista** é honesto e amplia o que você pode afirmar — desde que os dois lados sejam realmente coletados.

### 4. Quanto aos procedimentos técnicos

| Procedimento | O que é |
|---|---|
| **Bibliográfica** | A partir de material publicado |
| **Documental** | A partir de documentos não tratados analiticamente |
| **Experimental** | Manipula variáveis e observa efeitos, com controle |
| **Levantamento (survey)** | Interroga diretamente um grupo |
| **Estudo de caso** | Estudo profundo de um objeto específico |
| **Pesquisa-ação** | Pesquisador intervém no problema junto dos envolvidos |
| **Pesquisa de desenvolvimento** | Constrói e avalia um artefato |

---

## 🛠️ Design Science Research — o enquadramento certo para quem constrói software

Quando o trabalho **produz um artefato** (sistema, método, modelo), a *Design Science Research* (DSR) é o enquadramento metodológico mais adequado — e é subutilizada em TCC de computação.

Os sete requisitos de *Hevner et al. (2004)*:

| # | Requisito | O que significa |
|---|---|---|
| 1 | Produzir um artefato | Construto, modelo, método ou instanciação |
| 2 | Relevância do problema | O problema importa para alguém real |
| 3 | Avaliação rigorosa | O artefato é avaliado por método declarado |
| 4 | Contribuição clara | O que existe agora que não existia antes |
| 5 | Rigor na pesquisa | Fundamentado na literatura |
| 6 | Design como busca | O processo é iterativo, e as iterações são registradas |
| 7 | Comunicação | Descrito para o público técnico **e** o gerencial |

> [!success] DSR resolve o desconforto de "meu TCC é um sistema, não uma pesquisa"
> Construir um artefato **é** método científico reconhecido, desde que haja avaliação rigorosa e contribuição declarada. Enquadrar o trabalho como DSR transforma "eu fiz um site" em "foi desenvolvido e avaliado um artefato computacional" — e dá um capítulo de metodologia que se sustenta sozinho.

---

## 📝 Como escrever o parágrafo de classificação

O formato esperado, com as quatro dimensões e a citação:

> Quanto à natureza, esta pesquisa classifica-se como **aplicada**, por gerar conhecimento voltado à solução de um problema específico. Quanto aos objetivos, é **descritiva e exploratória**, ao caracterizar o comportamento do artefato desenvolvido em um domínio pouco explorado. Quanto à abordagem, é **quali-quantitativa**, combinando métricas objetivas de desempenho com a percepção de usuários. Quanto aos procedimentos, adota **pesquisa bibliográfica** para a fundamentação e **pesquisa de desenvolvimento** sob o enquadramento da *Design Science Research* (HEVNER et al., 2004) para a construção e avaliação do artefato.

---

## 🎯 Escolher o desenho de avaliação

| Se você quer afirmar... | O desenho precisa ser |
|---|---|
| "A funciona" | Estudo de caso com métricas declaradas |
| "A é melhor que B" | **Comparativo**, com A e B nas mesmas condições |
| "As pessoas preferem A" | Avaliação com usuários, N declarado |
| "A causa B" | Experimental, com controle de variáveis |

> [!warning] Não afirme mais do que o desenho sustenta
> Um estudo de caso não prova superioridade. Uma avaliação com 12 colegas não generaliza para a população. **Cada afirmação da conclusão precisa ser rastreável ao desenho que a sustenta** — e a banca vai testar exatamente isso. Ver [[met-validade-e-limitacoes|🎯 Validade e limitações]].

---

## 📚 Referências fundamentais

- **Gil, A. C. (2002)** — *Como Elaborar Projetos de Pesquisa*, Atlas — a referência mais citada no Brasil
- **Silva & Menezes (2005)** — *Metodologia da Pesquisa e Elaboração de Dissertação*, UFSC — livre online
- **Hevner, March, Park & Ram (2004)** — *Design Science in Information Systems Research*, MIS Quarterly
- **Marconi & Lakatos (2003)** — *Fundamentos de Metodologia Científica*

---

## 🔗 Conceitos relacionados

- [[met-metodo-cientifico|🔬 Método científico]] · [[met-estrutura-monografia|📄 Estrutura da monografia]]
- [[met-validade-e-limitacoes|🎯 Validade e limitações]] · [[met-revisao-bibliografica|📖 Revisão bibliográfica]]

## Veja também

- [[Conceitos|🧠 Conceitos]]

---

**Status:** ✅ Completo
