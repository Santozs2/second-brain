---
title: "Camada de IA no quiz — decisão do grupo (Desenho B modificado)"
aliases: ["Camada de IA TCC", "LLM no TCC", "Decisão IA quiz"]
tags: [tcc, ia, llm, arquitetura, decisao, orcamento, gestao]
status: decidido
projeto: TCC
criado: 2026-08-19
atualizado: 2026-08-20
---

> [!info] Projeto: [[TCC|🎓 TCC]] · Plano: [[camada-ia-plano-implementacao|🧩 Plano de implementação]] · Prompt: [[prompt-padrao-recomendacao|📝 Prompt padrão]] · Base: [[engine-matching-cosseno|Engine de cosseno]]

# 🤖 Camada de IA no quiz — decisão do grupo

> [!success] ✅ Decidido em 2026-08-20 — Desenho B modificado
> A engine de cosseno seleciona os **top-5**; a **LLM entrega os cursos** ao usuário, podendo **reordenar dentro desses 5**, através de um **prompt padrão único**. A tela passa a mostrar **1 curso principal + 4 alternativas**. Provedor padrão: **Gemini Flash (free tier)**, plugável por `.env`.
> **O como está em [[camada-ia-plano-implementacao|🧩 Plano de implementação da camada de IA]].** O texto do prompt está em [[prompt-padrao-recomendacao|📝 Prompt padrão]].

> [!abstract] O que estava em jogo
> Hoje o quiz recomenda cursos por **similaridade de cosseno** (`quiz/engine.py`), 100% determinístico e testado. A proposta era acrescentar uma **camada de IA (LLM)** que lesse as respostas e recomendasse o curso. Esta nota levantou as opções, custos e riscos — e agora **registra a decisão tomada**. O levantamento abaixo fica preservado de propósito: é ele que sustenta o capítulo de metodologia (*por que B e não A ou C*).

## 🎯 O que já existe (ponto de partida)

| Peça | Estado |
|---|---|
| Modelagem (áreas, cursos, pesos) | ✅ concluída |
| Engine de cosseno + `explain()` | ✅ concluída, 12 testes automatizados |
| API DRF (`/api/quiz/`) | ✅ concluída |
| Site (wizard + página de resultado) | ✅ concluído |
| **Camada de IA** | 🟡 **decidida — em implementação (Passos 7 a 11)** |

> [!important] O TCC já funciona sem IA
> Isso é uma vantagem de negociação, não um detalhe. A camada de IA é **incremento**, não dependência. Qualquer desenho escolhido tem que preservar essa propriedade.

---

## 🧭 Os três desenhos possíveis

### Desenho A — IA substitui a engine
Manda perguntas, respostas e o catálogo inteiro pro LLM; ele escolhe o curso.

- ✅ Simples de implementar, texto de saída muito bom
- ❌ Descarta o núcleo defensável (álgebra vetorial, explicabilidade, testes)
- ❌ Não é reprodutível — mesma entrada pode dar saída diferente
- ❌ Pode "alucinar" um curso que não existe no catálogo
- ❌ Sem internet ou sem crédito, o sistema **para**

### Desenho B — Híbrido (engine + LLM) ✅ **ESCOLHIDO**
Engine ranqueia e devolve os **top-5**; o LLM lê o perfil + esses 5 + o `explanation` e faz **reordenação do topo + justificativa em linguagem natural**.

- ✅ Aproveita tudo que já foi construído
- ✅ Fallback natural: falhou a IA, mostra o resultado da engine
- ✅ Prompt pequeno e barato (5 cursos, não o catálogo todo)
- ✅ Rende o **capítulo comparativo** da monografia (engine pura × híbrido)
- ⚠️ Mais peças para manter (provider, prompt, cache, fallback)

> [!success] As três modificações que o grupo fez sobre o B original
> **1. A LLM é a voz da entrega.** Não é um bloco extra ao lado do ranking: a tela de resultado passa a ser o texto dela. O ranking da engine continua visível, mas em "por que este curso".
> **2. Formato fixo 1 + 4.** Um curso principal com justificativa desenvolvida e quatro alternativas em uma ou duas frases — em vez de cinco parágrafos de peso igual.
> **3. Prompt padrão único.** Um só prompt, versionado em arquivo, para todos os perfis. É o que torna a comparação do experimento válida — variando o perfil e mantendo o prompt, a diferença observada é atribuível ao perfil.
>
> O poder de reordenar continua **limitado ao top-5** e agora tem um limiar declarado: só troca o 1º lugar em diferença de score menor que 0,05 (regra 2 do [[prompt-padrao-recomendacao|📝 prompt padrão]]).

### Desenho C — IA só redige o texto
A engine decide 100% do ranking; o LLM só transforma o `explanation` num parágrafo de orientador profissional.

- ✅ Risco quase zero, implementação de um dia
- ✅ Ganho de UX real (o texto atual é técnico)
- ❌ Contribuição científica magra — "usei IA para formatar texto"

### Matriz de decisão

| Critério | A (IA decide) | B (híbrido) | C (IA redige) |
|---|:---:|:---:|:---:|
| Aproveita o que já foi feito | ❌ | ✅ | ✅ |
| Reprodutível na banca | ❌ | ✅ | ✅ |
| Funciona offline (fallback) | ❌ | ✅ | ✅ |
| Rende capítulo de resultados | ⚠️ | ✅ | ❌ |
| Esforço de implementação | Baixo | Médio | Baixo |
| Risco de alucinação | Alto | Baixo | Nulo |

> [!tip] Recomendação do orientador — acatada
> **Desenho B.** É o único que aumenta a contribuição científica sem aumentar o risco de defesa. O grupo deve discutir se topa o esforço médio — se o prazo estiver apertado, **C é um recuo honesto** e pode virar B depois (o código do provider é o mesmo).

> [!note] O recuo para C continua disponível
> Como a LLM só pode reordenar dentro do top-5 e sob limiar, **desligar a regra 2 do prompt transforma o B modificado em C** sem tocar em arquitetura: a IA passa a só redigir a ordem da engine. Se o calendário apertar em cima do Passo 11, esse é o plano de recuo — e ele custa uma linha de prompt, não uma refatoração.

---

## 💰 Orçamento

Volume estimado do TCC inteiro: **300 a 500 recomendações** (desenvolvimento, testes com colegas, defesa). Prompt de ~2.000 tokens de entrada e ~400 de saída por recomendação.

| Provedor | Custo por recomendação | 500 usos | Exige cartão internacional? |
|---|---|---|---|
| **Google Gemini Flash** (free tier) | R$ 0 | **R$ 0** | ❌ só conta Google |
| Groq / OpenRouter (modelos free) | R$ 0 | R$ 0 | ❌ |
| Ollama local (Llama, Qwen) | R$ 0 | R$ 0 | ❌ (exige RAM/GPU) |
| Claude Haiku | ~R$ 0,02 | ~R$ 10 | ✅ sim |
| Claude Sonnet | ~R$ 0,08 | ~R$ 40 | ✅ sim |

> [!warning] Assinatura ≠ API
> Assinatura Claude Pro/Max ou ChatGPT Plus **não libera API key**. Serve para usar o chat, não para a aplicação chamar programaticamente. A API é cobrada à parte, em crédito pré-pago com cartão internacional.

> [!success] Conclusão de orçamento — confirmada na decisão
> O custo em dinheiro é irrelevante (< R$ 40 no pior caso). O gargalo real é **meio de pagamento**. Como o grupo não tem cartão internacional garantido, o padrão é **Gemini Flash free tier**, com o código escrito de forma que trocar de provedor seja mudar uma linha do `.env`. **Custo previsto do TCC: R$ 0.**

---

## 🧱 Stack proposta (sem ferramenta nova)

> [!info] Versão final da stack: [[camada-ia-plano-implementacao|🧩 Plano de implementação]]
> A tabela abaixo é o esboço que embasou a decisão. Ao detalhar o plano, três coisas mudaram: `quiz/llm.py` virou o pacote `quiz/llm/` (para o `FakeProvider` rodar sem o SDK instalado), o prompt passou a se chamar `entrega_v1.md` (o papel da LLM é entregar, não recomendar) e os metadados do experimento foram para o `QuizAttempt`, que é onde eles de fato pertencem.

| Peça | Arquivo | Papel |
|---|---|---|
| Interface do provedor | `quiz/llm.py` | Protocolo `LLMProvider` + `GeminiProvider`, `AnthropicProvider`, `FakeProvider` |
| Prompt versionado | `quiz/prompts/recommend_v1.md` | Prompt em arquivo, não string solta — vira anexo da monografia |
| Validação da saída | serializer DRF | LLM devolve JSON; só aceita `course_id` que veio no prompt |
| Registro do experimento | campos em `Recommendation` | `llm_rationale`, `llm_model`, `prompt_version`, `latency_ms`, tokens, `used_fallback` |

> [!important] O `FakeProvider` não é firula
> Ele permite escrever e testar **toda** a camada de IA offline, de graça, e mantém os 12 testes atuais rodando sem chamar rede. É o que garante que o TCC não depende de crédito para ser desenvolvido.

> [!note] Decisões técnicas de apoio
> - `temperature = 0` + **cache por hash das respostas** → mesma entrada, mesma saída (reprodutibilidade na banca)
> - Timeout curto (~8s) e, se estourar, cai na engine sem o usuário perceber
> - Chave de API só no backend (`.env`), **nunca** no template ou no JS

---

## ⚠️ Riscos e mitigações

| Risco | Impacto | Mitigação |
|---|---|---|
| Sem internet no dia da defesa | 🔴 Crítico | Fallback pra engine + cache de resultados pré-gerados |
| Crédito/free tier esgotado | 🟠 Alto | Provedor plugável + fallback |
| LLM inventa curso inexistente | 🟠 Alto | Só pode escolher entre os IDs enviados; validação no servidor |
| Resultado não reprodutível | 🟠 Alto | `temperature=0`, cache, saída salva no banco |
| Latência de 3–5s incomodar | 🟡 Médio | Mostra o resultado da engine na hora, texto da IA carrega depois |
| Banca perguntar "por que IA e não só a engine?" | 🟡 Médio | Capítulo comparativo com números (ver Passo 11) |
| LLM promover um curso pior ao 1º lugar | 🟠 Alto | Novo no B modificado: limiar de 0,05 no prompt + `diverged` gravado + avaliação humana |

---

## 🗓️ Cronograma

> [!warning] Numeração corrigida — a camada de IA é o Passo **7 a 11**
> Esta nota nascera numerando os passos de IA como "6 a 10", mas o **Passo 6 já existe e é o front em templates**, concluído ([[guia-tcc-quiz-perfil|🏗️ Guia de implementação]]). A numeração válida é a de baixo, e o detalhamento de cada passo está em [[camada-ia-plano-implementacao|🧩 Plano de implementação]].

| Passo | Entrega | Depende de crédito/internet? |
|---|---|---|
| **7** | `quiz/llm/` + `FakeProvider` + contratos + testes | ❌ não |
| **8** | Prompt padrão v1 + validação da saída | ❌ não |
| **9** | `GeminiProvider` + cache + timeout + persistência dos metadados | ✅ sim |
| **10** | Tela de entrega: 1 principal + 4 alternativas + selo de fallback | ❌ não |
| **11** | Experimento comparativo (N perfis: engine × híbrido) + capítulo | ✅ sim |

> [!tip] Por onde começar
> **Passo 7 primeiro**, sempre. É totalmente offline, não gasta nada e blinda o resto: quando o grupo fechar o provedor, é só plugar. Note que **três dos cinco passos não dependem de credencial nenhuma** — a falta de chave de API não pode ser desculpa para o cronograma parar.

---

## ✅ As perguntas, respondidas

| # | Pergunta | Resposta do grupo |
|---|---|---|
| 1 | Desenho A, B ou C? | **B modificado** — engine seleciona, LLM entrega e pode reordenar o top-5 |
| 2 | Alguém tem cartão internacional? | Não é necessário — padrão **Gemini Flash free tier**, provedor plugável |
| 3 | A IA muda a ordem ou só explica? | **Muda**, dentro do top-5 e só em diferença de score < 0,05 |
| 4 | O experimento vira capítulo próprio? | **Sim** — é o Passo 11 e a resposta à pergunta "por que IA?" na banca |
| 5 | Quem guarda a credencial? | ⬜ **em aberto** — definir antes do Passo 9 |

> [!todo] Únicos pontos ainda em aberto
> - **Responsável pela credencial** da API (pergunta 5) — não bloqueia os Passos 7, 8 e 10.
> - **N do experimento** do Passo 11: quantos perfis rodar e quem faz a avaliação humana das divergências.
> - **v2 do prompt**: se a confiança do texto deve ser modulada quando o score do curso principal for baixo — decidir depois de ver saídas reais.

---

## 📎 Veja também

- [[camada-ia-plano-implementacao|🧩 Plano de implementação da camada de IA]] — o "como" desta decisão
- [[prompt-padrao-recomendacao|📝 Prompt padrão de entrega]] — o texto e o porquê de cada regra
- [[engine-matching-cosseno|Engine de matching por cosseno]]
- [[api-quiz-drf|API do quiz em DRF]]
- [[guia-tcc-quiz-perfil|Guia de passos do TCC]]
- [[defesa-monografia-tcc|Defesa e monografia]]
- [[Projetos]] · [[Home]]
