---
type: concept
area: Conceitos
status: estavel
difficulty: intermediate
id: ia-llm-fundamentos
category: Inteligência Artificial
tags:
  - ia
  - llm
  - concept
created: 2026-08-24
updated: 2026-08-24
---
# 🧠 LLM — Fundamentos

> Um modelo de linguagem prediz o próximo token. Todo o resto — resumir, traduzir, programar, conversar — é consequência dessa única operação repetida.

---

## 📖 O que é

Um **Large Language Model** é uma rede neural treinada para estimar a distribuição de probabilidade do próximo token dado o contexto anterior.

$$P(t_n \mid t_1, t_2, \ldots, t_{n-1})$$

```
Entrada: "O céu está"
         ↓
Distribuição: azul (0,42) · nublado (0,18) · escuro (0,09) · ...
         ↓
Amostragem → "azul"
         ↓
Repete com "O céu está azul" como novo contexto
```

> [!important] O modelo não sabe se está certo — ele sabe o que é provável
> Não existe, dentro do modelo, uma separação entre "verdade" e "texto plausível". Ele otimiza plausibilidade. Isso explica de uma vez por que LLMs escrevem bem e por que inventam com confiança. Ver [[ia-alucinacao-e-grounding|🎭 Alucinação e grounding]].

---

## 🏗️ A arquitetura: Transformer

Introduzida em *Vaswani et al. (2017)* — *Attention Is All You Need*. A peça central é o mecanismo de **atenção**, que permite a cada token consultar todos os outros e pesar quais importam.

$$\text{Attention}(Q,K,V) = \text{softmax}\left(\frac{QK^T}{\sqrt{d_k}}\right)V$$

O que mudou em relação ao que veio antes (RNN, LSTM):

| | RNN / LSTM | Transformer |
|---|---|---|
| Processamento | Sequencial | **Paralelo** |
| Dependências longas | Degradam | Acesso direto |
| Treino | Lento | Escalável em GPU |

O paralelismo é o que tornou viável treinar em escala — e a escala é o que produziu as capacidades emergentes.

---

## 🔤 Tokens

O modelo não vê letras nem palavras: vê **tokens**, unidades de subpalavra produzidas por um tokenizador (BPE ou similar).

```
"desenvolvimento"  →  ["desenvolv", "imento"]        2 tokens
"cosseno"          →  ["cos", "seno"]                2 tokens
"the"              →  ["the"]                        1 token
```

Regra prática: **em português, ~1 token para cada 3 a 4 caracteres.** Português consome mais tokens que inglês para o mesmo conteúdo, porque os tokenizadores foram otimizados majoritariamente em inglês.

Isso importa por três razões: **custo** (cobra-se por token), **limite de contexto** (a janela é medida em tokens) e **latência** (mais tokens de saída = resposta mais lenta).

---

## 🎛️ Parâmetros de geração

| Parâmetro | O que faz | Quando ajustar |
|---|---|---|
| **temperature** | Achata ou aguça a distribuição | `0` para determinismo; `0.7+` para criatividade |
| **top_p** | Amostra do menor conjunto que soma p | Alternativa à temperatura |
| **max_tokens** | Teto da resposta | Controle de custo e de corte |
| **stop** | Sequências que encerram a geração | Formatos delimitados |
| **seed** | Semente de amostragem | Reprodutibilidade (quando suportado) |

> [!warning] `temperature=0` reduz a variação, mas não garante reprodutibilidade
> Mesmo com temperatura zero, o mesmo prompt pode gerar saídas diferentes: paralelismo de GPU, versão do modelo e roteamento interno introduzem não-determinismo. **Para trabalho experimental, registre o identificador exato do modelo e a data da execução** — e não prometa reprodutibilidade que a plataforma não oferece. Ver [[met-validade-e-limitacoes|🎯 Validade e limitações]].

---

## 🪟 Janela de contexto

O total de tokens que o modelo processa de uma vez (prompt + resposta). Ultrapassar o limite gera erro ou truncamento silencioso.

Contexto grande não é gratuito: o custo de atenção cresce quadraticamente com o comprimento, e modelos degradam na recuperação de informação no **meio** de contextos longos — fenômeno documentado como *lost in the middle* (*Liu et al., 2023*). Informação crítica deve ir no **começo ou no fim** do prompt.

---

## ⚠️ Limitações estruturais

| Limitação | Detalhe |
|---|---|
| **Corte de conhecimento** | Não sabe nada posterior ao treino |
| **Sem estado** | Cada chamada é independente; "memória" é o histórico reenviado |
| **Alucinação** | Inventa com fluência → [[ia-alucinacao-e-grounding\|🎭 nota dedicada]] |
| **Aritmética frágil** | Prediz tokens, não calcula |
| **Sensível ao fraseado** | Mudanças pequenas no prompt alteram a saída |
| **Não-determinístico** | Dificulta teste automatizado |

> [!tip] A consequência arquitetural: nunca delegue o cálculo ao modelo
> Se um número precisa estar certo, ele deve ser **computado em código** e entregue pronto ao modelo. O papel do LLM é redigir, classificar e reformular — não decidir nem calcular. Essa fronteira é o que separa um sistema confiável de um sistema que funciona na demonstração.

---

## 📚 Referências fundamentais

- **Vaswani et al. (2017)** — *Attention Is All You Need*, NeurIPS
- **Brown et al. (2020)** — *Language Models are Few-Shot Learners* (GPT-3)
- **Bommasani et al. (2021)** — *On the Opportunities and Risks of Foundation Models*, Stanford CRFM
- **Liu et al. (2023)** — *Lost in the Middle: How Language Models Use Long Contexts*

---

## 🔗 Conceitos relacionados

- [[ia-engenharia-de-prompt|📝 Engenharia de prompt]] · [[ia-alucinacao-e-grounding|🎭 Alucinação e grounding]]
- [[ia-embeddings|🧬 Embeddings]] · [[ia-tokens-e-custo|💰 Tokens e custo]]
- [[ia-avaliacao-de-llm|📐 Avaliação de LLM]]

## Veja também

- [[Conceitos|🧠 Conceitos]]

---

**Status:** ✅ Completo
