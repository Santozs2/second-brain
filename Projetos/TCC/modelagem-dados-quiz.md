---
title: "Modelagem de dados — quiz de perfil e catálogo de cursos"
aliases: ["Modelagem TCC", "Models do TCC", "Modelo de dados do quiz"]
tags: [django, tcc, models, modelagem, banco-de-dados]
status: concluido
projeto: TCC
criado: 2026-08-17
---

> [!info] Projeto: [[TCC|🎓 TCC]] · Conceitos: [[Models|Models]] · [[Migrations|Migrations]] · [[ORM|ORM]] · [[Banco de Dados|Banco de Dados]]

# 🗃️ Modelagem de dados

> [!abstract] Ideia central
> Perfil e curso precisam ser **o mesmo tipo de objeto** para poderem ser comparados: um vetor de pesos por área. Toda a modelagem existe para produzir esses dois vetores.

## 🧩 Diagrama

```
        catalog                                quiz
    ┌─────────────┐                     ┌──────────────┐
    │    Area     │◄────────┐  ┌───────►│   Question   │
    │  slug (PK   │         │  │        │  text, order │
    │  natural)   │         │  │        └──────┬───────┘
    └──────┬──────┘         │  │               │ 1:N
           │ 1:N            │  │               ▼
           ▼                │  │        ┌──────────────┐
    ┌─────────────┐         │  │        │    Choice    │
    │   Course    │         │  │        │  text, order │
    │ main_area   │         │  │        └──────┬───────┘
    └──────┬──────┘         │  │               │ 1:N
           │ 1:N            │  │               ▼
           ▼                │  │        ┌──────────────────┐
    ┌──────────────────┐    │  └────────┤ ChoiceAreaWeight │
    │ CourseAreaWeight ├────┘           │  weight (0..5)   │
    │   weight (0..5)  │                └──────────────────┘
    └──────────────────┘

    ┌──────────────┐  1:N   ┌──────────┐        ┌──────────────────┐
    │ QuizAttempt  ├───────►│  Answer  │        │  Recommendation  │
    │ created_at   │        │ question │        │ rank, score      │
    │ respondent   ├───────────────────────────►│ explanation JSON │
    └──────────────┘  1:N   │ choice   │        └──────────────────┘
                            └──────────┘
```

## 📦 App `catalog` — o domínio

### `Area`
A dimensão do espaço vetorial. **7 áreas** = 7 eixos.

| Campo | Tipo | Papel |
|---|---|---|
| `slug` | `SlugField(unique=True)` | **Chave estável** usada pela engine e pelo `explanation` |
| `name` | `CharField(100)` | Rótulo legível, pode mudar sem quebrar nada |
| `description` | `TextField(blank=True)` | Texto de apoio |

> [!important] Por que o slug existe
> A engine indexa tudo por slug. Se ela dependesse do `name`, renomear "TI" para "Tecnologia da Informação" invalidaria os dados históricos. O slug é o contrato; o nome é apresentação.

### `Course`
`name`, `description`, `main_area` (FK `PROTECT`), `duration_hours`, `start_date`/`end_date` (opcionais), `is_active`.

- **`PROTECT` no `main_area`**: apagar uma área com cursos vinculados tem que falhar, não apagar os cursos junto.
- **`is_active`**: a engine filtra por ele. Curso descontinuado sai da recomendação sem perder o histórico das tentativas antigas.

### `CourseAreaWeight`
O vetor do curso. `course` + `area` + `weight` (`FloatField` 0–5 validado), com `unique_together = ("course", "area")`.

## 📋 App `quiz` — o instrumento de medida

### `Question` e `Choice`
`text`, `order`, `is_active` na pergunta; `text` e `order` na alternativa, com FK para a pergunta. O `ordering = ["order"]` no `Meta` garante que o quiz sempre apareça na mesma sequência.

### `ChoiceAreaWeight`
O vetor da alternativa: escolher aquela opção soma `weight` naquela área. `unique_together = ("choice", "area")`.

### `QuizAttempt`, `Answer`, `Recommendation`

| Model | Papel | Restrição importante |
|---|---|---|
| `QuizAttempt` | Uma sessão de resposta | `ordering = ["-created_at"]` |
| `Answer` | Resposta a uma pergunta | `unique_together = ("attempt", "question")` — uma resposta por pergunta |
| `Recommendation` | Resultado calculado | `unique_together = ("attempt", "course")`, `ordering = ["rank"]` |

O `Recommendation.explanation` é um `JSONField` com as áreas que mais contribuíram — ver [[engine-matching-cosseno|🧮 Engine]].

> [!todo] Extensão prevista para a camada de IA (Passo 7)
> Com o [[decisao-camada-ia|Desenho B modificado]] aprovado, `Recommendation` ganha `rank_engine` (o `rank` atual), `rank_final` (a ordem entregue pela LLM), `llm_text` e `is_primary`; e `QuizAttempt` ganha os metadados da chamada (`llm_model`, `prompt_version`, `latency_ms`, tokens, `used_fallback`, `diverged`, `cache_hit`).
> **Guardar as duas ordens é o ponto:** é a diferença entre `rank_engine` e `rank_final` que vira a métrica de divergência do capítulo comparativo. Detalhes e justificativa em [[camada-ia-plano-implementacao|🧩 Plano da camada de IA]].

## 🧠 Decisões defensáveis

### 1. Pesos em tabela relacional, não em `JSONField`

> [!question] A alternativa descartada
> Guardar `{"eletrica": 5, "ti": 2}` num JSON dentro de `Course` seria mais rápido de escrever.

| Critério | Tabela (escolhido) | JSONField |
|---|---|---|
| Integridade referencial | ✅ FK garante que a área existe | ❌ typo vira área fantasma |
| Consulta | ✅ `filter(area_weights__weight__gte=4)` | ❌ depende de suporte a JSON |
| Edição | ✅ inline no admin | ❌ editar JSON na mão |
| Renomear área | ✅ propaga sozinho | ❌ reescrever todos os registros |

### 2. Persistir a recomendação em vez de recalcular

O resultado fica **congelado** no banco. Se os pesos de um curso mudarem amanhã, a tentativa de hoje continua mostrando o que a pessoa realmente viu. Isso dá reprodutibilidade — e um histórico analisável para a monografia.

### 3. Dois apps

`catalog` (o que a instituição oferece) e `quiz` (como se descobre o perfil) mudam por motivos diferentes. O scraping futuro toca só o `catalog`.

## ⚠️ Armadilhas encontradas

> [!warning] `unique_together` fora do `Meta`
> Solto no corpo da classe, o Django **ignora em silêncio** — sem erro, sem constraint, sem proteção.

> [!warning] Idioma misturado
> `catalog` em português e `quiz` em inglês gerou referência quebrada (`catalog.Curso`). Padronizado em inglês **antes** de gerar as migrations definitivas; migrations e `db.sqlite3` foram apagados e recriados do zero (projeto ainda sem dados reais).

> [!warning] O campo que é o núcleo da feature
> A primeira versão de `CourseAreaWeight` saiu **sem o campo `weight`** — a tabela de pesos sem peso. Ver [[bugs-e-licoes-tcc|🐛 Bugs e lições]].

## Veja também

- [[TCC|🎓 TCC]]
- [[engine-matching-cosseno|🧮 Engine de matching]]
- [[catalogo-areas-e-cursos|📚 Catálogo]]
