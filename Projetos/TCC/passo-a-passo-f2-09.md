---
title: "Passo a passo — F2-09 (endpoint de entrega, contrato C3, e o texto na tela)"
aliases: ["Passo a passo F2-09", "Endpoint de entrega", "Contrato C3", "Como publicar a entrega para a F4"]
tags: [tcc, ia, llm, django, drf, api, execucao, passo-a-passo, contrato]
status: em-andamento
projeto: TCC
criado: 2026-08-27
---

> [!info] Spec: [[spec-motor-e-ia-frentes-1-2|🧭 Spec das Frentes 1 e 2]] · API: [[api-quiz-drf|🔌 API do quiz]] · Front: [[front-templates-django|🎨 Templates]] · Testes: [[testes-e-validacao-tcc|✅ Testes]]
> **Vem depois de:** [[passo-a-passo-f2-07|🎛️ F2-07 (orquestração)]] · **Desbloqueia:** a Frente 4 inteira

# 🔌 Passo a passo — F2-09

> [!abstract] O que esta nota é
> O **como** da tarefa F2-09: publicar o contrato **C3**, que é o único ponto por onde a Frente 4 enxerga a camada de IA. Mais o mínimo de fiação no site para o texto da LLM aparecer em `/resultado/<pk>/`.
> Escrito contra o repositório no commit `1b4f48e`, **assumindo a [[passo-a-passo-f2-07|F2-07]] já aplicada**. Sem o `deliver()`, `llm_text` é sempre vazio e este endpoint responde `pronto: false` para sempre.

> [!warning] Nada disto foi executado
> Código escrito contra a leitura do repositório, não rodado. A seção 5️⃣ é a prova.

> [!success] Por que esta vem antes da F2-08, e não depois
> A [[spec-motor-e-ia-frentes-1-2|spec]] põe a F2-09 na semana 4 e a F2-08 na 3 — mas põe também o contorno: *"puxe F1-07 e F2-09 para a semana 3 e empurre a recalibração; o que não pode acontecer é você parar esperando"*. A F2-08 é a **única** tarefa da frente que depende de uma credencial que ainda não existe. A F2-09 não toca a rede, e é ela que tira a Frente 4 da fila de espera. Fazer o bloqueado antes do que desbloqueia os outros é a ordem errada em qualquer semana.

---

## 🚦 Antes de abrir o editor

```bash
git checkout -b feat/ia-entrega-c3
```

> [!warning] Este é o PR que a Frente 4 lê
> Diferente da F2-07, **aqui você publica contrato**. Quando esse PR entrar, avise no grupo com o JSON de exemplo colado na mensagem: a partir dali, a F4 desenha a tela do Passo 10 sem precisar te perguntar nada — que é o critério de pronto da tarefa na spec.

---

## 1️⃣ O contrato C3, em código

No fim de `quiz/delivery.py`:

```python
def contrato_c3(attempt):
    """A entrega como a Frente 4 consome. Ver C3 na spec das frentes 1 e 2."""
    recs = list(
        attempt.recommendations.select_related("course__main_area").order_by("rank_final")
    )
    if not recs:
        return {
            "attempt_id": attempt.id,
            "pronto": False,
            "used_fallback": attempt.used_fallback,
            "diverged": attempt.diverged,
            "confianca": {"nivel": _nivel(attempt)},
            "principal": None,
            "alternativas": [],
        }

    principal, alternativas = recs[0], recs[1:]
    return {
        "attempt_id": attempt.id,
        "pronto": bool(principal.llm_text),
        "used_fallback": attempt.used_fallback,
        "diverged": attempt.diverged,
        "confianca": {"nivel": _nivel(attempt)},
        "principal": {
            "course_id": principal.course_id,
            "course_name": principal.course.name,
            "descricao": principal.course.description,
            "duration_hours": principal.course.duration_hours,
            "match": round(principal.score * 100),
            "texto_llm": principal.llm_text,
            "explanation": principal.explanation,
        },
        "alternativas": [
            {
                "course_id": rec.course_id,
                "course_name": rec.course.name,
                "match": round(rec.score * 100),
                "texto_llm": rec.llm_text,
                "explanation": rec.explanation,
            }
            for rec in alternativas
        ],
    }


def _nivel(attempt):
    return "empate_tecnico" if len(attempt.tie_set or []) > 1 else "ordem_do_motor"
```

> [!tip] `pronto` é o que permite o carregamento em duas etapas
> A F4 desenha **uma** tela e não precisa saber se a sua chamada foi síncrona. Enquanto `pronto` for `false`, ela mostra o resultado do motor com o `explanation`; quando virar `true`, troca o texto. Sem esse campo, a tela fica bloqueada esperando a camada de IA ficar pronta — e o paralelismo entre as frentes morre.

> [!note] `pronto: false` não é erro
> Ele acontece em três situações completamente diferentes e todas normais: a IA está desligada, o `deliver()` caiu em fallback, ou a chamada ainda não rodou. Quem distingue são `used_fallback` e `diverged`, que vêm no mesmo JSON. A tela não precisa distinguir — por isso são três campos e não um `status` com cinco valores que a F4 teria que decorar.

> [!warning] `descricao` e `duration_hours` só existem no principal
> É o C3 como está congelado na spec. Se a F4 pedir esses campos também nos cards das alternativas, **acrescente** — adicionar chave não quebra ninguém. O que não pode é remover ou renomear o que já está publicado. Se acrescentar, avise no mesmo canal em que publicou o contrato.

---

## 2️⃣ A view e a rota

### `quiz/views.py`

```python
from django.shortcuts import get_object_or_404

from quiz.delivery import contrato_c3


class AttemptDeliveryView(APIView):
    """GET /api/quiz/attempts/<pk>/entrega/ — contrato C3."""

    def get(self, request, pk):
        attempt = get_object_or_404(QuizAttempt, pk=pk)
        return Response(contrato_c3(attempt))
```

### `quiz/urls.py`

```python
from quiz.views import (
    AttemptDeliveryView,
    AttemptResultView,
    QuestionListView,
    SubmitQuizView,
)

urlpatterns = [
    path("questions/", QuestionListView.as_view(), name="quiz-questions"),
    path("submit/", SubmitQuizView.as_view(), name="quiz-submit"),
    path("attempts/<int:pk>/", AttemptResultView.as_view(), name="quiz-attempt"),
    path("attempts/<int:pk>/entrega/", AttemptDeliveryView.as_view(), name="quiz-entrega"),
]
```

> [!note] Endpoint novo em vez de campo novo no `/attempts/<pk>/`
> O `AttemptResultView` devolve uma lista plana de recomendações — é o formato que a API já publicou e que alguém pode estar consumindo. O C3 é uma **forma diferente** dos mesmos dados: principal separado das alternativas, com os metadados da entrega no topo. Enfiar isso no endpoint antigo seria quebrar um contrato para publicar outro. Duas rotas custam quatro linhas e nenhuma migração de cliente.

> [!warning] Este endpoint não tem autorização
> Qualquer pessoa que souber o `pk` lê a entrega de qualquer tentativa — igual ao `/attempts/<pk>/` de hoje. É consistente com o que existe, **não é aceitável em produção**, e já tem dono: a frente de autenticação ([[spec-autenticacao-lista-interesse|🔐 spec de autenticação]]) filtra por dono no queryset. Ao aplicar aquela spec, esta rota entra na mesma varredura — anote isso lá, não aqui, senão some.

---

## 3️⃣ O texto na tela do site

### `quiz/web_views.py` — dentro do `result_page`, no dicionário de `itens`

```python
    itens = [
        {
            "rank": rec.rank_final,
            "match": round(rec.score * 100),
            "course": rec.course,
            "areas": rec.explanation.get("top_areas", []),
            "texto_llm": rec.llm_text,
        }
        for rec in recomendacoes
    ]
```

### `templates/quiz/result.html` — linha 24

```html
        <p class="destaque__descricao">{{ destaque.texto_llm|default:destaque.course.description }}</p>
```

> [!success] `|default:` é a tela inteira do fallback, em uma linha
> Sem texto de LLM (desligada, fallback, ou ainda não rodou), a página mostra a descrição do curso — exatamente o que ela mostra hoje. Com texto, mostra o texto. **Nenhum `{% if %}`, nenhuma tela alternativa para manter.** É o mesmo princípio do `pronto` do C3, aplicado ao HTML.

> [!note] Aqui você para
> Card das alternativas, ordem visual, estados de carregamento — é tudo território da **F4**. Esta seção existe só para provar que o dado chega ao template, não para desenhar a tela. Se você continuar, vai estar refazendo o trabalho de outra pessoa com menos contexto que ela.

---

## 4️⃣ Os testes novos

Arquivo novo, `quiz/tests_entrega.py`:

```python
from io import StringIO

from django.core.management import call_command
from django.test import TestCase, override_settings
from django.urls import reverse

from quiz.delivery import deliver
from quiz.engine import recommend
from quiz.models import Answer, Choice, QuizAttempt

CHAVES_C3 = {
    "attempt_id", "pronto", "used_fallback", "diverged",
    "confianca", "principal", "alternativas",
}


class EntregaC3Test(TestCase):
    """O contrato que a Frente 4 consome. Se algum teste daqui falhar,
    alguem de outra frente descobre pela tela quebrada."""

    ELETRICO = ("Montar e ligar", "Chão de fábrica", "Eletricidade", "alicate")

    @classmethod
    def setUpTestData(cls):
        silencio = StringIO()
        for comando in ("seed_areas", "seed_courses", "seed_questions"):
            call_command(comando, stdout=silencio)

    def tentativa(self):
        attempt = QuizAttempt.objects.create(respondent_name="teste")
        for trecho in self.ELETRICO:
            choice = Choice.objects.get(text__icontains=trecho)
            Answer.objects.create(attempt=attempt, question=choice.question, choice=choice)
        recommend(attempt)
        attempt.refresh_from_db()
        return attempt

    def buscar(self, attempt):
        return self.client.get(reverse("quiz-entrega", args=[attempt.pk]))

    @override_settings(LLM_ENABLED=True, LLM_PROVIDER="fake")
    def test_contrato_c3_tem_todas_as_chaves(self):
        attempt = self.tentativa()
        deliver(attempt)
        corpo = self.buscar(attempt).json()

        self.assertEqual(set(corpo), CHAVES_C3)
        self.assertEqual(corpo["confianca"]["nivel"], "empate_tecnico")
        self.assertIn("texto_llm", corpo["principal"])
        self.assertEqual(len(corpo["alternativas"]), 4)

    @override_settings(LLM_ENABLED=False)
    def test_sem_llm_o_endpoint_responde_200_com_pronto_false(self):
        """Botao de panico: a F4 continua tendo tela, so nao tem texto."""
        attempt = self.tentativa()
        deliver(attempt)
        corpo = self.buscar(attempt).json()

        self.assertFalse(corpo["pronto"])
        self.assertFalse(corpo["used_fallback"])
        self.assertEqual(corpo["principal"]["texto_llm"], "")
        self.assertTrue(corpo["principal"]["explanation"]["top_areas"])

    @override_settings(LLM_ENABLED=True, LLM_PROVIDER="fake")
    def test_ordem_publicada_e_a_rank_final(self):
        attempt = self.tentativa()
        deliver(attempt)
        corpo = self.buscar(attempt).json()

        primeiro = attempt.recommendations.get(rank_final=1)
        self.assertEqual(corpo["principal"]["course_id"], primeiro.course_id)
        self.assertTrue(corpo["pronto"])

    def test_tentativa_inexistente_da_404(self):
        self.assertEqual(self.client.get(reverse("quiz-entrega", args=[99999])).status_code, 404)

    @override_settings(LLM_ENABLED=True, LLM_PROVIDER="fake")
    def test_pagina_de_resultado_mostra_o_texto_da_llm(self):
        attempt = self.tentativa()
        deliver(attempt)
        texto = attempt.recommendations.get(rank_final=1).llm_text

        resposta = self.client.get(reverse("quiz-resultado", args=[attempt.pk]))
        self.assertContains(resposta, texto[:40])
```

> [!tip] `set(corpo) == CHAVES_C3`, e não uma sequência de `assertIn`
> Igualdade de conjunto pega as duas direções: chave que **sumiu** e chave que **apareceu sem aviso**. Como este é o contrato de outra frente, a segunda importa tanto quanto a primeira — campo que entra sem anúncio é campo que a F4 não sabe que pode usar, ou que ela passa a usar antes de você garantir que ele sempre existe.

> [!note] Por que arquivo separado
> `tests_llm.py` já cobre provider, prompt e validação; `tests.py` cobre o motor. O C3 é contrato **publicado**, e ter um arquivo com esse nome é o que faz alguém pensar duas vezes antes de mudar a forma do JSON. É documentação por organização de pasta.

---

## 5️⃣ Verificação

```bash
PYTHONIOENCODING=utf-8 ./.venv/Scripts/python.exe manage.py test
```

**50 testes verdes** — os 45 que a [[passo-a-passo-f2-07|F2-07]] deixou mais os 5 novos.

E o contrato visto com os próprios olhos, que é o que você vai colar no grupo:

```bash
./.venv/Scripts/python.exe manage.py runserver 8010
```

Responda o quiz, anote o `pk` do resultado e abra `http://localhost:8010/api/quiz/attempts/<pk>/entrega/`. O DRF renderiza a API navegável — **é essa tela que você manda para a F4**, não um JSON copiado à mão que pode estar desatualizado.

> [!important] Rode uma vez com a IA desligada
> `LLM_ENABLED=false` no `.env`, refaça o quiz e abra o mesmo endpoint. Tem que responder **200 com `pronto: false`**, nunca 500 e nunca 404. Esse é o comportamento que sustenta a promessa de rodar offline no dia da defesa — e é a única parte do C3 que a F4 não tem como testar sozinha.

---

## 6️⃣ Commit

```bash
git add -A && git commit -m "feat(ia): endpoint de entrega no contrato C3 e texto da llm no resultado"
```

E, no mesmo dia, a mensagem no grupo com o JSON de exemplo. Contrato publicado que ninguém sabe que existe não desbloqueia ninguém.

## 📎 Veja também

- [[passo-a-passo-f2-07|🎛️ F2-07 — orquestração da entrega]] — o pré-requisito
- [[passo-a-passo-f2-08|🌐 F2-08 — GeminiProvider, cache e timeout]] — o próximo, quando a credencial sair
- [[spec-motor-e-ia-frentes-1-2|🧭 Spec das Frentes 1 e 2]] · [[api-quiz-drf|🔌 API do quiz]] · [[front-templates-django|🎨 Templates Django]]
- [[spec-autenticacao-lista-interesse|🔐 Spec de autenticação]] — onde esta rota ganha dono
- [[divisao-de-trabalho-tcc|👥 Divisão de trabalho]] · [[testes-e-validacao-tcc|✅ Testes e validação]]
- [[TCC|🎓 TCC]] · [[Projetos|🚀 Projetos]]
