---
title: "Bugs e lições aprendidas — TCC"
aliases: ["Bugs do TCC", "Lições do TCC", "Armadilhas do TCC"]
tags: [tcc, bugs, licoes, debugging, django]
status: continuo
projeto: TCC
criado: 2026-08-17
---

> [!info] Projeto: [[TCC|🎓 TCC]] · Ver também: [[testes-e-validacao-tcc|✅ Testes]]

# 🐛 Bugs e lições aprendidas

> [!abstract] Por que registrar isso
> Cada bug abaixo custou tempo e ensinou algo reaproveitável. Vários viraram teste automatizado — a lição fica cravada no código, não só na memória.

---

## 🗃️ Passo 2 — Modelagem

> [!bug] `unique_together` fora do `class Meta`
> Escrito solto no corpo da classe. O Django **ignora em silêncio**: sem erro, sem constraint, sem proteção. Só aparece quando dados duplicados começam a entrar.
> **Lição:** opção de `Meta` fora do `Meta` é atributo comum de classe. Nenhum framework avisa.

> [!bug] Model declarado sem nenhum campo
> A `Answer` nasceu só com a linha do `unique_together` — sem `attempt`, `question` nem `choice`.
> **Lição:** ao escrever vários models seguidos, reler cada um perguntando "esta tabela guarda o quê?".

> [!bug] O campo que É a feature ficou de fora
> `CourseAreaWeight` sem o campo `weight` — a tabela de pesos sem peso.
> **Lição:** o campo mais importante é o mais fácil de esquecer, porque o nome da tabela já "diz" que ele existe.

> [!bug] Typos silenciosos: `crated_at`, `Recomendation`
> **Lição:** typo em nome de campo vira coluna com nome errado no banco. Custa migration para corrigir depois.

> [!bug] Idioma misturado entre apps
> `catalog` em português e `quiz` em inglês, com referência quebrada `catalog.Curso`.
> **Lição:** padronizar idioma **antes** da primeira migration definitiva. Depois, custa apagar e recriar migrations.

---

## 🛠️ Passo 3 — Admin

> [!bug] `setattr = [...]` no lugar de `search_fields`
> `setattr` é builtin: atribuir a ele **não dá erro**, só não faz nada. O autocomplete continuava quebrado sem explicação.
> **Lição:** quando uma opção do admin "não faz efeito", conferir se o nome do atributo está certo antes de investigar o Django.

> [!bug] `CouseAreaWeightInline` / `ChoiceAreaWeigthInline`
> Typos em nome de classe.

> [!info] Não é bug, é limitação
> **O Django não faz inline aninhado.** Não dá para editar o peso dentro da alternativa que já está dentro da pergunta. Solução: `ChoiceAdmin` próprio.
> E `autocomplete_fields` exige `search_fields` no admin **de destino**, não no de origem.

---

## 🧮 Passo 4 — Engine

> [!bug] `rank_courses` sem `return`
> `lista.sort()` ordena **in-place** e devolve `None`. A função terminava sem retornar nada e o erro estourava longe da causa:
> ```
> TypeError: 'NoneType' object is not subscriptable
>   ranking = rank_courses(profile, courses)[:limit]
> ```
> **Lição:** `.sort()` muta e devolve `None`; `sorted()` devolve lista nova. Erro de `NoneType` "sem sentido" quase sempre é função sem `return` alguns quadros acima.

> [!bug] Meu, no `test_engine`: trechos ambíguos
> `"computador"` e `"multímetro"` casavam com alternativas de **duas** perguntas diferentes → duas respostas para a mesma pergunta → `IntegrityError` no `unique_together`.
> **Lição:** buscar por `icontains` em teste é frágil. Corrigido com textos únicos e, na suíte definitiva, trocando `filter().first()` por `get()`, que **falha alto** no ambíguo.

---

## 🔌 Passo 5 — API

> [!bug] Campo `answer` (singular) em vez de `answers` — bug em cascata
> Sintoma: `POST /submit/` sempre 400, "This field is required".
> O efeito escondido é pior: **o DRF só chama `validate_<campo>` se o campo existir com o nome exato**. Como o campo se chamava `answer`, o método `validate_answers` (que barra pergunta duplicada) **nunca rodava** — validação escrita e morta em silêncio.
> **Lição:** um erro de nome no serializer pode desligar validações inteiras sem nenhum aviso. Ao ver 400 "field is required", conferir o nome antes do payload.

> [!bug] `transaction` importado e não usado
> Faltava o `with transaction.atomic():` no `SubmitQuizView.post`. Sem ele, `QuizAttempt` e `Answer` ficariam órfãos se `recommend()` estourasse.
> **Lição:** import não utilizado costuma ser rastro de intenção esquecida.

---

## 🎨 Passo 6 — Front

> [!info] Template Django não faz lookup por chave dinâmica
> Reexibir o formulário com o que já foi marcado exigiria `marcadas[question.id]` no template — que não existe. Solução idiomática: pendurar `question.marcada_id` na instância dentro da view.

> [!important] Nunca confiar na validação do front
> O JS impede avançar sem responder, mas `curl` ignora o JS. Alternativa cruzada e quiz incompleto são revalidados no servidor.

---

## 💻 Ambiente (Windows)

> [!warning] `UnicodeEncodeError` (cp1252)
> Qualquer comando que imprima acento ou emoji quebra. Prefixar sempre: `PYTHONIOENCODING=utf-8 ./.venv/Scripts/python.exe ...`

> [!warning] `!=` no bash do Windows
> Dentro de `python -c "..."`, o `!=` vira `\!=` e dá `SyntaxError`. Reescrever com `==` ou pôr o script num arquivo.

> [!warning] `DisallowedHost: 'testserver'`
> Testes com `django.test.Client` precisam de `Client(SERVER_NAME="localhost")`.

> [!tip] Não sujar o banco de desenvolvimento
> Scripts exploratórios dentro de `transaction.savepoint()` + `savepoint_rollback()`; comandos de demonstração com `transaction.set_rollback(True)`.

---

## 🧵 Padrões recorrentes

| Padrão | Como pegar cedo |
|---|---|
| Opção de `Meta` fora do `Meta` | Reler o `Meta` de cada model após escrever |
| Função de ordenação sem `return` | Erro de `NoneType` distante da causa |
| Nome de campo/atributo errado | Sintoma "não faz nada" ou 400 genérico |
| Colar bloco novo sem apagar o antigo | Conferir duplicação depois de cada spec aplicada |
| Esquecer o campo central da feature | Perguntar "esta tabela guarda o quê?" |

## Veja também

- [[TCC|🎓 TCC]]
- [[testes-e-validacao-tcc|✅ Testes e validação]]
- [[guia-tcc-quiz-perfil|🏗️ Guia de implementação]]
