---
type: concept
area: Conceitos
status: estavel
difficulty: beginner
id: dad-modelagem-relacional
category: Dados
tags:
  - dados
  - concept
  - backend
created: 2026-08-24
updated: 2026-08-24
---
# 🗺️ Modelagem Relacional

> Traduzir um problema do mundo real em tabelas, colunas e relações. É a decisão mais cara de reverter em qualquer sistema.

---

## 🧱 Os elementos

| Elemento | O que é |
|---|---|
| **Entidade** | Uma coisa sobre a qual se guarda informação → tabela |
| **Atributo** | Uma característica da entidade → coluna |
| **Chave primária** | Identifica unicamente uma linha |
| **Chave estrangeira** | Aponta para a chave primária de outra tabela |
| **Cardinalidade** | Quantos de A se relacionam com quantos de B |
| **Relacionamento** | A ligação entre entidades |

---

## 🔗 As três cardinalidades

### 1:1 — Um para um
Cada A tem no máximo um B. **Raro** — geralmente indica que as duas entidades poderiam ser uma tabela só.

```python
class Perfil(models.Model):
    user = models.OneToOneField(User, on_delete=models.CASCADE)
```

Justifica-se quando: campos opcionais e volumosos, ou dados com regra de acesso diferente.

### 1:N — Um para muitos
O caso mais comum. A FK fica **no lado "muitos"**.

```python
class Course(models.Model):
    nome = models.CharField(max_length=120)

class Turma(models.Model):
    course = models.ForeignKey(Course, on_delete=models.CASCADE, related_name="turmas")
```

### N:N — Muitos para muitos
Exige uma tabela de associação.

```python
# Sem atributo na relação — Django gerencia a tabela intermediária
class Course(models.Model):
    tags = models.ManyToManyField(Tag)

# COM atributo na relação — tabela explícita, com through
class Course(models.Model):
    areas = models.ManyToManyField(Area, through="CourseAreaWeight")

class CourseAreaWeight(models.Model):
    course = models.ForeignKey(Course, on_delete=models.CASCADE, related_name="pesos")
    area = models.ForeignKey(Area, on_delete=models.PROTECT)
    peso = models.PositiveSmallIntegerField()
```

> [!important] Se a relação tem um dado próprio, ela é uma entidade
> "Qual o peso deste curso nesta área?" não é atributo do curso nem da área — é atributo do **par**. Reconhecer isso é o que separa uma modelagem que cresce bem de uma que precisa ser refeita. Ver [[dad-normalizacao|🧬 Normalização]].

---

## 🗝️ Escolha da chave primária

| Tipo | Vantagem | Desvantagem |
|---|---|---|
| **Inteiro sequencial** | Compacto, índice eficiente, legível | Expõe volume; previsível |
| **UUID** | Não previsível, gerável no cliente | Maior; índice menos eficiente |
| **Chave natural** (CPF, código) | Sem coluna extra | Muda de valor; regras mudam |

> [!warning] Chave natural quase sempre acaba mal
> CPF é reemitido, código de curso muda de padrão, e-mail é trocado. Quando a chave primária muda, **toda FK que aponta para ela precisa mudar junto**. Use chave artificial (`id`) como primária e coloque `unique=True` na chave natural.

---

## 🧹 Comportamento na exclusão

A decisão de `on_delete` é uma regra de negócio disfarçada de detalhe técnico.

| Opção | O que faz | Quando usar |
|---|---|---|
| `CASCADE` | Apaga os filhos junto | O filho não existe sem o pai (resposta ↔ tentativa) |
| `PROTECT` | Impede a exclusão | Apagar seria perda de dado (área com cursos) |
| `SET_NULL` | Deixa a FK nula | A relação é opcional (requer `null=True`) |
| `RESTRICT` | Como PROTECT, mas em cascata parcial | Casos específicos |
| `DO_NOTHING` | Nada | Quase nunca — quebra integridade |

> [!tip] `PROTECT` como padrão para dado de referência
> Se apagar uma `Area` silenciosamente apagasse todos os pesos que dependem dela, o modelo perderia informação sem aviso. `PROTECT` transforma isso em um erro explícito, que força a decisão consciente.

---

## 📐 Diagrama ER

Todo trabalho acadêmico precisa de um. Notação **pé de galinha** (*crow's foot*):

```
┌──────────┐         ┌────────────────────┐         ┌──────────┐
│  Course  │────o<───│ CourseAreaWeight   │───>o────│   Area   │
├──────────┤ 1     N ├────────────────────┤ N     1 ├──────────┤
│ id (PK)  │         │ id (PK)            │         │ id (PK)  │
│ nome     │         │ course_id (FK)     │         │ nome     │
│ carga    │         │ area_id (FK)       │         │ slug     │
└──────────┘         │ peso (0-5)         │         └──────────┘
                     └────────────────────┘
```

Ferramentas: **dbdiagram.io** (texto → diagrama), **Mermaid** (funciona dentro do Obsidian), **django-extensions** com `graph_models` (gera do código).

```bash
python manage.py graph_models quiz catalog -o modelo.png
```

---

## ✅ Checklist de modelagem

- [ ] Toda tabela tem chave primária
- [ ] Toda FK tem `on_delete` escolhido conscientemente
- [ ] Campos obrigatórios são `null=False`
- [ ] Regras de unicidade viraram `UniqueConstraint`
- [ ] Faixas de valor viraram `CheckConstraint` ou validator
- [ ] Datas de criação/atualização existem onde importam
- [ ] Nenhuma coluna guarda lista → [[dad-normalizacao|🧬 1FN]]
- [ ] Nenhum dado derivado armazenado sem motivo declarado
- [ ] Diagrama ER gerado e conferido

> [!success] Restrição no banco é documentação executável
> `CheckConstraint(check=Q(peso__gte=0) & Q(peso__lte=5))` afirma, no lugar mais confiável possível, que o peso vive entre 0 e 5. Em uma monografia, isso permite escrever "a integridade do domínio de pesos é garantida em nível de banco" — uma afirmação verificável, não uma promessa.

---

## 📚 Referências

- **Elmasri & Navathe** — *Sistemas de Banco de Dados*
- **Chen, P. (1976)** — *The Entity-Relationship Model* — a origem do diagrama ER
- **Documentação do Django** — Model field reference

---

## 🔗 Conceitos relacionados

- [[dad-normalizacao|🧬 Normalização]] · [[dad-indices|🔎 Índices]]
- [[dad-transacoes-acid|🔒 Transações e ACID]] · [[Models|Models]] · [[Migrations|Migrations]]
- [[ORM|ORM]] · [[Banco de Dados|Banco de Dados]]

## Veja também

- [[Conceitos|🧠 Conceitos]]

---

**Status:** ✅ Completo
