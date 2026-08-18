---
title: "API REST do quiz (DRF)"
aliases: ["API do TCC", "Endpoints do quiz"]
tags: [tcc, drf, api, rest, django, serializers]
status: concluido
projeto: TCC
criado: 2026-08-17
---

> [!info] Projeto: [[TCC|🎓 TCC]] · Conceitos: [[Django REST Framework|DRF]] · [[REST API|REST API]] · [[Serializers|Serializers]] · [[Views|Views]] · [[HTTP|HTTP]]

# 🔌 API REST do quiz

> [!abstract] O que é
> Três endpoints em `/api/quiz/` que expõem o quiz e o resultado em JSON. O site em templates ([[front-templates-django|🎨 front]]) não depende deles, mas **ambos chamam a mesma `recommend()`** — a regra de negócio vive num lugar só.

## 🛣️ Endpoints

| Método | Rota | View | O quê |
|---|---|---|---|
| `GET` | `/api/quiz/questions/` | `QuestionListView` | Perguntas ativas com alternativas aninhadas |
| `POST` | `/api/quiz/submit/` | `SubmitQuizView` | Envia respostas → cria tentativa → devolve ranking (201) |
| `GET` | `/api/quiz/attempts/<pk>/` | `AttemptResultView` | Recupera o resultado de uma tentativa |

Registradas em `quiz/urls.py`, incluídas com `path("api/quiz/", include("quiz.urls"))`.

## 📥 Contrato do `POST /submit/`

```json
{
  "respondent_name": "Luis",
  "answers": [
    {"question": 1, "choice": 3},
    {"question": 2, "choice": 7}
  ]
}
```

Resposta `201`:

```json
{
  "attempt_id": 12,
  "recommendations": [
    {
      "rank": 1,
      "score": 0.968,
      "course_name": "Injeção Eletrônica Automotiva",
      "course_description": "Sensores, atuadores e diagnóstico com scanner automotivo.",
      "duration_hours": 120,
      "main_area": "Mecânica Automotiva",
      "explanation": {"top_areas": [{"area": "mecanica-automotiva", "area_name": "Mecânica Automotiva", "contribuicao": 45.0, "percentual": 52.3}]}
    }
  ]
}
```

## 🧱 Serializers

| Serializer | Papel |
|---|---|
| `ChoiceSerializer` | `id` + `text` — nunca expõe os pesos |
| `QuestionSerializer` | Pergunta com `choices` aninhadas (`many=True, read_only=True`) |
| `AnswerInputSerializer` | Valida **um** par pergunta/alternativa |
| `SubmitQuizSerializer` | O envelope: `respondent_name` + `answers` |
| `RecommendationSerializer` | Achata o curso na resposta com `source="course.name"` etc. |

> [!important] Os pesos nunca saem na API
> `ChoiceSerializer` expõe só `id` e `text`. Se os pesos vazassem, dava para engenharia reversa do resultado — e um respondente mal-intencionado poderia "escolher" o curso que quisesse. O cálculo fica inteiramente no servidor.

## 🛡️ As duas validações que importam

### 1. Alternativa que não pertence à pergunta
```python
if attrs["choice"].question_id != attrs["question"].id:
    raise serializers.ValidationError(...)
```
Sem isso, um cliente manda `{"question": 1, "choice": 15}` com a alternativa de outra pergunta e o perfil sai contaminado — sem erro nenhum, porque as duas FKs existem.

### 2. Duas respostas para a mesma pergunta
`validate_answers` compara `len(ids) != len(set(ids))`. Sem isso, o `bulk_create` só estouraria lá na frente com `IntegrityError` do `unique_together`, virando erro 500 em vez de 400.

> [!warning] A validação silenciosa que quase passou batido
> O campo foi escrito como `answer` (singular) em vez de `answers`. Efeito em cascata: o DRF só chama `validate_<campo>` se o campo existir **com o nome exato**, então o `validate_answers` nunca rodava — validação escrita e **morta em silêncio**, além do 400 permanente. Ver [[bugs-e-licoes-tcc|🐛 Bugs e lições]].

## ⚛️ Atomicidade na view

```python
with transaction.atomic():
    attempt = QuizAttempt.objects.create(...)
    Answer.objects.bulk_create([...])
    recomendacoes = recommend(attempt)
```

Sem a transação, se `recommend()` estourar no meio, sobram `QuizAttempt` e `Answer` órfãos — uma tentativa sem resultado, que o `AttemptResultView` depois devolveria vazia.

## 🧪 Como testar a API manualmente

```python
from django.test import Client
from django.db import transaction

c = Client(SERVER_NAME="localhost")      # senão: DisallowedHost 'testserver'
sp = transaction.savepoint()
resp = c.post("/api/quiz/submit/", data={...}, content_type="application/json")
transaction.savepoint_rollback(sp)       # não suja o banco
```

> [!tip] Pegadinha do bash no Windows
> `!=` dentro de `python -c "..."` vira `\!=` e dá `SyntaxError`. Reescrever com `==` ou pôr o script num arquivo.

## Veja também

- [[TCC|🎓 TCC]]
- [[front-templates-django|🎨 Front em templates Django]]
- [[engine-matching-cosseno|🧮 Engine de matching]]
