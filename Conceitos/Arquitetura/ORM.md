---
type: concept
area: Conceitos
status: estavel
tags:
  - concept
created: 2026-06-30
updated: 2026-08-24
---
# ORM

## Definição

Object-Relational Mapping: técnica que mapeia tabelas do banco de dados para classes e objetos da linguagem de programação, evitando escrever SQL manualmente na maior parte dos casos.

```
Tabela  ←→  Classe (Model)
Linha   ←→  Instância
Coluna  ←→  Atributo
FK      ←→  Referência a outro objeto
```

## Quando usar

Na maior parte das operações de [[CRUD|CRUD]] em aplicações Django — o ORM gera o SQL por trás dos panos a partir dos [[Models|Models]].

**Quando escrever SQL puro:** consultas analíticas complexas, operações em massa com performance crítica, recursos específicos do banco (window functions, CTEs recursivas) que o ORM não expõe bem.

```python
from django.db import connection

with connection.cursor() as cur:
    cur.execute("SELECT area_id, AVG(peso) FROM catalog_pesos GROUP BY area_id")
    resultado = cur.fetchall()
```

## O que o ORM entrega

| Ganho | Detalhe |
|---|---|
| **Portabilidade** | O mesmo código roda em SQLite, Postgres e MySQL |
| **Segurança** | Parametriza consultas — protege contra [[cs-sql-injection\|SQL injection]] por padrão |
| **Produtividade** | Não escrever SQL repetitivo de CRUD |
| **Migrations** | O esquema evolui versionado → [[Migrations\|Migrations]] |
| **Validação** | Tipos e restrições declarados uma vez |

> [!important] A proteção contra SQL injection vem da parametrização, e você pode desfazê-la
> `Model.objects.filter(nome=entrada)` é seguro. `Model.objects.raw(f"SELECT ... WHERE nome = '{entrada}'")` **não é** — a f-string reintroduz exatamente a vulnerabilidade que o ORM evitava. Em `raw()` e `extra()`, sempre passe parâmetros:
> ```python
> Model.objects.raw("SELECT * FROM t WHERE nome = %s", [entrada])
> ```

## O problema da impedância objeto-relacional

Objetos e tabelas são modelos de dados diferentes, e a tradução nunca é perfeita:

| Objetos | Tabelas |
|---|---|
| Herança | Não existe nativamente |
| Identidade por referência | Identidade por chave primária |
| Navegação por atributo (barata) | JOIN (custa) |
| Coleções aninhadas | Tabelas separadas |

É dessa lacuna que nascem as armadilhas — principalmente a navegação por atributo, que **parece** barata e é uma consulta.

## Boas práticas

- **Evitar consultas N+1** — usar `select_related` / `prefetch_related` → [[dad-otimizacao-consultas|⚡ Otimização de consultas]]
- Usar o ORM para a maioria dos casos, e SQL puro só quando necessário para performance
- Entender a **avaliação preguiçosa**: o QuerySet só executa quando é consumido
- Agregar no banco (`aggregate`, `annotate`), não em Python
- Usar `exists()` em vez de `count()` para checar existência
- Usar `F()` para operações atômicas em vez de ler-modificar-salvar
- Medir com `assertNumQueries` e `EXPLAIN ANALYZE`

```python
# ❌ N+1 silencioso: uma consulta por curso
for curso in Course.objects.all():
    print(curso.instrutor.nome)

# ✅ Uma consulta
for curso in Course.objects.select_related("instrutor"):
    print(curso.instrutor.nome)
```

## Quando o ORM atrapalha

> [!warning] O ORM esconde o custo, não o elimina
> `curso.instrutor.nome` parece acesso a atributo e é uma consulta ao banco. Em um laço, isso vira dezenas de idas ao banco sem nada no código sinalizando. **O ORM é uma abstração vazante** — em algum momento é preciso olhar o SQL gerado para entender o que está acontecendo.

```python
print(Course.objects.filter(ativo=True).query)          # o SQL gerado
print(Course.objects.filter(ativo=True).explain())      # o plano de execução
```

Esta é uma instância da **Lei das Abstrações Vazantes** (*Joel Spolsky*): toda abstração não-trivial vaza, e quem a usa acaba precisando entender o que está por baixo.

## Active Record × Data Mapper

| | Active Record | Data Mapper |
|---|---|---|
| Exemplo | Django ORM, Rails | SQLAlchemy, Doctrine |
| O objeto sabe se salvar | ✅ `obj.save()` | ❌ `session.add(obj)` |
| Acoplamento domínio↔persistência | Alto | Baixo |
| Simplicidade | ✅ | 🟡 |
| Domínio puro e testável | ❌ | ✅ |

O Django adota **Active Record** — mais simples e produtivo, ao custo de o model de domínio estar acoplado ao banco. Quando isso incomoda, a saída é extrair a lógica para funções puras. Ver [[arq-camadas|🏛️ Arquitetura em camadas]].

## Conceitos relacionados

- [[Models|Models]] · [[Migrations|Migrations]] · [[CRUD|CRUD]]
- [[dad-otimizacao-consultas|⚡ Otimização de consultas]] · [[dad-indices|🔎 Índices]]
- [[dad-modelagem-relacional|🗺️ Modelagem relacional]] · [[cs-sql-injection|🚨 SQL Injection]]
- [[arq-camadas|🏛️ Arquitetura em camadas]]

## Veja também

- [[Documentações|Documentações]]
- [[Banco de Dados|Banco de Dados]]
