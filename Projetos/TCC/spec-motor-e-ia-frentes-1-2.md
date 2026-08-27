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
> **Execução da Frente 2:** [[passo-a-passo-f2-07|🎛️ F2-07 orquestração]] → [[passo-a-passo-f2-09|🔌 F2-09 endpoint C3]] → [[passo-a-passo-f2-08|🌐 F2-08 Gemini]] *(F2-08 por último: é a única bloqueada pela credencial)*

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

### 🎭 O papel da IA — desempate e calibração

> [!important] Decidido em conversa de 2026-08-24, **falta ratificar no grupo**
> A regra antiga ("a LLM pode reordenar se o gap for menor que 0,05") **não sobrevive à escala** e já está furada hoje: em 3 dos 4 perfis de aceite documentados em [[engine-matching-cosseno|🧮 Engine]], o gap entre 1º e 2º já é menor que 0,05. A exceção virou o caso comum, com 12 cursos. Com 180, vira 100% dos casos.

No lugar dela, **dois eixos independentes**:

| Eixo | Quem decide | Regra |
|---|---|---|
| **Qual curso é o principal** | engine **ou** LLM | A engine publica o **conjunto de empate**: os candidatos dentro de ε do topo. Conjunto de 1 → a engine decidiu, a LLM só apresenta. Conjunto ≥ 2 → a LLM escolhe entre os empatados |
| **Com quanta certeza o texto fala** | sempre a LLM | A **banda** do score do principal define o tom: alta, média ou baixa |

Nos dados reais do projeto:

| Perfil | Top-1 × Top-2 | Gap | Conjunto | Quem escolhe |
|---|---|---|---|---|
| Automotivo + elétrico | 0,968 × 0,891 | 0,077 | 1 | **engine** |
| Costura | 0,999 × 0,969 | 0,030 | ≥ 2 | LLM |
| TI e dados | 1,000 × 0,979 | 0,021 | ≥ 2 | LLM |
| Elétrico (6 respostas) | 0,9877 × 0,9875 | 0,0002 | ≥ 2 | LLM |

> [!success] Por que o automotivo tem que ficar fora
> `Injeção Eletrônica > Motores a Combustão` é o resultado do `test_perfil_misto_prefere_o_curso_hibrido` — o único critério de aceite em que a ordem correta **emergiu do cálculo**, sem nenhum `if`. Se a LLM puder virar esse caso, ela pode desfazer a evidência sobre a qual o capítulo de metodologia está construído. Gap de 0,077 é diferença real: o sistema dizer "aqui não se mexe" é ele funcionando.

> [!important] A regra de arquitetura que decorre disso
> **Se alguma coisa deveria mudar a ordem, é porque a engine não sabia de algo.** Pré-requisito que a pessoa não tem, curso que não abre turma na unidade dela, módulo que ela já fez — nada disso é reordenação, é **filtragem**, e o lugar dela é *antes* da engine. Toda vez que alguém propuser "e se a IA pudesse considerar X", a resposta certa é transformar X em entrada do cálculo. Duas fontes de ordenação discordando é um sistema que ninguém consegue auditar.

### 📄 C1 — `build_payload(attempt)` · F1 publica, F2 consome

Base em [[camada-ia-plano-implementacao|🧩 plano]], **com duas adições**: o bloco `confianca` (que nasce da tarefa F1-04) e o bloco `respostas`.

```json
{
  "perfil": {
    "areas_fortes": [
      {"area": "eletrica", "area_name": "Elétrica", "pontuacao": 9}
    ],
    "respostas_dadas": 6
  },
  "respostas": [
    {"pergunta": "Onde você prefere trabalhar?", "escolha": "Chão de fábrica"},
    {"pergunta": "O que mais te interessa?", "escolha": "Eletricidade e comandos"}
  ],
  "confianca": {
    "score_top1": 0.9877,
    "gap_top1_top2": 0.0002,
    "banda": "alta",
    "conjunto_empate": [11, 7],
    "quem_escolhe": "llm"
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

| Campo | Valores | Para quê |
|---|---|---|
| `banda` | `alta` · `media` · `baixa` | Calibra o tom do texto (eixo C) |
| `conjunto_empate` | lista de `course_id` dentro de ε do topo | Delimita entre quem a LLM pode escolher (eixo D) |
| `quem_escolhe` | `engine` · `llm` | Derivado: `engine` quando o conjunto tem 1 elemento |

> [!warning] Sem o bloco `respostas`, o eixo D não fecha
> Se a engine empatou dois cursos usando os números do perfil, e a LLM recebe **os mesmos números**, ela não tem com o que desempatar — estaria chutando sobre exatamente o dado que já empatou. O desempate só existe se a LLM tiver algo que o vetor perdeu: *"prefiro oficina"* e *"prefiro chão de fábrica"* podem virar o mesmo peso de mecânica no vetor, mas dizem coisas diferentes sobre a pessoa.

> [!note] Isso não abre superfície de injeção de prompt
> As alternativas são **fechadas e escritas pelo grupo** — o estudante não digita texto livre. O alerta de [[prompt-padrao-recomendacao|📝 prompt v1]] continua valendo só para o dia em que entrar um campo aberto no quiz.

> [!todo] Pendente da decisão sobre escala
> Com 180 cursos, cinco candidatos do mesmo eixo têm descrição quase idêntica e a LLM desempata no vazio. Nesse cenário os `candidatos` precisam carregar os atributos que **de fato** separam cursos irmãos: nível (iniciação / qualificação / técnico), carga horária, pré-requisito e eixo tecnológico. **Só entra no contrato depois de o grupo decidir se a camada de IA nasce para 18 ou já para 180.**

### 📄 C2 — saída da LLM · F2 valida

O **formato** é idêntico ao de [[camada-ia-plano-implementacao|🧩 plano]] (`principal` + 4 `alternativas`) e não muda.

O que muda é a **validação**: o eixo D acrescenta uma quinta regra às quatro que já existiam.

| # | Regra | Se violar |
|---|---|---|
| 1 | Todo `course_id` veio em `candidatos` | ❌ fallback |
| 2 | Exatamente 5 ids, sem repetição | ❌ fallback |
| 3 | Nenhum candidato sumiu | ❌ fallback |
| 4 | `texto` não vazio e dentro do limite | ❌ fallback |
| **5** | **O `principal` está dentro do `conjunto_empate`** | ❌ fallback |

> [!success] A regra 5 é o que torna o limite verificável em vez de confiável
> Antes, "não reordene fora do limiar" era só uma instrução em linguagem natural no prompt — o servidor não tinha como checar se foi obedecida. Com o conjunto de empate calculado pela engine e enviado no payload, **o servidor sabe exatamente quem podia ser promovido**. Quando o conjunto tem 1 elemento, a regra 5 sozinha garante que a ordem da engine foi respeitada. É cinto e suspensório, igual à regra 1 contra alucinação de curso.

> [!warning] A regra 2 do prompt v1 precisa ser reescrita antes de F2-04
> O texto atual em [[prompt-padrao-recomendacao|📝 Prompt v1]] fala em "diferença de `score` menor que 0,05". Com o eixo D isso sai e entra "escolha o principal **entre os ids listados em `conjunto_empate`**; se a lista tiver um id só, ele é o principal". É mais simples de o modelo obedecer e é verificável no servidor. Como o prompt ainda não foi copiado para arquivo, a correção é barata **agora** e cara depois de v1 estar congelada.

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

**Campos novos em `QuizAttempt`:** `llm_model`, `prompt_version`, `latency_ms`, `tokens_in`, `tokens_out`, `used_fallback`, `diverged`, `cache_hit` — mais `confidence_gap`, `confidence_band` e `tie_set`, que são do motor e não da IA.

> [!note] `tie_set` é `JSONField(default=list)` e precisa ficar gravado
> Guardar o conjunto de empate na tentativa é o que permite, três meses depois, auditar uma divergência: *"a LLM promoveu o curso 11 — ele estava no conjunto de empate daquela tentativa?"*. Sem isso o experimento do Passo 11 mede divergência sem conseguir dizer se ela era **permitida**.

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
| **F1-04** | Conjunto de empate (eixo D) + banda (eixo C) | `quiz/engine.py`, `quiz/models.py` | F1-02 | Testes: conjunto de 1 no perfil automotivo, conjunto ≥ 2 no elétrico, banda nas 3 faixas |
| **F1-03** | `build_payload(attempt)` → C1 | `quiz/delivery.py` | F1-02, **F1-04** | Teste compara o dict com o JSON de C1, campo a campo, com `confianca` e `respostas` |
| **F1-05** | Análise de sensibilidade | `quiz/management/commands/sensibilidade.py` | catálogo (qualquer) | CSV com Δposição por Δpeso; tabela pronta para o artigo |
| **F1-06** | Recalibração com os 18 cursos reais | seeds (F3) + revisão dos 4 critérios de aceite | **F3** | Os 4 perfis de aceite passam com o catálogo real |
| **F1-07** | Bateria nova de testes | `quiz/tests.py` | F1-01 a F1-04 | Contagem sobe de 12 para ~22 |
| **F1-08** | Seção de metodologia do cálculo | [[artigo-secao-calculo-cosseno\|✍️ artigo]] | F1-05, F1-06 | Texto com os números do catálogo real |

### 🎚️ F1-04 em detalhe — o indicador de confiança

São **duas funções puras**, uma para cada eixo. Nenhuma delas toca o banco.

```python
EPSILON_EMPATE = 0.05   # quiz/engine.py — margem do conjunto de empate

def conjunto_empate(ranking, epsilon=EPSILON_EMPATE):
    """Candidatos a menos de epsilon do topo. Sempre inclui o 1o colocado."""

def banda(score_top1):
    """alta | media | baixa — calibra a confianca do texto, nao a ordem."""
```

**Eixo D — conjunto de empate**

| Tamanho do conjunto | `quem_escolhe` | Efeito |
|---|---|---|
| 1 | `engine` | A ordem do motor vale. A regra 5 do C2 rejeita qualquer outra escolha |
| ≥ 2 | `llm` | A LLM escolhe entre os empatados, usando o bloco `respostas` |

**Eixo C — banda de confiança**

| Faixa | `banda` | Como o texto fala |
|---|---|---|
| score alto | `alta` | "esse curso combina muito com o seu perfil" |
| score médio | `media` | "esse é o que mais se aproxima do que você descreveu" |
| score baixo | `baixa` | "esse é o caminho mais próximo — vale conversar com um orientador" |

> [!important] Os dois eixos são independentes — e é isso que faz a combinação funcionar
> Um perfil pode ter conjunto de empate = 1 **e** banda baixa: a engine escolheu, e a LLM fala com cautela. Ou conjunto = 4 **e** banda alta: a LLM escolhe entre quatro ótimos candidatos e fala com segurança. Tratar os dois como a mesma coisa é o erro que a regra do limiar de 0,05 cometia.

> [!question] Os cortes das bandas ainda não têm número, e não deviam ter por chute
> Diferente do ε, que tem significado matemático (margem de empate), os cortes de banda são uma **escolha de produto**: a partir de qual score o sistema tem o direito de soar confiante ao recomendar uma formação profissional a alguém. Defina-os **depois** de rodar a distribuição de scores do catálogo real (é saída da F1-05) — e registre o critério, porque isso vai para o capítulo de ética. Chutar 0,30 e 0,70 agora é criar mais um número que ninguém sabe defender.

> [!success] Em compensação, o ε deixa de estar escrito no prompt
> O limiar de 0,05 vivia em dois lugares — Python e o texto do prompt em `.md` — e mudar um sem o outro era o risco declarado. Com o eixo D, **só o Python conhece o número**: o prompt recebe a lista de ids já calculada. A duplicação some, e a obediência da LLM vira verificável no servidor em vez de confiável.

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
| **F2-04** | `quiz/prompts/entrega_v1.md` — copiar de [[prompt-padrao-recomendacao\|📝 prompt v1]] **com a regra 2 reescrita** para o eixo D | 8 | ❌ | O arquivo e a nota batem, e a regra 2 fala em `conjunto_empate`, não em 0,05 |
| **F2-05** | Carregador do prompt: lê o arquivo e injeta `{{PERFIL_JSON}}` e `{{CANDIDATOS_JSON}}` | 8 | ❌ | Zero f-string de prompt em view ou serializer |
| **F2-06** | `DeliverySerializer` — as 5 regras de validação do C2 | 8 | ❌ | 7 testes de saída maliciosa (as 6 antigas + principal fora do `conjunto_empate`), todos em fallback e **nenhum** em 500 |
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
| Conjunto de empate: tamanho 1 no automotivo, ≥ 2 no elétrico | F1 | `SimpleTestCase` (função pura) |
| Banda de confiança nas três faixas | F1 | `SimpleTestCase` (função pura) |
| `principal` fora do `conjunto_empate` cai em fallback | F2 | `TestCase` |
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
| **1** | 24/08 – 30/08 | F1-01, F1-02, **F1-04**, F1-03 | F2-01, F2-02, F2-03 | **Contratos C1/C2/C3 anunciados** + camada offline de pé |
| **2** | 31/08 – 06/09 | F1-05 | F2-04, F2-05, F2-06 | Prompt em arquivo + validador que não deixa lixo entrar |
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

> [!todo] Sem estas sete, a semana 1 não começa limpa
> 0. **Ratificar o papel da IA (eixos D + C)** — substitui o limiar de 0,05, que já está furado em 3 dos 4 perfis de aceite. Muda C1, C2 e a regra 2 do prompt. **É mudança de contrato, então é decisão de grupo, não sua.**
> 0b. **A camada de IA nasce para 18 cursos ou já para 180?** Se for 180, os `candidatos` precisam dos atributos discriminantes (nível, carga horária, pré-requisito, eixo) já no C1 — cinco cursos do mesmo eixo têm descrição quase idêntica, e sem isso a LLM desempata no vazio.
> 0c. **O que o quadro quis dizer com `180 cursos → IA → 18 cursos`** — se 18 é o recorte do catálogo, o alerta de escopo está certo; se é a lista personalizada de cada pessoa, o alerta está calibrado errado e a nota [[escopo-fluxo-educmatch\|🗺️ escopo]] precisa ser reescrita. **De qualquer forma, quem reduz 180 → N é a engine, nunca a LLM** — se ela vir os 180, volta a poder alucinar curso e o sistema para sem internet.
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
