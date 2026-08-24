---
type: concept
area: Conceitos
status: estavel
difficulty: intermediate
id: dad-otimizacao-consultas
category: Dados
tags:
  - dados
  - concept
  - backend
  - performance
created: 2026-08-24
updated: 2026-08-24
---
# ⚡ Otimização de Consultas

> A maior parte da lentidão de uma aplicação web não está no algoritmo — está em consultas ao banco que ninguém contou.

---

## 🚨 O problema N+1

O bug de performance mais comum em qualquer ORM, e o mais fácil de não perceber.

```python
# 1 consulta para buscar os cursos
for curso in Course.objects.all():
    # + 1 consulta POR CURSO para buscar o instrutor
    print(curso.instrutor.nome)

# 20 cursos → 21 consultas
```

Em desenvolvimento, com 5 registros, é imperceptível. Em produção, com 2.000, a página trava.

### As duas soluções

```python
# select_related — para ForeignKey e OneToOne
# Faz JOIN: 1 consulta só
Course.objects.select_related("instrutor")

# prefetch_related — para ManyToMany e reverso
# Faz 2 consultas e junta em Python
Course.objects.prefetch_related("pesos__area")

# Combinados
Course.objects.select_related("instrutor").prefetch_related("pesos__area")
```

| | `select_related` | `prefetch_related` |
|---|---|---|
| Relação | FK, OneToOne (para frente) | M2M, FK reversa |
| Mecanismo | `JOIN` no SQL | Consulta separada + junção em Python |
| Consultas | 1 | 2 |

> [!success] `Prefetch` com queryset customizado resolve os casos difíceis
> ```python
> from django.db.models import Prefetch
> Course.objects.prefetch_related(
>     Prefetch("pesos", queryset=CourseAreaWeight.objects.select_related("area").order_by("-peso"))
> )
> ```
> Isso carrega os pesos já ordenados e com a área junto — três níveis em duas consultas.

---

## 🔍 Como detectar

### Em teste — a defesa permanente

```python
def test_listagem_nao_tem_n_mais_1(self):
    criar_cursos(20)
    with self.assertNumQueries(3):        # trava o número
        self.client.get("/api/cursos/")
```

> [!tip] `assertNumQueries` transforma regressão de performance em teste vermelho
> Se alguém remover o `select_related` num refactor, o teste falha imediatamente — em vez de a lentidão aparecer em produção meses depois. É a forma mais barata de proteger performance. Ver [[tst-testes-django|🐍 Testes em Django]].

### Em desenvolvimento

- **django-debug-toolbar** — mostra todas as consultas da requisição, com tempo e duplicatas
- **django-silk** — perfilamento persistente
- `connection.queries` — inspeção manual com `DEBUG=True`

---

## 🎯 As outras alavancas

### Buscar só o que usa

```python
Course.objects.only("id", "nome")          # só estas colunas
Course.objects.defer("descricao_longa")    # todas menos esta
Course.objects.values("id", "nome")        # dicts, sem instanciar model
Course.objects.values_list("id", flat=True)  # lista simples de ids
```

### Agregar no banco, não em Python

```python
# ❌ Traz tudo e soma na aplicação
total = sum(a.score for a in QuizAttempt.objects.all())

# ✅ O banco soma e devolve um número
from django.db.models import Avg, Count, Sum
total = QuizAttempt.objects.aggregate(Sum("score"))["score__sum"]

# Anotação por linha
Course.objects.annotate(
    n_pesos=Count("pesos"),
    peso_medio=Avg("pesos__peso"),
)
```

### Operações em lote

```python
Course.objects.bulk_create(objs, batch_size=500)
Course.objects.bulk_update(objs, ["nome"], batch_size=500)
Course.objects.filter(ativo=False).update(status="arquivado")   # 1 UPDATE
Course.objects.filter(...).delete()
```

> [!warning] Operações em lote não disparam `save()` nem signals
> `bulk_create` e `update()` vão direto ao SQL. Se a lógica depende de `save()` customizado, de `auto_now` ou de signals, ela **não roda**. É o preço da velocidade — e a causa de bugs sutis quando isso é esquecido.

### Existência em vez de contagem

```python
if Course.objects.filter(ativo=True).exists():   # ✅ LIMIT 1
if Course.objects.filter(ativo=True).count():    # ❌ conta tudo
if len(Course.objects.filter(ativo=True)):       # ❌❌ traz tudo pra memória
```

### Paginação sempre

```python
Course.objects.all()[:50]     # LIMIT 50 no SQL
```

Endpoint de lista sem paginação é uma bomba-relógio: funciona até a tabela crescer.

---

## 🧠 Avaliação preguiçosa

QuerySet só executa quando é consumido. Entender isso evita consultas acidentais.

```python
qs = Course.objects.filter(ativo=True)   # nada aconteceu ainda
qs = qs.filter(area="mecanica")          # ainda nada — só compôs
lista = list(qs)                         # AGORA executa

# ❌ Duas consultas idênticas
if Course.objects.filter(ativo=True):
    for c in Course.objects.filter(ativo=True): ...

# ✅ Uma consulta, resultado em cache
cursos = list(Course.objects.filter(ativo=True))
if cursos:
    for c in cursos: ...
```

---

## 📋 Checklist de diagnóstico

1. **Meça** — debug-toolbar ou `assertNumQueries`; não adivinhe
2. **Conte as consultas** — número que cresce com os dados é N+1
3. **`EXPLAIN ANALYZE`** na consulta lenta → [[dad-indices|🔎 Índices]]
4. **Reduza o volume** — `only`, `values`, paginação
5. **Empurre trabalho para o banco** — agregação, anotação
6. **Cacheie** o que é caro e muda pouco → [[Cache|Cache]]
7. **Assíncrono** o que não precisa ser imediato → [[arq-event-driven|📡 Eventos]]

> [!important] Otimize depois de medir, sempre
> Otimização por intuição costuma acertar o lugar errado e deixar o gargalo real intacto — com o bônus de tornar o código mais difícil de ler. Meça, corrija o maior item, meça de novo.

---

## 🔗 Conceitos relacionados

- [[dad-indices|🔎 Índices]] · [[dad-normalizacao|🧬 Normalização]]
- [[ORM|ORM]] · [[Cache|Cache]] · [[cs-big-o|📈 Big O]]
- [[tst-testes-django|🐍 Testes em Django]] · [[Django|Django]]

## Veja também

- [[Conceitos|🧠 Conceitos]]

---

**Status:** ✅ Completo
