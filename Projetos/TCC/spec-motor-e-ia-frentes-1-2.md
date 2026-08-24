---
title: "Spec de atualização — Frentes 1 e 2 (motor e camada de IA)"
aliases: ["Spec F1 e F2", "Spec do motor e da IA", "Plano de execução F1+F2", "Spec scrum master"]
tags: [tcc, spec, planejamento, gestao, engine, ia, llm, scrum]
status: em-andamento
projeto: TCC
criado: 2026-08-24
---

> [!info] Projeto: [[TCC|🎓 TCC]] · Divisão: [[divisao-de-trabalho-tcc|👥 4 frentes]] · Plano da IA: [[camada-ia-plano-implementacao|🧩 Camada de IA]] · Prompt: [[prompt-padrao-recomendacao|📝 Prompt v1]] · Motor: [[engine-matching-cosseno|🧮 Engine]]
> **Execução:** [[passo-a-passo-f1-01-f1-02|🔧 Passo a passo de F1-01 e F1-02]]

# 🧭 Spec — Frentes 1 e 2 sob o mesmo dono

> [!abstract] O que este documento é
> A **especificação executável** das duas primeiras frentes da [[divisao-de-trabalho-tcc|divisão de trabalho]], com dono definido: **você, que também é scrum master**. Não repete o "porquê" (isso está nas notas de decisão) — aqui está o **delta entre o código que existe hoje e o que precisa existir**, tarefa a tarefa, com contrato, arquivo, teste e critério de pronto.
> Auditoria do repositório feita em **2026-08-24**, sobre o commit `a628753`.

## 0️⃣ O que muda na divisão

| Frente                                  | Dono anterior | Dono agora                                                              |
| --------------------------------------- | ------------- | ----------------------------------------------------------------------- |
| 1️⃣ 🧮 Motor e calibração               | você          | **você**                                                                |
| 2️⃣ 🤖 Camada de IA e experimento       | *a definir*   | **você**                                                                |
| 3️⃣ 📚 Catálogo, trilha e integração    | *a definir*   | *a definir* — **é o caminho crítico, tem que sair na primeira reunião** |
| 4️⃣ 🎨 Jornada, avaliação e indicadores | *a definir*   | *a definir*                                                             |

> [!success] F1 + F2 na mesma pessoa é a única fusão que não custa caro
> As duas frentes estão **coladas no mesmo contrato**: o motor publica `build_payload`, a camada de IA consome. Era exatamente ali que ia morar a maior parte da conversa entre duas pessoas. Juntando, a fronteira vira código em vez de reunião — e o guardião do contrato passa a ser o mesmo dos dois lados.

> [!warning] E o custo que ela cobra
> Você acumula **as duas frentes que dependem uma da outra, mais o chapéu de scrum master**. Se você travar, dois quintos do TCC travam junto. Duas mitigações que valem como regra:
> 1. **Branches separadas mesmo assim** (`feat/motor`, `feat/camada-ia`), com o PR de cada uma revisado por outra pessoa do grupo. Dono único não pode virar merge sem revisor.
> 2. **F1 antes de F2, sempre.** Nunca abrir tarefa de IA com contrato do motor em aberto — é o que impede as duas frentes de se atropelarem dentro da sua própria cabeça.

---

## 1️⃣ Estado real do código hoje

> [!note] Verificado no repositório, não nas notas
> As notas descrevem o alvo; a tabela descreve o que o `git` tem agora.

| Peça | Arquivo | Estado |
|---|---|---|
| Matemática do cosseno | `quiz/engine.py` | ✅ completa (`course_vector`, `profile_vector`, `dot`, `norm`, `cosine_similarity`, `explain`, `rank_courses`) |
| Persistência do ranking | `quiz/engine.py::recommend` | ✅ atômica, idempotente, 3 queries |
| Testes | `quiz/tests.py` | ✅ 12 verdes (4 de matemática em `SimpleTestCase`, 8 de recomendação) |
| Modelo de recomendação | `quiz/models.py::Recommendation` | ⚠️ tem `score`, `rank`, `explanation` — **falta tudo da camada de IA** |
| Metadados do experimento | `quiz/models.py::QuizAttempt` | ❌ só `created_at` e `respondent_name` |
| `build_payload` | — | ❌ não existe |
| Indicador de confiança | — | ❌ não existe |
| Pacote `quiz/llm/` | — | ❌ não existe |
| `quiz/delivery.py` | — | ❌ não existe |
| `quiz/prompts/entrega_v1.md` | — | ❌ não existe (o texto só existe nesta vault, em [[prompt-padrao-recomendacao\|📝 Prompt v1]]) |
| Configuração por `.env` | `config/settings.py` | ❌ nenhuma variável `LLM_*`; o `.env` já está no `.gitignore` |
| Catálogo | seeds | ⚠️ **12 cursos fictícios / 7 áreas / 6 perguntas** — os 18 reais são da F3 |

> [!important] O `limit=5` está escrito no código e em nenhum contrato
> `recommend(attempt, limit=5)` traz o 5 no default da função; `web_views.py` e `views.py` chamam sem passar nada. Se o grupo decidir top-3, hoje isso vira caça ao número mágico. **É a primeira tarefa da spec** (F1-01).

---

## 2️⃣ Os três contratos que esta spec congela

> [!important] Contrato antes de código — vale para você também
> Nenhuma tarefa de implementação abaixo começa antes destes três blocos estarem fechados e anunciados ao grupo. É a mesma regra que a [[divisao-de-trabalho-tcc|divisão]] impõe às outras frentes.

### 📄 C1 — `build_payload(attempt)` · F1 publica, F2 consome

Base em [[camada-ia-plano-implementacao|🧩 plano]], **com uma adição**: o bloco `confianca`, que não existia lá e nasce da tarefa F1-04.

```json
{
  "perfil": {
    "areas_fortes": [
      {"area": "eletrica", "area_name": "Elétrica", "pontuacao": 9}
    ],
    "respostas_dadas": 6
  },
  "confianca": {
    "gap_top1_top2": 0.0002,
    "nivel": "empate_tecnico",
    "score_top1": 0.9877
  },
  "candidatos": [
    {
      "course_id": 7,
      "nome": "Eletricista Instalador",
      "descricao": "…",
      "score": 0.9875,
      "rank_engine": 1,
      "top_areas": [{"area_name": "Elétrica", "percentual": 52.3}]
    }
  ]
}
```

`nivel` ∈ `alta` · `empate_tecnico` · `perfil_fraco`. Regras em F1-04.

### 📄 C2 — saída da LLM · F2 valida

Idêntico ao de [[camada-ia-plano-implementacao|🧩 plano]] (`principal` + 4 `alternativas`). **Sem alteração** — está fechado e o prompt v1 já foi escrito contra ele.

### 📄 C3 — entrega ao front · F2 publica, F4 consome

O contrato que **ainda não existia em lugar nenhum** e que a F4 precisa para desenhar a tela do Passo 10.

```
GET /api/quiz/attempts/<pk>/entrega/
```

```json
{
  "attempt_id": 42,
  "pronto": true,
  "used_fallback": false,
  "diverged": true,
  "confianca": {"nivel": "empate_tecnico"},
  "principal": {
    "course_id": 7, "course_name": "…", "descricao": "…", "duration_hours": 160,
    "match": 99, "texto_llm": "…", "explanation": {"top_areas": []}
  },
  "alternativas": [
    {"course_id": 3, "course_name": "…", "match": 98, "texto_llm": "…", "explanation": {}}
  ]
}
```

> [!tip] `pronto: false` é o que permite o carregamento em duas etapas
> O Passo 10 pede que o resultado da engine apareça na hora e o texto da LLM chegue depois. Com este campo, a F4 desenha **uma** tela e não precisa saber se a chamada foi síncrona: enquanto `pronto` for `false`, mostra o resultado do motor com o `explanation`; quando virar `true`, troca o texto. Sem isso, a F4 fica bloqueada esperando a sua camada de IA — e o paralelismo morre.

---

## 3️⃣ Migração de dados — a parte que quebra os outros

> [!warning] É a única mudança desta spec que atinge código de outra frente
> Renomear `Recommendation.rank` toca `quiz/serializers.py`, `quiz/web_views.py` e `quiz/admin.py`, território da **F4**. Anuncie **antes**, faça num único PR e não fatie: meia renomeação é pior que nenhuma.

**Campos novos em `Recommendation`:** `rank` → `rank_engine`; e nascem `rank_final`, `llm_text` (`TextField(blank=True)`), `is_primary` (`BooleanField(default=False)`).

**Campos novos em `QuizAttempt`:** `llm_model`, `prompt_version`, `latency_ms`, `tokens_in`, `tokens_out`, `used_fallback`, `diverged`, `cache_hit` — mais `confidence_gap` e `confidence_level`, que são do motor e não da IA.

Sequência da migração (o banco de desenvolvimento tem tentativas reais — não dá para `default=0` e seguir):

1. Soltar o `ordering` do `Meta` (senão o state histórico aponta para um campo que deixou de existir)
2. `RenameField('rank' → 'rank_engine')`
3. `AddField('rank_final', null=True)` e os demais campos
4. **Data migration**: `rank_final = rank_engine` e `is_primary = (rank_engine == 1)` em toda linha existente
5. `AlterField('rank_final', null=False)`
6. `ordering = ["rank_final"]` — a ordem que o usuário viu, não a que o motor calculou

> [!note] Por que `ordering` passa a ser `rank_final`
> Se ficar em `rank_engine`, toda tela e todo serializer devolvem a ordem do motor mesmo quando a LLM reordenou — e o bug aparece só no primeiro dia em que houver divergência, que tende a ser o dia da defesa. Ordenar pelo que foi entregue é o padrão seguro; quem quiser a ordem do motor pede explicitamente.

> [!tip] O código exato das duas primeiras tarefas está separado
> Migration escrita à mão, arquivo por arquivo, com os comandos de verificação: [[passo-a-passo-f1-01-f1-02|🔧 Passo a passo de F1-01 e F1-02]]. Esta spec diz **o que** e **por quê**; aquela nota é **como**, para executar com o editor aberto.

---

## 4️⃣ Frente 1 — backlog do motor

| # | Tarefa | Arquivos | Depende de | Pronto quando |
|---|---|---|---|---|
| **F1-01** | `RECOMMENDATION_LIMIT` no settings; `recommend(attempt, limit=None)` lê de lá | `config/settings.py`, `quiz/engine.py` | decisão top-5/top-3 | Nenhum `5` literal fora do settings; teste cobre `limit` alternativo |
| **F1-02** | Migração dos campos novos (seção 3) | `quiz/models.py`, migration, `serializers.py`, `web_views.py`, `admin.py` | anúncio à F4 | `manage.py test` verde; tentativa antiga abre no site sem erro |
| **F1-03** | `build_payload(attempt)` → C1 | `quiz/delivery.py` | F1-02 | Teste compara o dict com o JSON de C1, campo a campo |
| **F1-04** | Indicador de confiança | `quiz/engine.py`, `quiz/models.py` | F1-02 | 3 testes: alta, empate técnico, perfil fraco |
| **F1-05** | Análise de sensibilidade | `quiz/management/commands/sensibilidade.py` | catálogo (qualquer) | CSV com Δposição por Δpeso; tabela pronta para o artigo |
| **F1-06** | Recalibração com os 18 cursos reais | seeds (F3) + revisão dos 4 critérios de aceite | **F3** | Os 4 perfis de aceite passam com o catálogo real |
| **F1-07** | Bateria nova de testes | `quiz/tests.py` | F1-01 a F1-04 | Contagem sobe de 12 para ~22 |
| **F1-08** | Seção de metodologia do cálculo | [[artigo-secao-calculo-cosseno\|✍️ artigo]] | F1-05, F1-06 | Texto com os números do catálogo real |

### 🎚️ F1-04 em detalhe — o indicador de confiança

Uma constante, um lugar só:

```python
LIMIAR_EMPATE = 0.05   # quiz/engine.py — MESMO número da regra 2 do prompt v1
```

| Situação | Nível | Consequência |
|---|---|---|
| `score_top1 - score_top2 < LIMIAR_EMPATE` | `empate_tecnico` | A LLM **pode** reordenar (regra 2 do prompt) |
| `score_top1 < 0.30` | `perfil_fraco` | O texto não pode fingir certeza — é o caso de teste que a nota do prompt aponta como "o mais fácil de esquecer" |
| resto | `alta` | A ordem do motor é para ser respeitada |

> [!important] O limiar 0,05 vai existir em dois lugares — e um deles é um arquivo `.md`
> A regra 2 do [[prompt-padrao-recomendacao|📝 prompt v1]] tem o número escrito no texto do prompt; o motor vai ter o mesmo número em Python. **A duplicação é real e é aceitável** — o modelo precisa do número em linguagem natural. O que não é aceitável é mudar um e esquecer o outro: se o valor mudar, o prompt vira `entrega_v2.md` e a constante muda no mesmo commit. Deixe um comentário no código apontando para o arquivo do prompt.

> [!question] O que a banca pergunta em F1-05
> *"E se vocês tivessem dado peso 4 em vez de 5 nesse curso?"* — a análise de sensibilidade é a resposta pronta. Sem ela, a resposta honesta é "não sei", e aí a atribuição de pesos (que já é a limitação mais visível do trabalho) vira a limitação **não medida**. Rode-a mesmo com o catálogo fictício na semana 2: o método fica validado e é só reexecutar quando os 18 cursos chegarem.

---

## 5️⃣ Frente 2 — backlog da camada de IA

Decomposição dos Passos 7 a 11 do [[camada-ia-plano-implementacao|🧩 plano]] em tarefas de um dia.

| # | Tarefa | Passo | Rede? | Pronto quando |
|---|---|:---:|:---:|---|
| **F2-01** | `quiz/llm/base.py` — `LLMProvider` (Protocol, só `complete`) + `LLMTimeout`, `LLMUnavailable` | 7 | ❌ | Import limpo sem nenhum SDK instalado |
| **F2-02** | `quiz/llm/fake.py` — `FakeProvider` monta JSON válido a partir dos ids recebidos | 7 | ❌ | Devolve C2 válido para qualquer payload |
| **F2-03** | `get_provider()` + variáveis `LLM_*` no settings, com `python-dotenv` e `.env.example` | 7 | ❌ | `LLM_ENABLED=false` não instancia provider nenhum |
| **F2-04** | `quiz/prompts/entrega_v1.md` — copiar **literal** de [[prompt-padrao-recomendacao\|📝 prompt v1]] | 8 | ❌ | O arquivo e a nota têm o mesmo texto, caractere a caractere |
| **F2-05** | Carregador do prompt: lê o arquivo e injeta `{{PERFIL_JSON}}` e `{{CANDIDATOS_JSON}}` | 8 | ❌ | Zero f-string de prompt em view ou serializer |
| **F2-06** | `DeliverySerializer` — as 4 regras de validação | 8 | ❌ | 6 testes de saída maliciosa, todos terminando em fallback e **nenhum** em 500 |
| **F2-07** | `delivery.deliver(attempt)` — orquestra, valida, grava, calcula `diverged`, decide fallback | 8 | ❌ | Com `FakeProvider`, a tentativa grava `rank_final`, `llm_text` e `is_primary` |
| **F2-08** | `quiz/llm/gemini.py` + cache + timeout de 8s + metadados | 9 | ✅ | 2ª execução do mesmo perfil sai do cache com `cache_hit=True` |
| **F2-09** | Endpoint de entrega — contrato **C3** | 9/10 | ❌ | A F4 monta a tela sem precisar te perguntar nada |
| **F2-10** | `manage.py run_experiment --n=30` → CSV | 11 | ✅ | CSV com divergência, latência, fallback e tokens por tentativa |
| **F2-11** | Capítulo de engenharia de prompt e resultados | 11 | — | Escrito em [[defesa-monografia-tcc\|🎤 defesa]] |

> [!tip] Oito das onze tarefas rodam em modo avião
> F2-01 a F2-07, mais F2-09, não tocam a rede. Isso é de propósito: a pendência da credencial (pergunta 5 de [[decisao-camada-ia|🤖 Decisão]], **ainda em aberto**) só bloqueia F2-08 e F2-10. Se a chave demorar duas semanas, a frente 2 continua andando — desde que você não tenha começado por ela.

> [!warning] `LLM_ENABLED=false` precisa de teste, não de boa-fé
> Ele é o botão de pânico da defesa. Um botão de pânico que ninguém apertou desde que foi instalado não é mitigação, é esperança. Teste explícito em F2-03: com a flag desligada, `deliver()` não instancia provider, não chama rede, e a tela responde 200 com o resultado do motor.

---

## 6️⃣ Base conceitual — o que sustenta cada tarefa

> [!success] Novidade de 2026-08-24: a fundamentação parou de ser dívida
> O vault ganhou quatro pastas de conceito (`Recomendação`, `Inteligência Artificial`, `Metodologia Científica`, `Testes`). Isso muda o peso de F1-08 e F2-11: **escrever os capítulos deixou de ser pesquisar do zero e virou amarrar código a nota que já existe**. Cada linha abaixo é uma citação que a monografia pode fazer sem inventar referência.

| Tarefa | Leitura de apoio | Para quê |
|---|---|---|
| F1-01, F1-06 | [[rec-sistemas-de-recomendacao\|Sistemas de recomendação]] · [[rec-filtragem-conteudo\|Filtragem por conteúdo]] | Situar o EducMatch na taxonomia — é filtragem por conteúdo, não colaborativa |
| F1-04, F1-05 | [[rec-metricas-similaridade\|Métricas de similaridade]] · [[rec-normalizacao-vetorial\|Normalização vetorial]] · [[rec-similaridade-cosseno\|Similaridade de cosseno]] | Defender **por que cosseno** e não outra métrica; o argumento do "curso gordo" tem nome próprio |
| F1-03, F1-08 | [[rec-modelo-espaco-vetorial\|Modelo de espaço vetorial]] · [[rec-tf-idf\|TF-IDF]] | O VSM é a moldura teórica do `profile_vector` e do `course_vector` |
| Explicação na tela | [[rec-explicabilidade\|Recomendação explicável]] | O `explanation` é o diferencial do trabalho — aqui está o vocabulário para defendê-lo |
| F1-06 | [[rec-cold-start\|Cold start]] · [[rec-vieses-e-etica\|Vieses e ética]] | Por que o quiz existe (não há histórico) e o que a atribuição manual de pesos introduz de viés |
| F2-04, F2-05 | [[ia-engenharia-de-prompt\|Engenharia de prompt]] | Prompt como especificação de interface — exatamente o argumento do prompt v1 em arquivo |
| F2-06, F2-07 | [[ia-alucinacao-e-grounding\|Alucinação e grounding]] | A regra 1 do prompt + a revalidação no servidor são **contenção por arquitetura**, e é assim que se escreve isso |
| F2-08 | [[ia-tokens-e-custo\|Tokens, custo e latência]] · [[ia-llm-fundamentos\|Fundamentos de LLM]] | Sustenta o orçamento de R$ 0 e a escolha do timeout de 8s |
| F2-10, F2-11 | [[ia-avaliacao-de-llm\|Avaliação de LLM]] · [[rec-metricas-avaliacao\|Métricas de avaliação]] · [[rec-sistemas-hibridos\|Sistemas híbridos]] | O experimento comparativo é avaliação de saída não-determinística; as duas notas dizem como se faz isso sem virar opinião |
| F1-07, F2-06 | [[tst-mocks-e-dubles\|Mocks e dublês]] · [[tst-testes-django\|Testes em Django]] · [[tst-piramide-de-testes\|Pirâmide de testes]] | O `FakeProvider` tem nome técnico e taxonomia; `SimpleTestCase` × `TestCase` também |
| F1-08, F2-11 | [[met-estrutura-monografia\|Estrutura da monografia]] · [[met-validade-e-limitacoes\|Validade e limitações]] · [[met-tipos-de-pesquisa\|Tipos de pesquisa]] · [[met-normas-abnt\|ABNT]] | Onde cada capítulo entra e como declarar as limitações antes que a banca as encontre |

> [!important] A nota que muda o capítulo de limitações
> [[met-validade-e-limitacoes|Validade e limitações]] traz a taxonomia dos quatro tipos de validade. As três limitações que o TCC já assume — perfis sintéticos, pesos atribuídos por julgamento, piloto simulado — deixam de ser um parágrafo de desculpas e viram **ameaças à validade classificadas**, que é a forma que a banca reconhece. Vale ler antes de escrever qualquer capítulo, não depois.

---

## 7️⃣ Testes — de 12 para ~22

| Nova cobertura | Frente | Tipo |
|---|---|---|
| `limit` configurável respeitado | F1 | `TestCase` |
| `build_payload` bate o contrato C1 | F1 | `TestCase` |
| Confiança: alta / empate técnico / perfil fraco | F1 | `SimpleTestCase` (função pura) |
| `rank_final` igual a `rank_engine` quando não há IA | F1 | `TestCase` |
| `FakeProvider` devolve C2 válido | F2 | `SimpleTestCase` |
| Saída maliciosa: id fantasma, id repetido, só 3 cursos, JSON quebrado, texto vazio, texto gigante | F2 | `TestCase` × 6 |
| `LLM_ENABLED=false` não chama provider | F2 | `TestCase` |
| `diverged` calculado pelo servidor, não lido da LLM | F2 | `TestCase` |
| Fallback grava `used_fallback=True` e preserva a ordem do motor | F2 | `TestCase` |

> [!success] A regra que protege as outras frentes de você
> `manage.py test` verde é condição de merge ([[divisao-de-trabalho-tcc|regra 4 de convivência]]). Com F1 e F2 na mesma mão, é o seu teste que avisa quando você quebrou o contrato entre as suas próprias duas frentes — porque não vai ter outra pessoa reclamando no merge.

---

## 8️⃣ Plano — 5 semanas, alinhado ao calendário do grupo

| Semana | Período | F1 (motor) | F2 (IA) | Entregável fechado |
|---|---|---|---|---|
| **1** | 24/08 – 30/08 | F1-01, F1-02, F1-03 | F2-01, F2-02, F2-03 | **Contratos C1/C2/C3 anunciados** + camada offline de pé |
| **2** | 31/08 – 06/09 | F1-04, F1-05 | F2-04, F2-05, F2-06 | Prompt em arquivo + validador que não deixa lixo entrar |
| **3** | 07/09 – 13/09 | F1-06 (depende da F3) | F2-07, F2-08 | Primeira chamada real ao Gemini, com fallback provado |
| **4** | 14/09 – 20/09 | F1-07 | F2-09, F2-10 | Experimento rodado, CSV na mão |
| **5** | 21/09 – 27/09 | F1-08 | F2-11 | Dois capítulos escritos |

> [!important] A semana 3 é a única com dependência externa — e são duas ao mesmo tempo
> F1-06 espera o catálogo real da **F3**; F2-08 espera a **credencial**. Se as duas atrasarem, sua semana 3 esvazia. O contorno se decide agora, não na hora: **puxe F1-07 e F2-09 para a semana 3** (as duas são offline e sem dependência) e empurre a recalibração. O que não pode acontecer é você parar esperando.

### 🎬 Ordem de partida — as cinco primeiras coisas

1. Fechar os nomes das frentes 3 e 4 no kickoff — **a F3 é o caminho crítico do TCC inteiro**
2. Decidir top-5 ou top-3 e registrar ([[escopo-fluxo-educmatch|🗺️ ponto 2]] — a divergência entre o quadro e a decisão continua aberta)
3. `git checkout -b feat/motor` → F1-01, seguindo o [[passo-a-passo-f1-01-f1-02|🔧 passo a passo]]
4. Anunciar a renomeação `rank → rank_engine` ao grupo **antes** de abrir o PR
5. F1-02 e F1-03 no mesmo PR, com os testes junto

---

## 9️⃣ Decisões que precisam sair do kickoff

> [!todo] Sem estas cinco, a semana 1 não começa limpa
> 1. **Top-5 ou top-3** — trava F1-01 e o texto do prompt. *Recomendação: manter top-5; o formato 1+4 já está desenhado e mandar 5 candidatos custa quase o mesmo que 3.*
> 2. **Donos das frentes 3 e 4** — F3 atrasada empurra F1-06 e esvazia a sua semana 3.
> 3. **Quem guarda a credencial** da API — aberta desde [[decisao-camada-ia|🤖 Decisão]]. Só trava F2-08, mas defina agora mesmo assim.
> 4. **N do experimento** (F2-10) e quem faz a avaliação humana das divergências. *Sugestão: N=30 e avaliação por duas pessoas de outras frentes — você é parte interessada no resultado e não deveria avaliar sozinho.*
> 5. **Quem revisa os seus PRs.** Dono único de duas frentes precisa de revisor fixo, senão a regra do pull request vira formalidade.

---

## 🔟 Seus ritos como scrum master

| Rito | Quando | O que sai dele |
|---|---|---|
| Kickoff | antes de 24/08 | As 5 decisões da seção 9 |
| Semanal de 30 min | fixo, mesma hora | "o que fechei / o que trava / o que preciso de quem", por frente |
| Revisão de contrato | sob demanda | Nenhuma mudança de formato entra sem anúncio prévio |
| Fechamento de semana | sexta | O que fechou e o que rolou — rolar duas semanas seguidas é sinal de escopo errado, não de preguiça |

> [!warning] O risco de escala que só você enxerga
> Você tem visão das quatro frentes **e** entrega em duas. Quando o tempo apertar, a tentação é cortar do próprio backlog para não ter que cobrar dos outros — e o que cai primeiro costuma ser F1-05 (sensibilidade) e F2-10 (experimento), que são justamente **as duas tarefas que a banca mais valoriza**. Se algo tiver que cair, que seja tela ou refinamento de texto, nunca a evidência medida.

## 📎 Veja também

- [[passo-a-passo-f1-01-f1-02|🔧 Passo a passo de F1-01 e F1-02]] — o código, arquivo por arquivo
- [[divisao-de-trabalho-tcc|👥 Divisão de trabalho entre as 4 frentes]] · [[escopo-fluxo-educmatch|🗺️ Escopo EducMatch]]
- [[camada-ia-plano-implementacao|🧩 Plano da camada de IA]] · [[prompt-padrao-recomendacao|📝 Prompt padrão v1]] · [[decisao-camada-ia|🤖 Decisão da camada de IA]]
- [[engine-matching-cosseno|🧮 Engine de matching]] · [[modelagem-dados-quiz|🗃️ Modelagem]] · [[testes-e-validacao-tcc|✅ Testes]]
- [[artigo-secao-calculo-cosseno|✍️ Artigo: seção do cálculo]] · [[defesa-monografia-tcc|🎤 Defesa e monografia]] · [[fundamentacao-teorica-recomendacao|📖 Fundamentação teórica]]
- **Conceitos:** [[rec-similaridade-cosseno|Cosseno]] · [[rec-explicabilidade|Explicabilidade]] · [[ia-engenharia-de-prompt|Engenharia de prompt]] · [[ia-avaliacao-de-llm|Avaliação de LLM]] · [[met-validade-e-limitacoes|Validade e limitações]] · [[tst-mocks-e-dubles|Mocks e dublês]]
- [[TCC|🎓 TCC]] · [[Projetos|🚀 Projetos]]
