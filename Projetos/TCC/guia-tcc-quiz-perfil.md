---
title: "Guia de Implementação — TCC: Quiz de Perfil e Recomendação de Cursos"
aliases: ["Guia do TCC", "Passos do TCC", "Roteiro TCC quiz"]
tags: [django, tcc, quiz, guia, recomendacao]
status: concluido
projeto: TCC
criado: 2026-08-17
---

> [!info] Projeto: [[TCC|🎓 TCC]] · Tecnologias: [[Django|Django]] · [[Django REST Framework|DRF]] · [[Python|Python]] · [[CSS|CSS]]

# 🏗️ Guia de implementação — do setup ao site no ar

> [!abstract] Objetivo
> Construir, em passos que sempre terminam com algo rodável, um sistema que aplica um quiz de perfil e recomenda cursos com justificativa. **Ao final, o fluxo fecha fim a fim:** cadastro no admin → quiz → engine → ranking explicado.

> [!tip] Regra do guia
> Cada passo depende do anterior e termina testável. Nada de "escrevo tudo e testo no final" — foi assim que os bugs foram pegos cedo (ver [[bugs-e-licoes-tcc|🐛 Bugs e lições]]).

---

## 🗺️ Visão geral dos passos

| Passo | Foco | Entrega | Commit |
|---|---|---|---|
| 1 | Setup do projeto Django | Projeto de pé | `dbfc5fa` |
| 2 | Modelagem de dados | Models + migrations | `9df637b` |
| 3 | Admin + seeds | Base populada | `a972cc7` |
| 4 | Engine de matching | Ranking calculado | `a75207d` |
| 4.1 | Testes automatizados | `manage.py test` verde | `e5bdd92` |
| 5 | API REST (DRF) | Endpoints JSON | `bcf5ccf` + `b3f3508` |
| 6 | Front em templates | Site navegável | `d3f326d` |

**Ordem não é acidental:** a engine vem **antes** de qualquer tela. Sem ranking calculado, front é decoração sem conteúdo.

---

## ⚙️ Passo 1 — Setup

### Objetivo
Projeto Django rodando, com o venv isolado e as dependências pinadas.

### Tarefas
- [x] `django-admin startproject config .` — o projeto chama-se **`config`**, não `core` ✅
- [x] Apps `catalog` e `quiz` criados e registrados no `INSTALLED_APPS` ✅
- [x] `venv` em `.venv` + `requirements.txt` pinado ✅
- [x] `.gitignore` com `db.sqlite3`, `__pycache__`, `.venv` ✅

### Decisões
- **SQLite, sem multi-tenant, sem JWT.** O valor do trabalho está na engine; infraestrutura complexa só rouba tempo da parte que a banca vai avaliar.
- **Dois apps, não um.** `catalog` é o domínio (o que a instituição oferece) e `quiz` é o instrumento de medida (como se descobre o perfil). O scraping futuro alimenta **só** o `catalog`, sem tocar no quiz.

> [!warning] Ambiente Windows
> Rodar sempre pelo Python do venv: `./.venv/Scripts/python.exe`. E prefixar com `PYTHONIOENCODING=utf-8`, senão acento e emoji quebram com `UnicodeEncodeError` (cp1252).

---

## 🗃️ Passo 2 — Modelagem

### Objetivo
Ter as tabelas que sustentam o cálculo — e o campo de peso, que é o núcleo de tudo.

### Tarefas
- [x] `catalog`: `Area`, `Course`, `CourseAreaWeight` ✅
- [x] `quiz`: `Question`, `Choice`, `ChoiceAreaWeight`, `QuizAttempt`, `Answer`, `Recommendation` ✅
- [x] `makemigrations` + `migrate` ✅

### Detalhes em [[modelagem-dados-quiz|🗃️ Modelagem de dados]]

> [!warning] Armadilhas deste passo
> - `unique_together` **dentro** do `class Meta`. Solto no corpo da classe, o Django ignora em silêncio.
> - `Area.slug` é a chave estável usada pela engine — renomear a área não pode quebrar o cálculo.
> - Padronizar o idioma do código (inglês) **antes** de criar migrations. Misturar `Curso` com `Question` gera referências quebradas tipo `catalog.Curso`.

### ✅ Definition of Done
- `manage.py migrate` roda limpo e as 9 tabelas existem.

---

## 🛠️ Passo 3 — Admin + seeds

### Objetivo
Conseguir cadastrar curso e peso em uma tela só, e ter um comando que popula a base do zero.

### Tarefas
- [x] `catalog/admin.py`: `AreaAdmin` (com `prepopulated_fields` do slug) e `CourseAdmin` com inline de `CourseAreaWeight` ✅
- [x] `quiz/admin.py`: `QuestionAdmin` com inline de `Choice`; `ChoiceAdmin` próprio com inline de `ChoiceAreaWeight` ✅
- [x] `seed_areas`, `seed_courses`, `seed_questions` ✅

> [!warning] O Django não faz inline aninhado
> Não dá para colocar o peso dentro da alternativa que já está dentro da pergunta. Por isso `Choice` ganha um `ModelAdmin` próprio — é lá que os pesos são editados.

> [!note] Padrão dos seeds
> `update_or_create` por chave natural (slug/name/text) + `exclude(...).delete()` para limpar órfãos, tudo em `@transaction.atomic`. Assim o arquivo do seed é a **fonte da verdade**: tirou um curso de lá, ele some do banco. E o comando é idempotente — roda 10 vezes, não duplica.

### ✅ Definition of Done
- `seed_areas && seed_courses && seed_questions` popula 7 áreas, 12 cursos e 6 perguntas em qualquer banco vazio.

---

## 🧮 Passo 4 — Engine de matching

### Objetivo
Transformar respostas em ranking, com nota e justificativa, sem `if` encadeado.

### Tarefas
- [x] `course_vector` / `profile_vector` — os dois lados viram `{área: número}` ✅
- [x] `dot` / `norm` / `cosine_similarity` — a matemática, em funções puras ✅
- [x] `explain` — top 3 áreas com contribuição e percentual ✅
- [x] `rank_courses` — ordena com desempate reprodutível ✅
- [x] `recommend(attempt, limit=5)` — única função que fala com o banco ✅
- [x] `manage.py test_engine` — 4 perfis simulados com rollback ✅

### Detalhes em [[engine-matching-cosseno|🧮 Engine de matching]]

### ✅ Definition of Done
Os quatro critérios de aceite passaram:
1. Perfil elétrico → Eletricista (0.995) e Comandos Elétricos (0.985) no topo.
2. Perfil automotivo+elétrico → **Injeção Eletrônica (0.968) acima de Motores a Combustão (0.891)**.
3. Perfil costura → 0.999 / 0.969, com mecânica caindo para 0.13.
4. Perfil TI → Python (1.000) / IA (0.979), e Automação Industrial no meio (0.471) por pesar TI 3 / IA 2.

> [!success] O critério 2 é o que prova a engine
> Ele só passa se o sistema entender **combinação** de áreas. Se inverter, o problema está nos pesos do seed, não na matemática.

---

## ✅ Passo 4.1 — Testes automatizados

### Objetivo
Trocar a conferência visual por `manage.py test`.

### Tarefas
- [x] `MatematicaTest(SimpleTestCase)` — 4 testes das funções puras, sem banco ✅
- [x] `RecomendacaoTest(TestCase)` — 8 testes com os seeds como fixture ✅

### Detalhes em [[testes-e-validacao-tcc|✅ Testes e validação]]

> [!tip] Os seeds viram fixture
> `setUpTestData` chama `call_command("seed_areas")` etc. Além de preparar os dados, isso **testa os próprios seeds** de brinde: se um quebrar, todos os testes de recomendação caem junto.

### ✅ Definition of Done
- `PYTHONIOENCODING=utf-8 ./.venv/Scripts/python.exe manage.py test` → **12 testes, OK**.

---

## 🔌 Passo 5 — API REST

### Objetivo
Expor o quiz e o resultado em JSON, para qualquer front consumir.

### Tarefas
- [x] `QuestionSerializer` (com alternativas aninhadas) e `RecommendationSerializer` ✅
- [x] `SubmitQuizSerializer` com validação de alternativa cruzada e pergunta duplicada ✅
- [x] `QuestionListView`, `SubmitQuizView`, `AttemptResultView` ✅
- [x] `explanation` passou a devolver `area_name` junto do slug ✅

### Detalhes em [[api-quiz-drf|🔌 API REST do quiz]]

### ✅ Definition of Done
- `GET /api/quiz/questions/` lista 6 perguntas com alternativas.
- `POST /api/quiz/submit/` devolve 201 com o ranking.
- `GET /api/quiz/attempts/<id>/` recupera o resultado depois.

---

## 🎨 Passo 6 — Front em templates Django

### Objetivo
Uma pessoa comum abrir o site, responder e ver o resultado — sem Postman.

### Tarefas
- [x] `quiz/web_views.py` (`quiz_page`, `result_page`) + `quiz/web_urls.py` ✅
- [x] `templates/base.html`, `quiz/quiz.html`, `quiz/result.html` ✅
- [x] `static/css/style.css` + `static/js/quiz.js` (wizard) ✅
- [x] Validação server-side de alternativa cruzada e quiz incompleto ✅

### Detalhes em [[front-templates-django|🎨 Front em templates Django]]

> [!note] Templates, não React
> Decisão consciente: um trabalho acadêmico ganha mais em ter tudo em um runtime só — sem build, sem `node_modules`, sem CORS. A API DRF continua existindo em `/api/quiz/` e **as duas pontas chamam a mesma `recommend()`**.

### ✅ Definition of Done
- `/` responde 200 com as 6 perguntas, POST completo redireciona (302) e `/resultado/<pk>/` mostra os 5 cursos.

---

## 🚀 Como rodar o projeto do zero

```bash
cd "C:/Users/USER/Documents/vs code/tcc-treine"
./.venv/Scripts/python.exe -m pip install -r requirements.txt
PYTHONIOENCODING=utf-8 ./.venv/Scripts/python.exe manage.py migrate
PYTHONIOENCODING=utf-8 ./.venv/Scripts/python.exe manage.py seed_areas
PYTHONIOENCODING=utf-8 ./.venv/Scripts/python.exe manage.py seed_courses
PYTHONIOENCODING=utf-8 ./.venv/Scripts/python.exe manage.py seed_questions
PYTHONIOENCODING=utf-8 ./.venv/Scripts/python.exe manage.py createsuperuser
PYTHONIOENCODING=utf-8 ./.venv/Scripts/python.exe manage.py runserver 8010
```

| URL | O quê |
|---|---|
| `/` | Quiz |
| `/resultado/<pk>/` | Resultado da tentativa |
| `/admin/` | Cadastro de áreas, cursos, pesos e perguntas |
| `/api/quiz/questions/` | API — perguntas |
| `/api/quiz/submit/` | API — enviar respostas |

> [!tip] Porta 8010
> A 8000 costuma estar ocupada pelo [[ChatBot|💬 ChatBot]].

## Veja também

- [[TCC|🎓 TCC]]
- [[Projetos|🚀 Projetos]]
