---
type: concept
area: Conceitos
status: estavel
difficulty: beginner
id: met-metodo-cientifico
category: Metodologia Científica
tags:
  - metodologia
  - concept
created: 2026-08-24
updated: 2026-08-24
---
# 🔬 Método Científico

> Um procedimento para produzir afirmações que outra pessoa pode verificar. A diferença entre "funciona" e "funcionou nas condições X, medido por Y".

---

## 📖 O ciclo

```
Observação  →  Pergunta  →  Hipótese  →  Método  →  Coleta
                  ↑                                    ↓
                  └──────  Conclusão  ←──  Análise  ───┘
```

O ciclo é **iterativo**. Conclusão que gera pergunta nova não é fracasso — é o funcionamento normal.

---

## 🧱 Os elementos

### Pergunta de pesquisa
Uma pergunta é boa quando é **específica**, **respondível com os recursos disponíveis** e **não respondível por opinião**.

```
❌ "Inteligência artificial melhora a educação?"        (amplo demais)
❌ "Nosso sistema é bom?"                                (não é verificável)
✅ "A adição de uma camada de linguagem natural altera
   a ordem das recomendações produzidas pelo cálculo,
   e em que proporção?"                                  (mensurável)
```

### Hipótese
Uma afirmação que pode ser **contrariada** pelos dados. Se nenhum resultado possível a refutaria, ela não é uma hipótese.

```
H₀ (nula):        não há diferença entre os dois caminhos
H₁ (alternativa): há diferença mensurável na ordenação
```

### Variáveis

| Tipo | O que é |
|---|---|
| **Independente** | O que você manipula |
| **Dependente** | O que você mede |
| **Controlada** | O que você mantém fixo para não contaminar |
| **Interveniente** | O que afeta o resultado sem estar sob controle |

> [!important] Variável não controlada é a origem da maior parte das conclusões erradas
> Se o prompt muda entre execuções, você não está comparando dois sistemas — está comparando duas coisas diferentes. **Listar explicitamente o que foi mantido fixo** é o que dá validade a uma comparação. Ver [[met-validade-e-limitacoes|🎯 Validade e limitações]].

---

## 🎯 Reprodutibilidade

Uma afirmação científica precisa ser verificável por terceiros. Em trabalho de software, isso exige registrar:

- [ ] Versões exatas (linguagem, framework, bibliotecas, modelo)
- [ ] Dados de entrada, ou o procedimento que os gerou
- [ ] Parâmetros e sementes aleatórias
- [ ] Ambiente de execução
- [ ] Código publicado ou anexado
- [ ] Data de execução (quando há dependência de serviço externo)

> [!warning] Sistemas com dependência externa são reprodutíveis apenas em parte
> Se o resultado depende de uma API de terceiros que muda sem versionamento, a reprodução exata é impossível. A saída honesta: **registrar a data, persistir as saídas brutas e declarar a limitação** — não fingir determinismo que não existe.

---

## 🚨 Erros de raciocínio comuns

| Falácia | Exemplo em trabalho de software |
|---|---|
| **Correlação ⇒ causa** | "Aumentou o uso depois da mudança, logo a mudança causou" |
| **Viés de confirmação** | Só testar os casos que você espera que funcionem |
| **Generalização apressada** | 12 colegas aprovaram, logo o sistema é bom |
| **Ausência de baseline** | "Teve 80% de acerto" — comparado com o quê? |
| **Escolha posterior da métrica** | Rodar tudo e depois escolher a métrica que ficou bonita |
| **Sobrevivência** | Analisar só quem terminou o quiz, ignorando quem desistiu |

> [!tip] Defina a métrica de sucesso ANTES de coletar
> Escrever "o sistema será considerado adequado se X" antes da coleta impede a racionalização posterior. É a versão prática do pré-registro usado em ciência experimental, e custa cinco minutos.

---

## 🧭 Baseline: o que torna um número significativo

Um resultado isolado não informa nada. Ele precisa de referência:

| Tipo de baseline | Exemplo |
|---|---|
| **Aleatório** | Recomendar itens ao acaso |
| **Trivial** | Recomendar sempre o mais popular |
| **Estado anterior** | Como era feito antes do sistema |
| **Alternativa da literatura** | Outra técnica publicada |

> "O sistema acertou 78%" não diz nada.
> "O sistema acertou 78%, contra 31% da linha de base aleatória e 52% da regra de popularidade" **diz tudo**.

---

## 📚 Referências

- **Popper, K. (1959)** — *The Logic of Scientific Discovery* — o critério de falseabilidade
- **Gil, A. C. (2002)** — *Como Elaborar Projetos de Pesquisa*
- **Marconi & Lakatos (2003)** — *Fundamentos de Metodologia Científica*
- **Wohlin et al. (2012)** — *Experimentation in Software Engineering*

---

## 🔗 Conceitos relacionados

- [[met-tipos-de-pesquisa|🔬 Tipos de pesquisa]] · [[met-validade-e-limitacoes|🎯 Validade e limitações]]
- [[met-estrutura-monografia|📄 Estrutura da monografia]] · [[rec-metricas-avaliacao|📊 Métricas de avaliação]]

## Veja também

- [[Conceitos|🧠 Conceitos]]

---

**Status:** ✅ Completo
