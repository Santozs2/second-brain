---
title: "Passo a passo — F2-07 (orquestração da entrega: deliver, divergência e fallback)"
aliases: ["Passo a passo F2-07", "Como fazer o deliver", "Orquestração da camada de IA", "Fallback da LLM"]
tags: [tcc, ia, llm, django, execucao, passo-a-passo, fallback]
status: em-andamento
projeto: TCC
criado: 2026-08-27
---

> [!info] Spec: [[spec-motor-e-ia-frentes-1-2|🧭 Spec das Frentes 1 e 2]] · Plano: [[camada-ia-plano-implementacao|🧩 Camada de IA]] · Prompt: [[prompt-padrao-recomendacao|📝 Prompt v1]] · Testes: [[testes-e-validacao-tcc|✅ Testes]]
> **Vem depois de:** F2-04, F2-05 e F2-06 · **Habilita:** [[passo-a-passo-f2-09|🔌 F2-09 (endpoint C3)]] e [[passo-a-passo-f2-08|🌐 F2-08 (Gemini)]]

# 🎛️ Passo a passo — F2-07

> [!abstract] O que esta nota é
> O **como** da tarefa F2-07 da [[spec-motor-e-ia-frentes-1-2|spec]]: a função que amarra a camada inteira — monta o payload, chama o provedor, valida contra o C2, grava no banco e decide se cai em fallback.
> Escrito contra o repositório no commit `1b4f48e`, branch `feat/ia-prompt-e-validacao`.

> [!warning] Nada disto foi executado
> O código abaixo foi escrito contra a leitura do repositório, não rodado. **A seção 5️⃣ é o que prova que deu certo** — não pule nenhum comando.

> [!danger] O bloco anterior não está fechado, e a F2-07 não sobe em cima dele
> Os 9 testes de `ValidacaoTest` em `quiz/tests_llm.py` estão com corpo `...`: passam sem executar nada. E o código que eles deveriam cobrir tem quatro defeitos que os fariam falhar assim que ganhassem corpo. **A seção 0️⃣ é obrigatória** — sem ela, `from quiz.delivery import validar_entrega` levanta `ImportError` e nada nesta nota roda.

---

## 🚦 Antes de abrir o editor

```bash
git checkout -b feat/ia-orquestracao
```

Não precisa avisar o grupo desta vez: a F2-07 só toca `quiz/delivery.py`, `quiz/serializers.py`, as duas views e os testes. **Nenhum contrato público muda** — a API e as telas continuam com a mesma forma de hoje. Quem mexe em contrato é a [[passo-a-passo-f2-09|F2-09]].

---

## 0️⃣ Fechar a F2-06 de verdade

### `quiz/serializers.py` — linha 66

`AlternativaSerializers` não herda de nada. Do jeito que está, o campo `alternativas` não valida lista nenhuma, e **as regras 1, 2 e 3 do contrato C2 nunca rodam** — o `validate()` inteiro está inalcançável.

```python
class AlternativaSerializers(ItemEntregaSerializers):
    texto = serializers.CharField(allow_blank=False, trim_whitespace=True, max_length=220)


class DeliverySerializers(serializers.Serializer):
    principal = PrincipalSerializers()
    alternativas = AlternativaSerializers(many=True)
    """ 5 Regras do contrato C2"""
```

> [!warning] `many=True` não é detalhe de estilo
> Sem ele, `attrs["alternativas"]` não é uma lista, e a primeira linha do `validate()` (`[item["course_id"] for item in attrs["alternativas"]]`) estoura antes de qualquer regra ser checada. Um serializer que **explode** em vez de **recusar** é um 500 na cara do usuário — exatamente o que a coluna "se violar → fallback" da spec proíbe.

### `quiz/delivery.py` — tirar as funções de dentro da exceção

Hoje `parse_saida` e `validar_entrega` estão indentadas **dentro** de `class EntregaInvalida`. Viraram métodos de uma exceção: não dá para importar nenhuma das duas. Troque o topo do arquivo (linhas 1 a 27) por:

```python
import json
import re
import time

from django.conf import settings
from django.db import transaction

from catalog.models import Area
from quiz.engine import profile_vector
from quiz.llm import get_provider
from quiz.llm.base import LLMTimeout, LLMUnavailable
from quiz.models import Recommendation
from quiz.serializers import DeliverySerializers

CERCA = re.compile(r"^\s*```(?:json)?\s*|\s*```\s*$", re.MULTILINE)


class EntregaInvalida(Exception):
    """A LLM respondeu fora do contrato C2. Sempre vira fallback, nunca 500."""


def parse_saida(texto_cru):
    limpo = CERCA.sub("", texto_cru or "").strip()
    try:
        return json.loads(limpo)
    except (json.JSONDecodeError, TypeError) as erro:
        raise EntregaInvalida(f"JSON invalido: {erro}") from erro


def validar_entrega(texto_cru, payload):
    dados = parse_saida(texto_cru)
    serializer = DeliverySerializers(
        data=dados,
        ids_permitidos=[c["course_id"] for c in payload["candidatos"]],
        conjunto_empate=payload["confianca"]["conjunto_empate"],
    )
    if not serializer.is_valid():
        raise EntregaInvalida(serializer.errors)
    return serializer.validated_data
```

> [!note] Três correções escondidas nesse bloco
> 1. `conjunto_emapte=` → `conjunto_empate=`. O `__init__` do serializer espera o nome certo; o errado dava `TypeError` na primeira chamada de verdade.
> 2. `serializer.erros` → `serializer.errors`. O DRF expõe `.errors`, e `.erros` levantaria `AttributeError` **exatamente no caminho de recusa** — o caminho que existe justamente para não quebrar.
> 3. `validar_entrega` deixa de receber `parse_saida` como parâmetro. Injetar uma função do próprio módulo não é injeção de dependência, é uma assinatura a mais para errar.

### `quiz/tests_llm.py` — dar corpo aos 9 testes

No import do topo:

```python
from quiz.delivery import EntregaInvalida, validar_entrega
```

E a classe inteira, no lugar dos `...`:

```python
class ValidacaoTest(SimpleTestCase):
    """Cada teste representa uma forma de a LLM sair do combinado."""

    def entrega(self, principal=7, alternativas=(3, 11), texto="Texto valido para o teste."):
        """Monta uma saida C2 valida e deixa cada teste estragar so um pedaco dela."""
        return json.dumps(
            {
                "principal": {"course_id": principal, "texto": texto},
                "alternativas": [
                    {"course_id": cid, "texto": "Alternativa valida."} for cid in alternativas
                ],
            },
            ensure_ascii=False,
        )

    def test_saida_feliz_passa(self):
        dados = validar_entrega(self.entrega(), PAYLOAD_FALSO)
        self.assertEqual(dados["principal"]["course_id"], 7)
        self.assertEqual(len(dados["alternativas"]), 2)

    def test_cerca_de_codigo_e_tolerada(self):
        """Modelo devolvendo cerca de codigo e o caso comum, nao a excecao."""
        cru = "```json\n" + self.entrega() + "\n```"
        self.assertEqual(validar_entrega(cru, PAYLOAD_FALSO)["principal"]["course_id"], 7)

    def test_json_quebrado_recusa(self):
        with self.assertRaises(EntregaInvalida):
            validar_entrega("{isso nao e json", PAYLOAD_FALSO)

    def test_curso_fantasma_recusa(self):
        """Regra 1: id que nunca foi enviado no prompt."""
        with self.assertRaises(EntregaInvalida):
            validar_entrega(self.entrega(alternativas=(3, 999)), PAYLOAD_FALSO)

    def test_curso_repetido_recusa(self):
        """Regra 2."""
        with self.assertRaises(EntregaInvalida):
            validar_entrega(self.entrega(alternativas=(7, 3)), PAYLOAD_FALSO)

    def test_candidato_omitido_recusa(self):
        """Regra 3."""
        with self.assertRaises(EntregaInvalida):
            validar_entrega(self.entrega(alternativas=(3,)), PAYLOAD_FALSO)

    def test_texto_vazio_recusa(self):
        """Regra 4: so espaco em branco nao e texto."""
        with self.assertRaises(EntregaInvalida):
            validar_entrega(self.entrega(texto="   "), PAYLOAD_FALSO)

    def test_texto_gigante_recusa(self):
        """Regra 4: o principal tem teto de 600 caracteres."""
        with self.assertRaises(EntregaInvalida):
            validar_entrega(self.entrega(texto="a" * 601), PAYLOAD_FALSO)

    def test_principal_fora_do_conjunto_de_empate_recusa(self):
        """Regra 5: o 11 nao esta em conjunto_empate=[7, 3]."""
        with self.assertRaises(EntregaInvalida):
            validar_entrega(self.entrega(principal=11, alternativas=(7, 3)), PAYLOAD_FALSO)
```

E apague o `def test_curso_fantasma_recusa(self)` solto no fim do arquivo (linha 121). Ele está fora de qualquer classe, então o `unittest` nunca o coleta — e chama um nome que nem existe nos imports.

> [!tip] `test_curso_repetido_recusa` viola duas regras ao mesmo tempo
> Com `alternativas=(7, 3)`, o id 7 repete **e** o 11 some. As duas regras recusam, e o teste passa de qualquer jeito porque só afirma o tipo da exceção. Está certo assim: o contrato promete *recusar*, não promete *qual mensagem*. Se um dia quiser isolar a regra, aumente a lista de `candidatos` do fixture — não afrouxe a asserção.

---

## 1️⃣ O coração da F2-07 — `deliver(attempt)`

No fim de `quiz/delivery.py`, depois de `build_payload`:

```python
def _fallback(attempt, motivo=""):
    """Devolve a tentativa para a ordem do motor. Idempotente de proposito:
    reprocessar uma tentativa nao pode deixar texto velho de LLM na tela."""
    recs = list(attempt.recommendations.all())
    for rec in recs:
        rec.rank_final = rec.rank_engine
        rec.is_primary = rec.rank_engine == 1
        rec.llm_text = ""
    Recommendation.objects.bulk_update(recs, ["rank_final", "is_primary", "llm_text"])

    attempt.used_fallback = bool(motivo)
    attempt.diverged = False
    attempt.save(update_fields=["used_fallback", "diverged"])
    return sorted(recs, key=lambda rec: rec.rank_final)


def _aplicar(attempt, dados, latency_ms):
    recs = {rec.course_id: rec for rec in attempt.recommendations.all()}
    principal_id = dados["principal"]["course_id"]

    textos = {principal_id: dados["principal"]["texto"]}
    textos.update({item["course_id"]: item["texto"] for item in dados["alternativas"]})

    # O principal vai para a frente; o resto mantem a ordem que o motor calculou.
    resto = sorted(
        (cid for cid in recs if cid != principal_id),
        key=lambda cid: recs[cid].rank_engine,
    )
    for posicao, cid in enumerate([principal_id] + resto, start=1):
        rec = recs[cid]
        rec.rank_final = posicao
        rec.is_primary = posicao == 1
        rec.llm_text = textos[cid]
    Recommendation.objects.bulk_update(
        recs.values(), ["rank_final", "is_primary", "llm_text"]
    )

    attempt.diverged = recs[principal_id].rank_engine != 1
    attempt.used_fallback = False
    attempt.llm_model = settings.LLM_MODEL
    attempt.prompt_version = settings.LLM_PROMPT_VERSION
    attempt.latency_ms = latency_ms
    attempt.save(
        update_fields=[
            "diverged", "used_fallback", "llm_model", "prompt_version", "latency_ms",
        ]
    )
    return sorted(recs.values(), key=lambda rec: rec.rank_final)


@transaction.atomic
def deliver(attempt):
    """Entrega final da tentativa. Nunca levanta: ou aplica a LLM, ou cai no motor."""
    provider = get_provider()
    if provider is None:
        return _fallback(attempt)

    payload = build_payload(attempt)
    if not payload["candidatos"]:
        return _fallback(attempt)

    from quiz.prompt import montar

    inicio = time.monotonic()
    try:
        cru = provider.complete(montar(payload))
        dados = validar_entrega(cru, payload)
    except (LLMTimeout, LLMUnavailable, EntregaInvalida):
        return _fallback(attempt, motivo="provedor ou contrato")

    return _aplicar(attempt, dados, latency_ms=int((time.monotonic() - inicio) * 1000))
```

> [!success] `used_fallback` só é `True` quando alguma coisa **falhou**
> Rodar com `LLM_ENABLED=false` não é falha, é configuração. Se o desligado contasse como fallback, o CSV do experimento (F2-10) mostraria uma taxa inflada por toda execução offline — e o número que vai para a monografia deixaria de significar "quantas vezes a LLM saiu do combinado". Desligado devolve a ordem do motor com `used_fallback=False`; provedor quebrado devolve a mesma ordem com `True`.

> [!warning] `except` largo, de propósito — mas só nestes três
> `LLMTimeout`, `LLMUnavailable` e `EntregaInvalida` são as três formas **previstas** de a camada falhar, e as três têm a mesma resposta: entregar o motor. Qualquer outra exceção sobe. Um `except Exception` aqui transformaria um `AttributeError` seu num fallback silencioso, e você descobriria o bug meses depois, olhando para uma taxa de fallback alta sem entender por quê.

> [!note] Por que o `import montar` está dentro da função
> `quiz/prompt.py` lê um arquivo do disco. Deixar o import no topo faz `quiz/delivery.py` — que a `tests.py` já importa só para testar `build_payload` — carregar o carregador de prompt em toda importação do módulo. Dentro da função, quem não chama `deliver()` não paga por isso.

> [!important] `diverged` mede promoção, não texto
> `diverged=True` quer dizer que a LLM colocou na frente alguém que o motor não tinha colocado — e a regra 5 do C2 garante que ela só conseguiu fazer isso **dentro do `conjunto_empate`**. É essa combinação que torna a coluna `diverged` do experimento auditável: toda divergência registrada era permitida por construção, e o `tie_set` gravado na tentativa prova isso três meses depois.

---

## 2️⃣ Chamar a entrega nos dois fluxos

### `quiz/web_views.py` — linha 56

```python
        recommend(attempt)
        deliver(attempt)
```

E no import do topo:

```python
from quiz.delivery import deliver
```

### `quiz/views.py` — `SubmitQuizView.post`, linha 36

```python
            recomendacoes = recommend(attempt)
            deliver(attempt)
```

Import junto do `recommend`:

```python
from quiz.delivery import deliver
```

> [!warning] `recomendacoes` fica velho depois do `deliver`
> `recommend()` devolve os objetos que o `bulk_create` criou; `deliver()` grava por cima **deles no banco**, não na sua lista em memória. Na `SubmitQuizView` isso importa: o JSON de resposta sairia com `rank_final` e `llm_text` antigos. Releia do banco antes de serializar:
> ```python
>             deliver(attempt)
>         recomendacoes = attempt.recommendations.select_related("course__main_area")
> ```
> (a segunda linha já fora do `with transaction.atomic()`, junto do `return`). É o tipo de bug que nenhum teste unitário pega e que aparece como "o texto some na API mas aparece no site" — ou o contrário.

---

## 3️⃣ Os testes novos

Em `quiz/tests_llm.py`, no topo:

```python
from io import StringIO
from unittest import mock

from django.core.management import call_command
from django.test import SimpleTestCase, TestCase, override_settings

from quiz.delivery import deliver
from quiz.engine import recommend
from quiz.llm.base import LLMTimeout
from quiz.models import Answer, Choice, QuizAttempt
```

E a classe nova, no fim do arquivo:

```python
@override_settings(LLM_ENABLED=True, LLM_PROVIDER="fake")
class DeliverTest(TestCase):
    """A orquestracao inteira, com o FakeProvider no lugar da rede."""

    ELETRICO = ("Montar e ligar", "Chão de fábrica", "Eletricidade", "alicate")
    AUTOMOTIVO = ("Desmontar máquinas", "Oficina", "Eletricidade", "diagnóstico")

    @classmethod
    def setUpTestData(cls):
        silencio = StringIO()
        for comando in ("seed_areas", "seed_courses", "seed_questions"):
            call_command(comando, stdout=silencio)

    def responder(self, *trechos):
        attempt = QuizAttempt.objects.create(respondent_name="teste")
        for trecho in trechos:
            choice = Choice.objects.get(text__icontains=trecho)
            Answer.objects.create(attempt=attempt, question=choice.question, choice=choice)
        recommend(attempt)
        attempt.refresh_from_db()
        return attempt

    @override_settings(LLM_ENABLED=False)
    def test_desligada_nao_chama_provedor_e_mantem_a_ordem_do_motor(self):
        """O botao de panico da defesa, exercido de ponta a ponta."""
        attempt = self.responder(*self.ELETRICO)
        recs = deliver(attempt)
        attempt.refresh_from_db()
        self.assertTrue(all(r.rank_final == r.rank_engine for r in recs))
        self.assertTrue(all(r.llm_text == "" for r in recs))
        self.assertFalse(attempt.used_fallback)
        self.assertFalse(attempt.diverged)

    def test_texto_da_llm_chega_em_todos_os_cursos(self):
        attempt = self.responder(*self.ELETRICO)
        recs = deliver(attempt)
        self.assertTrue(all(r.llm_text.strip() for r in recs))
        self.assertEqual(sum(1 for r in recs if r.is_primary), 1)

    def test_promocao_dentro_do_empate_reescreve_rank_final(self):
        attempt = self.responder(*self.ELETRICO)
        self.assertGreater(len(attempt.tie_set), 1, "perfil sem empate: teste sem objeto")
        segundo = attempt.recommendations.get(rank_engine=2).course_id

        with mock.patch(
            "quiz.delivery.get_provider", return_value=FakeProvider(promover_indice=1)
        ):
            recs = deliver(attempt)

        attempt.refresh_from_db()
        self.assertEqual(recs[0].course_id, segundo)
        self.assertTrue(recs[0].is_primary)
        self.assertTrue(attempt.diverged)
        self.assertFalse(attempt.used_fallback)

    def test_promocao_fora_do_empate_cai_em_fallback(self):
        """A regra 5 do C2 defendendo a ordem do motor quando o empate tem 1 so."""
        attempt = self.responder(*self.AUTOMOTIVO)
        self.assertEqual(len(attempt.tie_set), 1, "perfil empatado: teste sem objeto")

        with mock.patch(
            "quiz.delivery.get_provider", return_value=FakeProvider(promover_indice=1)
        ):
            recs = deliver(attempt)

        attempt.refresh_from_db()
        self.assertTrue(all(r.rank_final == r.rank_engine for r in recs))
        self.assertTrue(attempt.used_fallback)
        self.assertFalse(attempt.diverged)

    def test_timeout_do_provedor_nao_derruba_a_pagina(self):
        class Travado:
            def complete(self, prompt):
                raise LLMTimeout("8s")

        attempt = self.responder(*self.ELETRICO)
        with mock.patch("quiz.delivery.get_provider", return_value=Travado()):
            recs = deliver(attempt)

        attempt.refresh_from_db()
        self.assertTrue(attempt.used_fallback)
        self.assertTrue(all(r.llm_text == "" for r in recs))

    def test_metadados_ficam_gravados_na_tentativa(self):
        attempt = self.responder(*self.ELETRICO)
        deliver(attempt)
        attempt.refresh_from_db()
        self.assertEqual(attempt.prompt_version, "v1")
        self.assertTrue(attempt.llm_model)
        self.assertIsNotNone(attempt.latency_ms)
```

> [!tip] `mock.patch("quiz.delivery.get_provider")`, não `"quiz.llm.get_provider"`
> `delivery.py` faz `from quiz.llm import get_provider`, então o nome que ele usa mora no **namespace do delivery**. Trocar o original em `quiz.llm` não muda a referência que o módulo já capturou: o patch parece aplicado, a chamada real acontece do mesmo jeito e o teste falha de um jeito confuso. É o erro de mock mais comum em Python — [[tst-mocks-e-dubles|🎭 Mocks e dublês]].

> [!note] Os dois `assert` de pré-condição não são paranoia
> `test_promocao_dentro_do_empate` só significa alguma coisa se o perfil elétrico realmente empatar, e `test_promocao_fora_do_empate` só significa alguma coisa se o automotivo realmente **não** empatar. Quando a F3 trocar o catálogo pelos 18 cursos reais, essas premissas podem virar mentira — e sem as duas asserções os testes continuariam verdes testando nada. Com elas, a recalibração da F1-06 avisa na hora, e a mensagem já diz o que aconteceu.

---

## 4️⃣ Registrado, não resolvido: o ε ainda é um chute

Não é tarefa desta nota, mas anote antes que vire pergunta de banca: `quiz/engine.py:9` tem `EPSILON_EMPATE = 0.05`, e é ele que decide quem a LLM pode promover. A [[spec-motor-e-ia-frentes-1-2|spec]] manda definir os cortes **depois** da análise de sensibilidade (F1-05). Até lá, todo `diverged=True` do experimento está condicionado a um 0,05 que ninguém defendeu ainda.

---

## 5️⃣ Verificação

```bash
PYTHONIOENCODING=utf-8 ./.venv/Scripts/python.exe manage.py test
```

**45 testes verdes** — os 39 de hoje mais os 6 do `DeliverTest`. Os 9 do `ValidacaoTest` não mexem na contagem: eles já existiam como esqueleto, e agora passam a **testar** em vez de só passar.

> [!important] O teste que só o navegador dá
> ```bash
> ./.venv/Scripts/python.exe manage.py runserver 8010
> ```
> Responda o quiz com `LLM_ENABLED=true` / `LLM_PROVIDER=fake` no `.env`, e depois de novo com `LLM_ENABLED=false`. **As duas telas têm que abrir.** A segunda sem texto de LLM e sem quebrar é a demonstração do botão de pânico — é exatamente a cena que você quer ter ensaiado antes da defesa, não descoberto durante.

Depois, no `/admin`, abra uma tentativa entregue: `used_fallback`, `diverged`, `latency_ms` e `prompt_version` preenchidos, e as 5 recomendações com `llm_text`.

---

## 6️⃣ Commit

```bash
git add -A && git commit -m "feat(ia): orquestracao da entrega com divergencia auditavel e fallback"
```

Se preferir dois commits, corte entre a seção 0️⃣ (fechar a F2-06) e o resto. A correção do serializer é uma coisa, o `deliver()` é outra, e quem revisar agradece.

## 📎 Veja também

- [[passo-a-passo-f2-09|🔌 F2-09 — endpoint de entrega (C3)]] — o próximo, e o que desbloqueia a Frente 4
- [[passo-a-passo-f2-08|🌐 F2-08 — GeminiProvider, cache e timeout]] — o último, porque é o único que depende da credencial
- [[spec-motor-e-ia-frentes-1-2|🧭 Spec das Frentes 1 e 2]] · [[camada-ia-plano-implementacao|🧩 Plano da camada de IA]] · [[prompt-padrao-recomendacao|📝 Prompt v1]]
- **Conceitos:** [[ia-alucinacao-e-grounding|Alucinação e grounding]] · [[tst-mocks-e-dubles|Mocks e dublês]] · [[tst-testes-django|Testes em Django]]
- [[bugs-e-licoes-tcc|🐛 Bugs e lições]] · [[testes-e-validacao-tcc|✅ Testes e validação]]
- [[TCC|🎓 TCC]] · [[Projetos|🚀 Projetos]]
