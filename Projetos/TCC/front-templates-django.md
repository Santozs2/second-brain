---
title: "Front do quiz — templates Django, wizard e página de resultado"
aliases: ["Front do TCC", "Templates do quiz", "UI do TCC"]
tags: [tcc, django, templates, css, javascript, ux]
status: concluido
projeto: TCC
criado: 2026-08-17
---

> [!info] Projeto: [[TCC|🎓 TCC]] · Conceitos: [[HTML|HTML]] · [[CSS|CSS]] · [[JavaScript|JavaScript]] · [[Views|Views]] · [[MVC|MVC]]

# 🎨 Front em templates Django

> [!abstract] O que é
> O site que a pessoa usa: responde o quiz numa pergunta por vez e cai numa página de resultado com o curso destaque e a justificativa. Templates Django + [[CSS|CSS]] puro + [[JavaScript|JavaScript]] vanilla — **sem build, sem `node_modules`, sem CORS**.

## 🗂️ Arquivos

```
quiz/web_views.py          quiz_page (GET+POST) e result_page
quiz/web_urls.py           "/" e "/resultado/<pk>/"
templates/base.html        layout + carregamento de estáticos
templates/quiz/quiz.html   formulário do quiz
templates/quiz/result.html ranking
static/css/style.css       ~700 linhas
static/js/quiz.js          o wizard (~90 linhas)
```

`config/settings.py` ganhou `TEMPLATES["DIRS"] = [BASE_DIR / "templates"]` e `STATICFILES_DIRS = [BASE_DIR / "static"]`.

## 🤔 Por que templates e não React

| Critério | Templates (escolhido) | React |
|---|---|---|
| Runtime | Um só, Django | Dois processos, dois deploys |
| Setup na banca | `runserver` e pronto | build + servir estáticos |
| CORS/JWT | Não existem no problema | Precisa configurar |
| Foco do TCC | Sobra tempo para a engine | Gasta tempo em plumbing |

> [!note] A API não foi jogada fora
> `/api/quiz/` continua existindo e funcionando. O trabalho fica com **duas portas de entrada** (HTML e JSON) chamando a mesma `recommend()` — o que, na monografia, é o argumento de que a regra de negócio está isolada da camada de apresentação.

## 🧭 UX do quiz

- **Wizard**: uma pergunta por vez, com barra de progresso.
- **Auto-avanço** 320 ms depois de marcar — tempo suficiente para o olho registrar a seleção.
- **Label inteiro clicável**, não só o radiozinho.
- **Botão Voltar** para revisar sem perder o que já foi marcado.
- **Resultado**: card de destaque com anel `conic-gradient` mostrando o % de afinidade, barras do `explanation` sob o título *"por que este curso"*, e 4 cards menores abaixo.

> [!important] Progressive enhancement — o detalhe elegante
> O CSS **só** esconde os passos quando o JavaScript adiciona a classe `.quiz--wizard` ao formulário. Sem JS (ou se o script falhar), a página vira um formulário simples empilhado e o POST funciona igual.
> O wizard é um **aprimoramento**, não um requisito. Nunca existe a tela em branco que trava o usuário.

## 🛡️ Validação no servidor — não confiar no front

O JS impede avançar sem responder, mas isso é conveniência de UI. Quem manda `POST` direto ignora tudo. Por isso `quiz_page` revalida:

### Alternativa que não pertence à pergunta
```python
escolha = next((c for c in question.choices.all() if str(c.id) == enviado), None)
```
Itera as alternativas **daquela** pergunta (já no prefetch, sem query extra) e compara com `str(c.id)` — porque o que vem do POST é string. ID trocado no HTML pelo DevTools simplesmente não casa e é descartado.

### Quiz incompleto → 400 que preserva o preenchido
```python
question.marcada_id = getattr(selecionadas.get(question), "id", None)
```

> [!tip] Por que um atributo na instância
> O template Django **não faz lookup de dicionário por chave dinâmica** (`{{ marcadas[question.id] }}` não existe). A saída idiomática é pendurar o valor na própria instância dentro da view e ler `{{ question.marcada_id }}` no template.

## 🔀 Fluxo das requisições

```
GET  /                      → renderiza o quiz
POST /                      → valida
                              ├── incompleto/inválido → 400 + formulário preenchido
                              └── ok → transaction.atomic:
                                        QuizAttempt + Answer(bulk) + recommend()
                                        → 302 para /resultado/<pk>/
GET  /resultado/<pk>/       → destaque + 4 cards (404 se não existir)
```

> [!note] Redirect depois do POST (PRG)
> O POST não renderiza o resultado direto: ele redireciona. Assim o F5 na página de resultado não reenvia o formulário nem cria tentativa duplicada — e a URL do resultado vira um link compartilhável.

## 🎨 Preparo dos dados na view, não no template

`result_page` monta uma lista de dicionários já com `match` (score × 100 arredondado), `course` e `areas`, e separa `destaque` (o primeiro) de `outros` (o resto). Template Django não faz aritmética; a conta fica na view, onde é testável.

## ✅ Smoke test do site

Feito com `Client(SERVER_NAME="localhost")` dentro de savepoint com rollback:

| Cenário | Esperado | Resultado |
|---|---|---|
| `GET /` | 200 com 6 perguntas | ✅ |
| POST completo | 302 | ✅ |
| `GET /resultado/<pk>/` | 200 com 4 cards | ✅ |
| POST incompleto | 400 | ✅ |
| POST com alternativa cruzada | 400 | ✅ |
| Resultado inexistente | 404 | ✅ |

## Veja também

- [[TCC|🎓 TCC]]
- [[api-quiz-drf|🔌 API REST do quiz]]
- [[testes-e-validacao-tcc|✅ Testes e validação]]
