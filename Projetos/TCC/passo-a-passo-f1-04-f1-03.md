---
title: "Passo a passo — F1-04 e F1-03 (conjunto de empate, banda e build_payload)"
aliases: ["Passo a passo Bloco 1", "Como fazer F1-04", "build_payload", "Conjunto de empate no codigo"]
tags: [tcc, engine, ia, django, execucao, passo-a-passo, contrato]
status: em-andamento
projeto: TCC
criado: 2026-08-25
---

> [!info] Plano: [[plano-execucao-f1-f2|🗂️ Plano de execução F1+F2]] · Spec: [[spec-motor-e-ia-frentes-1-2|🧭 Spec F1+F2]] · Motor: [[engine-matching-cosseno|🧮 Engine]] · Nota anterior: [[passo-a-passo-f1-01-f1-02|🔧 Passo a passo F1-01 e F1-02]]

# 🎚️ Passo a passo — F1-04 e F1-03

> [!abstract] O que esta nota é
> O **Bloco 1** do [[plano-execucao-f1-f2|plano de execução]]: as duas funções puras que implementam os eixos **D** (conjunto de empate) e **C** (banda de confiança), a gravação delas na tentativa, e o `build_payload` que fecha o **contrato C1**.
> Escrito contra o repositório no commit `14be789`. Os números de linha do `engine.py` são os de depois do Bloco 0.

> [!warning] Nada disto foi executado
> O código foi escrito contra a leitura do repositório, não rodado. **A seção 5 é o que prova que deu certo.**

> [!success] As decisões deste bloco estão ratificadas
> Os eixos D e C foram aprovados pelo grupo. É o que libera escrever este código — antes disso, o `build_payload` estaria sendo construído sobre um contrato que ainda podia mudar.

---

## 🚦 Antes de abrir o editor

```bash
git checkout main && git pull
git checkout -b feat/motor-confianca-e-payload
```

> [!danger] Não encoste em `quiz/models.py` neste bloco
> Os campos `confidence_gap`, `confidence_band` e `tie_set` **já existem** desde o Bloco 0 — este bloco só os preenche. O colega da autenticação está trabalhando no `models.py` agora (AUT-04 acrescenta `QuizAttempt.user`). Abrir esse arquivo para acrescentar um helper ou uma property cria conflito no único arquivo que vocês dois disputam, e por conveniência.

---

## 1️⃣ F1-04 — as funções puras

Em `quiz/engine.py`, logo abaixo de `TOP_AREAS = 3` (linha 8):

```python
EPSILON_EMPATE = 0.05

# PROVISORIO: cortes de banda sao escolha de produto, nao matematica.
# Definir depois de ver a distribuicao real de scores (F1-05).
CORTE_BANDA_ALTA = 0.80
CORTE_BANDA_MEDIA = 0.50
```

E as três funções, depois de `rank_courses` (linha 58):

```python
def conjunto_empate(ranking, epsilon=EPSILON_EMPATE):
    """Candidatos a menos de epsilon do topo. O 1o colocado sempre entra."""
    if not ranking:
        return []
    topo = ranking[0][1]
    return [course.id for course, score, _ in ranking if topo - score < epsilon]


def banda(score_top1):
    """Calibra a confianca do texto. Nao mexe na ordem."""
    if score_top1 >= CORTE_BANDA_ALTA:
        return "alta"
    if score_top1 >= CORTE_BANDA_MEDIA:
        return "media"
    return "baixa"


def confianca(ranking):
    """Bloco `confianca` do contrato C1, a partir do ranking ja cortado no limite."""
    if not ranking:
        return {
            "score_top1": 0.0,
            "gap_top1_top2": 0.0,
            "banda": "baixa",
            "conjunto_empate": [],
            "quem_escolhe": "engine",
        }

    top1 = ranking[0][1]
    top2 = ranking[1][1] if len(ranking) > 1 else 0.0
    empate = conjunto_empate(ranking)

    return {
        "score_top1": round(top1, 4),
        "gap_top1_top2": round(top1 - top2, 4),
        "banda": banda(top1),
        "conjunto_empate": empate,
        "quem_escolhe": "llm" if len(empate) > 1 else "engine",
    }
```

> [!important] O conjunto de empate se calcula sobre o ranking **já cortado**
> `recommend` fatia em `[:limit]` antes. O conjunto tem que ser subconjunto dos candidatos que a LLM vai receber — se ele apontasse para um curso que não foi enviado no prompt, a regra 5 do contrato C2 rejeitaria a própria escolha que o servidor autorizou.

> [!note] `quem_escolhe` é derivado, não decidido
> Ele é `llm` quando o conjunto tem mais de um elemento, e `engine` quando tem um só. Não existe caminho em que alguém "escolha" esse campo — é consequência aritmética do ε. É o que impede a regra de virar negociação caso a caso.

> [!warning] Os cortes de banda vão mudar, e está certo assim
> `0.80` e `0.50` são chutes provisórios, marcados como tal no comentário. Com o catálogo atual quase todo perfil bom passa de 0,95, então na prática hoje quase tudo cai em `alta`. **Isso não é bug** — é a distribuição real de 12 cursos fictícios. Os números definitivos saem da F1-05, e o critério de escolha vira parágrafo do capítulo de ética. O que não pode é alguém apagar o comentário `PROVISORIO` e o número virar verdade por esquecimento.

---

## 2️⃣ F1-04 — gravar na tentativa

Em `recommend`, entre o fatiamento do ranking (linha 75) e o `delete()`:

```python
    ranking = rank_courses(profile, courses, area_names)[:limit]

    dados = confianca(ranking)
    attempt.confidence_gap = dados["gap_top1_top2"]
    attempt.confidence_band = dados["banda"]
    attempt.tie_set = dados["conjunto_empate"]
    attempt.save(update_fields=["confidence_gap", "confidence_band", "tie_set"])

    attempt.recommendations.all().delete()
```

> [!tip] `update_fields` não é otimização, é convivência
> Sem ele, o `save()` reescreve **todas** as colunas de `QuizAttempt` — inclusive o `user` que a AUT-04 está acrescentando. Numa tentativa recalculada, isso sobrescreveria o dono com o valor que estava em memória. Limitar os campos é o que faz o motor e a autenticação escreverem no mesmo modelo sem se pisarem.

> [!note] Continua tudo dentro do `@transaction.atomic`
> A gravação da confiança e o `bulk_create` das recomendações caem ou entram juntos. Uma tentativa com `tie_set` de um ranking e recomendações de outro seria um estado impossível de auditar depois.

---

## 3️⃣ F1-03 — `quiz/delivery.py`, arquivo novo

```python
"""Monta o que a camada de IA consome. Contrato C1 da spec das frentes 1 e 2."""

from catalog.models import Area
from quiz.engine import profile_vector


def build_payload(attempt):
    recomendacoes = list(
        attempt.recommendations.select_related("course").order_by("rank_engine")
    )
    answers = list(
        attempt.answers.select_related("question", "choice")
        .prefetch_related("choice__area_weights__area")
        .order_by("question__order")
    )

    area_names = dict(Area.objects.values_list("slug", "name"))
    perfil = profile_vector([answer.choice for answer in answers])
    fortes = sorted(perfil.items(), key=lambda item: (-item[1], item[0]))

    return {
        "perfil": {
            "areas_fortes": [
                {"area": slug, "area_name": area_names.get(slug, slug), "pontuacao": valor}
                for slug, valor in fortes
                if valor > 0
            ],
            "respostas_dadas": len(answers),
        },
        "respostas": [
            {"pergunta": answer.question.text, "escolha": answer.choice.text}
            for answer in answers
        ],
        "confianca": {
            "score_top1": recomendacoes[0].score if recomendacoes else 0.0,
            "gap_top1_top2": attempt.confidence_gap or 0.0,
            "banda": attempt.confidence_band or "baixa",
            "conjunto_empate": attempt.tie_set or [],
            "quem_escolhe": "llm" if len(attempt.tie_set or []) > 1 else "engine",
        },
        "candidatos": [
            {
                "course_id": rec.course_id,
                "nome": rec.course.name,
                "descricao": rec.course.description,
                "score": rec.score,
                "rank_engine": rec.rank_engine,
                "top_areas": [
                    {"area_name": area["area_name"], "percentual": area["percentual"]}
                    for area in rec.explanation.get("top_areas", [])
                ],
            }
            for rec in recomendacoes
        ],
    }
```

> [!important] Lê do banco, não recalcula o ranking
> O `recommend` já rodou e gravou. Se o `build_payload` recalculasse, existiriam **duas fontes de verdade** para o que a LLM recebe — e no dia em que alguém mexer no `engine.py` sem rodar os seeds de novo, o prompt passa a falar de um ranking que não é o que está na tela. Ler do banco é o que garante que payload e tela contam a mesma história.

> [!warning] `order_by("rank_engine")`, explícito, mesmo já existindo `Meta.ordering`
> O `ordering` do modelo é `rank_final` desde o Bloco 0 — o que a pessoa viu. Aqui você precisa da ordem **do motor**, porque é ela que o campo `rank_engine` do payload declara. Herdar o ordering padrão faria a lista sair na ordem entregue e o `rank_engine` de cada item não bateria com a posição dele no array. É o tipo de inconsistência que a LLM obedece sem reclamar e que ninguém percebe até ler o CSV do experimento.

> [!note] O bloco `respostas` é o que faz o eixo D funcionar
> Sem ele, a LLM recebe os mesmos números que já empataram e desempata no escuro. As alternativas são fechadas e escritas pelo grupo, então não há superfície de injeção de prompt — o alerta de [[prompt-padrao-recomendacao|📝 prompt v1]] vale só para o dia em que entrar campo aberto no quiz.

---

## 4️⃣ Os testes

### `quiz/tests.py` — os imports, primeiro

> [!danger] Faça esta parte antes de colar qualquer teste
> Cinco dos seis erros que aparecem ao rodar a suíte pela primeira vez neste bloco são **import faltando** — `banda`, `confianca` e, principalmente, `build_payload`, que vem de `quiz.delivery` e não da engine. Troque o topo do arquivo inteiro:

```python
# quiz/tests.py
from io import StringIO

from django.core.management import call_command
from django.test import SimpleTestCase, TestCase, override_settings

from catalog.models import Area
from quiz.delivery import build_payload
from quiz.engine import banda, confianca, cosine_similarity, recommend
from quiz.models import Answer, Choice, QuizAttempt
```

Isso também elimina a linha duplicada de `django.test` que sobrou do Bloco 0 — o `override_settings` tinha sido acrescentado num import novo em vez de no que já existia.

### `quiz/tests.py` — funções puras, sem banco

Numa classe nova, ao lado de `MatematicaTest`:

```python
class ConfiancaTest(SimpleTestCase):
    """Eixos D e C: recebem numeros, devolvem numeros. Nao tocam o banco."""

    class CursoFalso:
        def __init__(self, id, name):
            self.id, self.name = id, name

    def ranking(self, *scores):
        return [(self.CursoFalso(i, f"Curso {i}"), s, {}) for i, s in enumerate(scores, 1)]

    def test_diferenca_real_deixa_a_engine_decidir(self):
        """Perfil automotivo: 0,968 x 0,891. Gap de 0,077 nao pode ser reordenado."""
        dados = confianca(self.ranking(0.968, 0.891, 0.5, 0.4, 0.3))
        self.assertEqual(dados["conjunto_empate"], [1])
        self.assertEqual(dados["quem_escolhe"], "engine")

    def test_empate_tecnico_entrega_a_escolha_para_a_llm(self):
        """Perfil eletrico com 6 respostas: 0,9877 x 0,9875."""
        dados = confianca(self.ranking(0.9877, 0.9875, 0.4, 0.3, 0.2))
        self.assertEqual(dados["conjunto_empate"], [1, 2])
        self.assertEqual(dados["quem_escolhe"], "llm")

    def test_banda_nas_tres_faixas(self):
        self.assertEqual(banda(0.99), "alta")
        self.assertEqual(banda(0.60), "media")
        self.assertEqual(banda(0.10), "baixa")

    def test_ranking_vazio_nao_estoura(self):
        dados = confianca([])
        self.assertEqual(dados["conjunto_empate"], [])
        self.assertEqual(dados["quem_escolhe"], "engine")
```

### `quiz/tests.py` — gravação e payload

Dentro de `RecomendacaoTest`, **no nível de 4 espaços**:

> [!warning] O `def` desalinhado vira erro de banco, não erro de sintaxe
> Se um `def test_*` entrar com 8 espaços, ele vira função aninhada dentro do teste anterior: só a primeira linha do corpo acompanha, e as seguintes voltam a pertencer ao teste de fora. O sintoma é um `UNIQUE constraint failed: quiz_answer.attempt_id, quiz_answer.question_id` — porque o teste anterior passa a criar duas respostas para a mesma pergunta. **E o teste aninhado nem chega a ser coletado**, então a contagem final fecha com um a menos e ninguém percebe. Se der 20 em vez de 21, é isto.

```python
    def test_confianca_gravada_na_tentativa(self):
        attempt = QuizAttempt.objects.create(respondent_name="teste")
        choice = Choice.objects.get(text__icontains="Eletricidade")
        Answer.objects.create(attempt=attempt, question=choice.question, choice=choice)
        recommend(attempt)
        attempt.refresh_from_db()
        self.assertIsNotNone(attempt.confidence_gap)
        self.assertIn(attempt.confidence_band, ["alta", "media", "baixa"])
        self.assertTrue(attempt.tie_set)

    def test_quiz_em_branco_da_banda_baixa(self):
        attempt = QuizAttempt.objects.create(respondent_name="ninguem respondeu")
        recommend(attempt)
        attempt.refresh_from_db()
        self.assertEqual(attempt.confidence_band, "baixa")

    def test_payload_bate_o_contrato_c1(self):
        attempt = QuizAttempt.objects.create(respondent_name="teste")
        for trecho in ("Montar e ligar", "Chão de fábrica", "Eletricidade", "alicate"):
            escolha = Choice.objects.get(text__icontains=trecho)
            Answer.objects.create(attempt=attempt, question=escolha.question, choice=escolha)
        recommend(attempt)
        attempt.refresh_from_db()

        payload = build_payload(attempt)

        self.assertEqual(
            sorted(payload), ["candidatos", "confianca", "perfil", "respostas"]
        )
        self.assertEqual(payload["perfil"]["respostas_dadas"], 4)
        self.assertEqual(len(payload["respostas"]), 4)
        self.assertEqual(len(payload["candidatos"]), 5)
        self.assertEqual(
            [c["rank_engine"] for c in payload["candidatos"]], [1, 2, 3, 4, 5]
        )
        # Todo id do conjunto de empate tem que estar entre os candidatos enviados.
        ids = {c["course_id"] for c in payload["candidatos"]}
        self.assertTrue(set(payload["confianca"]["conjunto_empate"]) <= ids)
```

> [!success] A última asserção é a que protege o contrato C2
> Se o `conjunto_empate` contiver um id que não foi enviado em `candidatos`, o servidor estaria autorizando a LLM a escolher um curso que ela nunca viu — e a regra 5 rejeitaria a escolha, jogando toda tentativa em fallback sem ninguém entender por quê. É uma linha de teste que fecha um bug que só apareceria no Bloco 3.

> [!question] O quiz em branco entrega a escolha à LLM, e vale observar isso
> Com todas as respostas zeradas, todos os scores são 0,0, o conjunto de empate fica com os 5 e `quem_escolhe` vira `llm`. É coerente — sem informação, nenhum candidato é melhor que outro — e a banda `baixa` garante que o texto não finja certeza. Mas anote para o experimento (F2-10): se muitas tentativas reais caírem nesse caso, o problema é o quiz, não o motor.

---

## 5️⃣ Verificação

```bash
./.venv/Scripts/python.exe manage.py makemigrations --check --dry-run
```

Tem que dizer **No changes detected** — este bloco não mexe em modelo. Se ele quiser criar migration, você encostou no `models.py` sem querer.

```bash
PYTHONIOENCODING=utf-8 ./.venv/Scripts/python.exe manage.py test
```

**21 testes verdes** — os 14 de antes mais os 7 novos.

E o olho humano, que nenhum teste substitui:

```bash
PYTHONIOENCODING=utf-8 ./.venv/Scripts/python.exe manage.py shell
```

```python
from quiz.models import QuizAttempt
from quiz.delivery import build_payload
import json
print(json.dumps(build_payload(QuizAttempt.objects.first()), indent=2, ensure_ascii=False))
```

Leia o JSON inteiro uma vez. É exatamente o texto que a LLM vai receber no Bloco 3 — se algo estiver estranho aqui, estará estranho no prompt.

---

## 6️⃣ Commit e PR

```bash
git add -A && git commit -m "feat(motor): conjunto de empate, banda de confianca e build_payload"
git push -u origin feat/motor-confianca-e-payload
```

No PR, o campo que importa:

```markdown
## Toca contrato de outra frente? ( ) não (x) sim → Frente 2 (você mesmo) e Frente 4
Fecha o contrato C1. A partir daqui a camada de IA tem entrada estavel.
Nada muda na API nem na tela — este bloco so acrescenta.
```

> [!note] Este PR não quebra ninguém
> Diferente do Bloco 0, aqui é tudo adição: função nova, arquivo novo, campos que já existiam sendo preenchidos. O colega da autenticação pode continuar sem nem atualizar a branch dele.

## ▶️ Próxima ação

**Bloco 2** — `feat/ia-provider-offline`: protocolo `LLMProvider`, `FakeProvider` e as variáveis `LLM_*`. É a primeira vez que você escreve no `settings.py` desde o Bloco 0; se o colega ainda não tiver mesclado a autenticação, escreva **no fim do arquivo**, que é onde o bloco de LLM mora.

## 📎 Veja também

- [[plano-execucao-f1-f2|🗂️ Plano de execução F1+F2]] — onde este bloco se encaixa
- [[spec-motor-e-ia-frentes-1-2|🧭 Spec F1+F2]] — o contrato C1 completo e o porquê dos dois eixos
- [[passo-a-passo-f1-01-f1-02|🔧 Passo a passo F1-01 e F1-02]] — o bloco anterior
- [[engine-matching-cosseno|🧮 Engine de matching]] · [[camada-ia-plano-implementacao|🧩 Camada de IA]] · [[prompt-padrao-recomendacao|📝 Prompt v1]]
- **Conceitos:** [[tst-testes-django|Testes em Django]] · [[rec-metricas-similaridade|Métricas de similaridade]] · [[ia-engenharia-de-prompt|Engenharia de prompt]]
- [[TCC|🎓 TCC]] · [[Projetos|🚀 Projetos]]
