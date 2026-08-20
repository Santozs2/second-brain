---
title: "Camada de IA — plano de implementação (Desenho B modificado)"
aliases: ["Plano da camada de IA", "Stack de planejamento IA", "Passos 7 a 11 do TCC", "B modificado"]
tags: [tcc, ia, llm, plano, arquitetura, django, recomendacao]
status: em-andamento
projeto: TCC
criado: 2026-08-20
---

> [!info] Projeto: [[TCC|🎓 TCC]] · Decisão: [[decisao-camada-ia|🤖 Decisão da camada de IA]] · Prompt: [[prompt-padrao-recomendacao|📝 Prompt padrão]] · Base: [[engine-matching-cosseno|🧮 Engine de cosseno]]

# 🧩 Camada de IA — plano de implementação

> [!abstract] O que este documento é
> A **stack de planejamento** da camada de IA, já com a decisão do grupo fechada: **Desenho B modificado**. Aqui estão os contratos entre as peças, os arquivos que vão nascer, os cinco passos (7 a 11) e o *Definition of Done* de cada um. O "o quê" e o "por quê" ficaram em [[decisao-camada-ia|🤖 Decisão da camada de IA]]; aqui é o **como**.

## ✅ O que o grupo decidiu

| Ponto | Decisão |
|---|---|
| Desenho | **B modificado** — engine seleciona, LLM **entrega** |
| Poder da LLM | Pode **reordenar dentro do top-5** da engine. Não pode escolher fora dele, nem descartar cursos |
| Entrega ao usuário | **1 curso principal + 4 alternativas** |
| Provedor padrão | **Gemini Flash (free tier)** — plugável por `.env` |
| Prompt | **Um prompt padrão versionado em arquivo**, igual para todos os perfis |

> [!important] O que muda em relação ao Desenho B original
> No B original a LLM "reordenava e justificava" como um extra em cima da tela existente. No **B modificado, a LLM é a voz da entrega**: a tela de resultado passa a ser o texto dela. Isso eleva duas exigências que antes eram opcionais:
> 1. **A engine grava antes.** O ranking determinístico vai para o banco antes de qualquer chamada de rede. A LLM nunca é o único registro do resultado.
> 2. **Divergência é dado, não ruído.** Toda vez que a LLM promove outro curso ao 1º lugar, isso é medido e guardado — é exatamente o material do capítulo comparativo (Passo 11).

---

## 🔀 O fluxo, ponta a ponta

```
POST /enviar/  (6 respostas)
   │
   ▼
engine.recommend(attempt, limit=5)      ← determinístico, já existe e já é testado
   │  grava Recommendation × 5 com rank_engine 1..5
   ▼
[ RESULTADO JÁ EXISTE NO BANCO ]  ────────────────► o fallback vive aqui
   │
   ▼
delivery.build_payload(attempt)         ← monta o JSON de entrada do prompt
   │
   ▼
provider.complete(prompt_padrao_v1)     ← Gemini Flash · temperature=0 · timeout 8s
   │                                       cache por hash das respostas
   ▼
validate_delivery(saida, ids_permitidos) ← serializer DRF: rejeita id fora do top-5,
   │                                        duplicata, faltante ou texto vazio
   ├── ✅ válido  → grava rank_final, llm_* e diverged
   └── ❌ inválido/timeout/sem crédito → used_fallback=True, rank_final = rank_engine
   │
   ▼
/resultado/<pk>/   →  bloco do curso principal + 4 cards de alternativa
```

> [!success] A propriedade que não pode ser perdida
> **Desligue a internet em qualquer ponto depois da engine e o TCC continua respondendo.** Foi assim que o grupo negociou a camada de IA como incremento e não como dependência — o plano inteiro abaixo existe para preservar isso.

---

## 🧱 Stack (nenhuma ferramenta nova além do SDK do provedor)

| Peça | Arquivo | Papel |
|---|---|---|
| Protocolo do provedor | `quiz/llm/base.py` | `LLMProvider` (Protocol) + exceções `LLMTimeout`, `LLMUnavailable` |
| Provedor real | `quiz/llm/gemini.py` | Implementa o protocolo com o SDK do Gemini |
| Provedor de teste | `quiz/llm/fake.py` | `FakeProvider` — resposta fixa, offline, usado nos testes |
| Orquestração | `quiz/delivery.py` | Monta payload → chama provider → valida → grava → decide fallback |
| Prompt padrão | `quiz/prompts/entrega_v1.md` | O prompt **em arquivo**, versionado — vira anexo da monografia |
| Validação da saída | `quiz/serializers.py` (`DeliverySerializer`) | Só aceita `course_id` que foi enviado no prompt |
| Cache | `django.core.cache` | Chave = hash das respostas + versão do prompt + modelo |
| Configuração | `.env` | `LLM_PROVIDER`, `LLM_MODEL`, `LLM_API_KEY`, `LLM_TIMEOUT`, `LLM_ENABLED` |

> [!note] Por que `quiz/llm/` como pacote e não um `llm.py` solto
> Três provedores no mesmo arquivo viram 200 linhas com dois SDKs importados no topo — e aí o `FakeProvider` deixa de rodar sem o pacote do Gemini instalado. Pacote separado mantém a promessa de **desenvolver a camada inteira offline**.

### Variáveis de ambiente

```env
LLM_ENABLED=true
LLM_PROVIDER=gemini          # gemini | fake
LLM_MODEL=gemini-2.0-flash   # trocar de modelo não deve exigir mexer em código
LLM_API_KEY=...              # só no backend, nunca no template ou no JS
LLM_TIMEOUT=8                # segundos; estourou, cai na engine
LLM_PROMPT_VERSION=v1
```

> [!warning] `LLM_ENABLED=false` é o botão de pânico da defesa
> Uma linha no `.env` desliga a camada inteira e o sistema volta a ser o de antes, com a tela da engine. Se a internet da instituição falhar cinco minutos antes da apresentação, **essa é a mitigação** — não improvisar código na hora.

---

## 📐 Os dois contratos

Tudo depende destes dois JSONs. Fechá-los primeiro é o que permite escrever o prompt, o validador e os testes **em paralelo**, sem ninguém esperar ninguém.

### Entrada — o que vai para o prompt

```json
{
  "perfil": {
    "areas_fortes": [
      {"area": "eletrica", "area_name": "Elétrica", "pontuacao": 9},
      {"area": "eletromecanica", "area_name": "Eletromecânica", "pontuacao": 8}
    ]
  },
  "candidatos": [
    {
      "course_id": 7,
      "nome": "Eletricista Instalador",
      "descricao": "…",
      "score": 0.995,
      "rank_engine": 1,
      "top_areas": [
        {"area_name": "Elétrica", "percentual": 52.3},
        {"area_name": "Eletromecânica", "percentual": 37.2}
      ]
    }
  ]
}
```

> [!tip] O prompt recebe `score` e `rank_engine` de propósito
> Sem eles a LLM reordena no escuro e a divergência vira aleatoriedade. Com eles, ela sabe **de quanto** é a diferença que está contrariando — 0,995 × 0,985 é empate técnico, 0,99 × 0,42 não é. Isso transforma "a IA discordou" numa decisão que dá para defender na banca.

### Saída — o que a LLM devolve

```json
{
  "principal": {"course_id": 7, "texto": "…"},
  "alternativas": [
    {"course_id": 3, "texto": "…"},
    {"course_id": 11, "texto": "…"},
    {"course_id": 2, "texto": "…"},
    {"course_id": 9, "texto": "…"}
  ]
}
```

**Regras de validação no servidor** (o que o `DeliverySerializer` recusa):

| Regra | Se violar |
|---|---|
| Todo `course_id` veio em `candidatos` | ❌ fallback — é tentativa de alucinar curso |
| Exatamente 5 ids, sem repetição | ❌ fallback |
| Nenhum candidato sumiu | ❌ fallback — a LLM não pode descartar |
| `texto` não vazio e dentro do limite de caracteres | ❌ fallback |

> [!important] A LLM não declara se reordenou — o servidor calcula
> Seria mais fácil pedir um campo `"reordenei": true` no JSON. Mas aí a métrica do capítulo comparativo passaria a depender do **auto-relato do modelo**, que é justamente a fonte que o trabalho está avaliando. O servidor compara `rank_final` com `rank_engine` e grava `diverged` sozinho. Métrica medida, não declarada.

### Campos novos em `Recommendation`

| Campo | Tipo | Para quê |
|---|---|---|
| `rank_engine` | `PositiveSmallIntegerField` | Ordem determinística (o `rank` atual passa a ser este) |
| `rank_final` | `PositiveSmallIntegerField` | Ordem efetivamente entregue ao usuário |
| `llm_text` | `TextField(blank=True)` | O texto que a pessoa leu |
| `is_primary` | `BooleanField(default=False)` | Qual foi o curso principal |

E em `QuizAttempt` (metadados do experimento, um por tentativa e não por curso):

`llm_model`, `prompt_version`, `latency_ms`, `tokens_in`, `tokens_out`, `used_fallback`, `diverged`, `cache_hit`.

> [!note] Por que os metadados ficam no `QuizAttempt`
> São propriedades **da chamada**, não de cada curso. Repetir `latency_ms` em cinco linhas seria desnormalizar sem motivo — e o Passo 11 consulta exatamente por tentativa: "em quantas das N tentativas houve divergência".

---

## 🗓️ Os cinco passos

> [!warning] Renumeração — atenção ao ler notas antigas
> A nota de decisão original numerava a camada de IA como "Passos 6 a 10", mas o **Passo 6 já existe e é o front em templates** ([[guia-tcc-quiz-perfil|🏗️ Guia de implementação]]). A camada de IA é **Passo 7 a 11**. A numeração antiga foi corrigida em [[decisao-camada-ia|🤖 Decisão]].

| Passo | Entrega | Depende de crédito/internet? |
|---|---|:---:|
| **7** | Protocolo + `FakeProvider` + contratos + testes | ❌ |
| **8** | Prompt padrão v1 + validador da saída | ❌ |
| **9** | `GeminiProvider` + cache + timeout + fallback + persistência | ✅ |
| **10** | Tela de entrega (principal + 4 alternativas) | ❌ |
| **11** | Experimento comparativo + capítulo da monografia | ✅ |

---

### 🔌 Passo 7 — Protocolo, `FakeProvider` e contratos

**Objetivo:** ter a camada inteira de pé, testada, **sem uma linha de rede**.

- [ ] `quiz/llm/base.py` — `LLMProvider` com um método só: `complete(prompt: str) -> str`
- [ ] `quiz/llm/fake.py` — devolve um JSON fixo e válido, montado a partir dos ids que recebeu
- [ ] `quiz/delivery.py` — `build_payload(attempt)` (o JSON de entrada acima)
- [ ] Migration dos campos novos de `Recommendation` e `QuizAttempt`
- [ ] `get_provider()` lendo `LLM_PROVIDER` do `.env`
- [ ] Testes: payload correto, `FakeProvider` respeita o contrato, `LLM_ENABLED=false` não chama nada

> [!tip] Comece pelo `FakeProvider`, não pelo Gemini
> É contraintuitivo escrever o dublê antes do ator, mas é o que blinda o cronograma: se o grupo travar na credencial da API, **os passos 7, 8 e 10 já estarão prontos e testados**. Quando a chave chegar, é plugar.

> [!success] Definition of Done
> `manage.py test` passa com os 12 testes antigos **mais** os novos, com a máquina em modo avião.

---

### 📝 Passo 8 — Prompt padrão v1 e validação da saída

**Objetivo:** congelar o prompt que o grupo vai usar para todos os perfis, e garantir que nada que ele devolva entre no banco sem passar por revista.

- [ ] `quiz/prompts/entrega_v1.md` — ver [[prompt-padrao-recomendacao|📝 Prompt padrão]]
- [ ] Carregador que lê o arquivo e injeta os placeholders (sem f-string espalhada pelo código)
- [ ] `DeliverySerializer` com as 4 regras de validação da tabela acima
- [ ] Testes de saída maliciosa: id inexistente, id repetido, só 3 cursos, JSON quebrado, texto vazio
- [ ] Cada um desses casos precisa terminar em `used_fallback=True`, **nunca** em erro 500

> [!important] O prompt é arquivo, não string
> Prompt em `.md` versionado no Git dá três coisas de graça: entra como **anexo da monografia**, o `git diff` mostra a evolução v1 → v2 (material para o capítulo de metodologia), e trocar o texto não exige tocar em código Python.

> [!success] Definition of Done
> Um provider que devolve lixo proposital não derruba a aplicação nem grava nada inválido — a tela mostra o resultado da engine.

---

### 🌐 Passo 9 — `GeminiProvider`, cache, timeout e fallback

**Objetivo:** a primeira chamada real, com todas as proteções já no lugar.

- [ ] Conta no Google AI Studio e geração da chave — **conferir os limites atuais do free tier no painel**, eles mudam
- [ ] `quiz/llm/gemini.py` implementando o protocolo, com `temperature=0`
- [ ] `timeout` de 8s; estourou → `LLMTimeout` → fallback silencioso
- [ ] Cache: chave = `sha256(prompt_version + modelo + ids_das_alternativas_ordenados)`
- [ ] Gravar `latency_ms`, tokens, `llm_model`, `prompt_version`, `used_fallback`, `diverged`, `cache_hit`
- [ ] Testes com o provider **mockado** — a suíte automatizada continua sem tocar a rede

> [!warning] Reprodutibilidade não é só `temperature=0`
> `temperature=0` reduz a variação, mas não a elimina, e o provedor pode atualizar o modelo por baixo. O que garante a banca é o **cache**: mesma tentativa → mesma resposta, servida do cache, sem nova chamada. Antes da apresentação, rodar os perfis de demonstração uma vez para populá-lo.

> [!success] Definition of Done
> Uma tentativa real volta com texto do Gemini, `latency_ms` gravado, e a segunda execução do mesmo perfil sai do cache com `cache_hit=True`.

---

### 🎨 Passo 10 — A tela de entrega

**Objetivo:** o formato decidido — 1 principal + 4 alternativas — no ar.

- [ ] `result.html`: bloco destacado do curso principal com o texto da LLM
- [ ] 4 cards de alternativa, cada um com sua frase curta
- [ ] Score e `explanation` da engine continuam visíveis (em "por que este curso", recolhível)
- [ ] Selo discreto de fallback quando `used_fallback=True`
- [ ] Carregamento em duas etapas: resultado da engine aparece na hora, texto da LLM chega depois

> [!note] Não esconder a engine atrás da IA
> Tentação natural: a tela nova fica bonita e o `explanation` some. Não pode — **a engine é o núcleo defensável do trabalho**, o texto é a embalagem. A banca vai querer ver o número por trás da frase.

> [!success] Definition of Done
> Com `LLM_ENABLED=false`, a tela continua respondendo 200 e mostrando os 5 cursos. Com `true`, mostra a entrega da LLM.

---

### 📊 Passo 11 — Experimento comparativo e capítulo

**Objetivo:** transformar a camada de IA em resultado mensurado, não em "usamos IA".

- [ ] `manage.py run_experiment --n=30` — roda N perfis pelos dois caminhos e exporta CSV
- [ ] Métricas: **taxa de divergência** (% em que a LLM trocou o 1º lugar), latência média, taxa de fallback, tokens gastos
- [ ] Avaliação humana: o grupo classifica cada divergência em *melhorou · indiferente · piorou*
- [ ] Escrever o capítulo com esses números

> [!question] A pergunta que a banca vai fazer
> *"Por que IA, se a engine já recomendava?"* — a resposta só existe se este passo for feito. Sem números, a camada de IA é enfeite; com a taxa de divergência e a avaliação humana, ela vira **um experimento com resultado**, que é o que diferencia um TCC de um projeto de portfólio.

> [!success] Definition of Done
> CSV gerado, tabela de métricas fechada e o capítulo escrito em [[defesa-monografia-tcc|🎤 Defesa e monografia]].

---

## ⚠️ Riscos específicos do B modificado

| Risco | Por que aparece agora | Mitigação |
|---|---|---|
| LLM promove um curso claramente pior | Ela ganhou poder de reordenar | `score` no prompt + regra de só reordenar em diferença pequena + avaliação humana no Passo 11 |
| Texto bonito escondendo ranking ruim | A entrega passou a ser a voz da LLM | `explanation` da engine permanece na tela |
| Free tier esgotado durante os testes | O Passo 11 faz dezenas de chamadas | Cache + `FakeProvider` no desenvolvimento + provedor plugável |
| Divergência virar anedota, não dado | Sem instrumentação, ninguém mede | `diverged` calculado pelo servidor desde o Passo 9 |
| Texto genérico ("ótima área, muito futuro") | O prompt não proibiu | Regras de estilo no prompt + revisão manual de amostra |

## ▶️ Próxima ação

**Passo 7, e só ele.** Não abrir conta em provedor antes de o `FakeProvider` estar verde — a credencial não desbloqueia nada que o Passo 7 não tenha entregado antes.

## 📎 Veja também

- [[decisao-camada-ia|🤖 Decisão da camada de IA]] — desenhos, orçamento e a decisão registrada
- [[prompt-padrao-recomendacao|📝 Prompt padrão de entrega]]
- [[engine-matching-cosseno|🧮 Engine de matching]] · [[modelagem-dados-quiz|🗃️ Modelagem]] · [[api-quiz-drf|🔌 API]]
- [[front-templates-django|🎨 Front em templates]] · [[testes-e-validacao-tcc|✅ Testes]]
- [[defesa-monografia-tcc|🎤 Defesa e monografia]]
- [[TCC|🎓 TCC]] · [[Projetos|🚀 Projetos]] · [[Home|Painel Principal]]
