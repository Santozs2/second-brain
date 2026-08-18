---
title: "Testes e validação — suíte automatizada da engine"
aliases: ["Testes do TCC", "Suíte de testes do quiz"]
tags: [tcc, testes, django, qualidade, unittest]
status: concluido
projeto: TCC
criado: 2026-08-17
---

> [!info] Projeto: [[TCC|🎓 TCC]] · Ver também: [[engine-matching-cosseno|🧮 Engine]] · [[front-templates-django|🎨 Front]]

# ✅ Testes e validação

> [!abstract] Objetivo
> Sair da conferência visual ("olhei o ranking e pareceu certo") para `python manage.py test` respondendo **quebrou ou não**. Sem dependência nova: `unittest` nativo do Django e SQLite criando um banco temporário.

```
Ran 12 tests in 0.164s

OK
```

## 🧱 Duas classes, dois custos

| Classe | Base | Testes | Toca o banco? |
|---|---|---|---|
| `MatematicaTest` | `SimpleTestCase` | 4 | ❌ milissegundos |
| `RecomendacaoTest` | `TestCase` | 8 | ✅ com rollback |

> [!important] Isso só é possível por causa da arquitetura
> Testar a matemática sem abrir banco **só funciona porque a engine tem funções puras separadas do ORM**. É a decisão do [[engine-matching-cosseno|passo 4]] se pagando na prática — e um parágrafo pronto para a monografia.

## 🧮 `MatematicaTest` — as funções puras

| Teste | O que protege |
|---|---|
| `test_vetores_identicos_dao_1` | Sanidade do cosseno |
| `test_sem_area_em_comum_da_0` | Vetores ortogonais |
| `test_vetor_vazio_nao_estoura` | Guarda do divisor zero (quiz em branco) |
| `test_escala_nao_muda_o_score` | **A decisão de usar cosseno** |

> [!success] O teste que protege a arquitetura, não o código
> `test_escala_nao_muda_o_score` dobra todos os pesos do perfil e exige o mesmo resultado. Se alguém trocar o cosseno por uma soma de produtos, é o primeiro a quebrar. Ele não testa uma linha — testa uma **decisão de projeto**.

## 🎯 `RecomendacaoTest` — comportamento fim a fim

Os 4 critérios de aceite viraram assert:

```python
def test_perfil_misto_prefere_o_curso_hibrido(self):
    ranking = self.responder("Desmontar máquinas", "Oficina", "Eletricidade", "diagnóstico")
    self.assertLess(
        ranking.index("Injeção Eletrônica Automotiva"),
        ranking.index("Mecânica de Motores a Combustão"),
    )
```

Mais quatro que cobrem casos que ninguém lembra de testar na mão:

| Teste | O que protege |
|---|---|
| `test_quiz_em_branco_nao_quebra` | 5 recomendações com score 0, sem exceção |
| `test_resultado_e_reprodutivel` | Desempate por nome — mesma entrada, mesma saída |
| `test_recalcular_nao_duplica` | O `delete()` antes do `bulk_create` (5, não 10) |
| `test_explicacao_aponta_a_area_certa` | O `explanation` não é decorativo |

## 🌱 Os seeds como fixture

```python
@classmethod
def setUpTestData(cls):
    silencio = StringIO()
    for comando in ("seed_areas", "seed_courses", "seed_questions"):
        call_command(comando, stdout=silencio)
```

Roda **uma vez** para a classe inteira; cada teste roda em transação com rollback.

> [!tip] Dois benefícios de uma vez
> 1. Os dados de teste são exatamente os dados reais do projeto — nada de fixture paralela que envelhece.
> 2. **Os seeds passam a ser testados de brinde.** Se um deles quebrar, os 8 testes de recomendação caem junto e o erro aparece na hora.

## 🔍 `Choice.objects.get()` em vez de `filter().first()`

O helper `responder()` busca a alternativa por trecho de texto. Usando `get()`, um trecho ambíguo estoura com `MultipleObjectsReturned` — exatamente o bug que apareceu no `test_engine`, onde `"computador"` casava com alternativas de **duas** perguntas e gerava `IntegrityError` no `unique_together`.

> [!warning] O princípio
> Em teste, prefira a função que **falha alto** à que escolhe em silêncio. `first()` pegaria a alternativa errada e o teste passaria medindo a coisa errada.

## ▶️ Como rodar

```bash
PYTHONIOENCODING=utf-8 ./.venv/Scripts/python.exe manage.py test
PYTHONIOENCODING=utf-8 ./.venv/Scripts/python.exe manage.py test quiz.tests.RecomendacaoTest
PYTHONIOENCODING=utf-8 ./.venv/Scripts/python.exe manage.py test quiz -v 2
```

## 🎭 Teste automatizado ≠ comando de demonstração

| | `manage.py test` | `manage.py test_engine` |
|---|---|---|
| Responde | "quebrou?" | "como ficou?" |
| Saída | OK / FAILED | ranking com scores e áreas |
| Uso | antes de commitar | na apresentação e ao calibrar pesos |

Os dois convivem: um **garante**, o outro **demonstra**.

## 🧪 Validação manual complementar

- **Smoke test do site** (6 cenários, todos PASS) — ver [[front-templates-django|🎨 Front]].
- **Teste manual via Django Admin**: criar `QuizAttempt` com respostas, rodar `recommend()` no shell e conferir a grade de `Recommendations` no rodapé da tentativa.

> [!warning] Pegadinha do teste manual pelo admin
> O admin **não impede** escolher uma alternativa que não pertence à pergunta selecionada — o model não tem essa validação. Se acontecer, o score sai estranho e parece bug da engine. As duas pontas (API e site) validam isso; o admin não.

## Veja também

- [[TCC|🎓 TCC]]
- [[engine-matching-cosseno|🧮 Engine de matching]]
- [[bugs-e-licoes-tcc|🐛 Bugs e lições]]
