---
title: "Camada de IA no quiz — levantamento para decisão do grupo"
aliases: ["Camada de IA TCC", "LLM no TCC", "Decisão IA quiz"]
tags: [tcc, ia, llm, arquitetura, decisao, orcamento, gestao]
status: em-discussao
projeto: TCC
criado: 2026-08-19
---

> [!info] Projeto: [[TCC|🎓 TCC]] · Base: [[engine-matching-cosseno|Engine de cosseno]] · Tecnologias: [[Python|Python]] · [[Django|Django]] · [[REST API|REST API]]

# 🤖 Camada de IA no quiz — levantamento para o grupo

> [!abstract] O que está em jogo
> Hoje o quiz recomenda cursos por **similaridade de cosseno** (`quiz/engine.py`), 100% determinístico e testado. A proposta é acrescentar uma **camada de IA (LLM)** que leia as respostas e recomende o curso. Esta nota levanta as opções, custos e riscos para o grupo decidir — **nenhuma decisão foi tomada ainda**.

## 🎯 O que já existe (ponto de partida)

| Peça | Estado |
|---|---|
| Modelagem (áreas, cursos, pesos) | ✅ concluída |
| Engine de cosseno + `explain()` | ✅ concluída, 12 testes automatizados |
| API DRF (`/api/quiz/`) | ✅ concluída |
| Site (wizard + página de resultado) | ✅ concluído |
| **Camada de IA** | ⬜ **em decisão** |

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

### Desenho B — Híbrido (engine + LLM) ⭐
Engine ranqueia e devolve os **top-5**; o LLM lê o perfil + esses 5 + o `explanation` e faz **reordenação do topo + justificativa em linguagem natural**.

- ✅ Aproveita tudo que já foi construído
- ✅ Fallback natural: falhou a IA, mostra o resultado da engine
- ✅ Prompt pequeno e barato (5 cursos, não o catálogo todo)
- ✅ Rende o **capítulo comparativo** da monografia (engine pura × híbrido)
- ⚠️ Mais peças para manter (provider, prompt, cache, fallback)

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

> [!tip] Recomendação do orientador
> **Desenho B.** É o único que aumenta a contribuição científica sem aumentar o risco de defesa. O grupo deve discutir se topa o esforço médio — se o prazo estiver apertado, **C é um recuo honesto** e pode virar B depois (o código do provider é o mesmo).

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

> [!success] Conclusão de orçamento
> O custo em dinheiro é irrelevante (< R$ 40 no pior caso). O gargalo real é **meio de pagamento**. Como o grupo não tem cartão internacional garantido, o padrão deve ser **Gemini free tier**, com o código escrito de forma que trocar de provedor seja mudar uma linha do `.env`.

---

## 🧱 Stack proposta (sem ferramenta nova)

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
| Banca perguntar "por que IA e não só a engine?" | 🟡 Médio | Capítulo comparativo com números (ver Passo 9) |

---

## 🗓️ Cronograma sugerido

| Passo | Entrega | Depende de crédito? |
|---|---|---|
| **6** | `quiz/llm.py` + `FakeProvider` + testes | ❌ não |
| **7** | Prompt v1 + provedor real + persistência dos metadados | ✅ sim |
| **8** | UI: bloco "o que o orientador diz" + indicador de fallback | ❌ não |
| **9** | Experimento comparativo (N perfis: engine × híbrido) | ✅ sim |
| **10** | Documentação no vault + capítulo escrito | ❌ não |

> [!tip] Por onde começar
> **Passo 6 primeiro**, sempre. É totalmente offline, não gasta nada e blinda o resto: quando o grupo fechar o provedor, é só plugar.

---

## ❓ Perguntas para o grupo fechar

1. **Desenho A, B ou C?** (recomendação: B; C se o prazo apertar)
2. **Alguém do grupo tem cartão internacional?** Se não → Gemini free tier
3. A IA pode **mudar a ordem** dos cursos ou só **explicar** a ordem da engine?
4. O experimento comparativo do Passo 9 entra na monografia como capítulo próprio?
5. Quem do grupo fica responsável por **criar e guardar a credencial** da API?

---

## 📎 Veja também

- [[engine-matching-cosseno|Engine de matching por cosseno]]
- [[api-quiz-drf|API do quiz em DRF]]
- [[guia-tcc-quiz-perfil|Guia de passos do TCC]]
- [[defesa-monografia-tcc|Defesa e monografia]]
- [[Projetos]] · [[Home]]
