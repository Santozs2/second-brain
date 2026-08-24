---
type: concept
area: Conceitos
status: estavel
difficulty: intermediate
id: dad-indices
category: Dados
tags:
  - dados
  - concept
  - backend
  - performance
created: 2026-08-24
updated: 2026-08-24
---
# 🔎 Índices de Banco de Dados

> Uma estrutura auxiliar que troca **espaço e custo de escrita** por **velocidade de leitura**. O mesmo trade-off do índice remissivo de um livro.

---

## 📖 Por que funcionam

Sem índice, o banco faz *full table scan*: lê linha por linha. Com índice, ele navega uma árvore.

```
Sem índice:  O(n)       1.000.000 linhas → 1.000.000 leituras
Com B-tree:  O(log n)   1.000.000 linhas → ~20 leituras
```

A diferença não é percentual — é de ordem de grandeza. Ver [[cs-big-o|📈 Big O]].

---

## 🌳 Tipos de índice

| Tipo | Estrutura | Bom para |
|---|---|---|
| **B-tree** | Árvore balanceada | O padrão: `=`, `<`, `>`, `BETWEEN`, `ORDER BY`, prefixo de `LIKE` |
| **Hash** | Tabela hash | Só igualdade; não serve para faixa |
| **GIN** | Índice invertido | JSONB, arrays, busca textual |
| **GiST** | Generalizado | Dados geométricos, busca por proximidade |
| **BRIN** | Faixas por bloco | Tabelas enormes com dado correlacionado à ordem física |

Ver [[cs-tree|🌳 Árvores]] e [[cs-hash-table|🔑 Tabelas hash]].

---

## 💻 Índices no Django

```python
class QuizAttempt(models.Model):
    user = models.ForeignKey(User, on_delete=models.CASCADE)   # índice automático
    criado_em = models.DateTimeField(auto_now_add=True, db_index=True)
    status = models.CharField(max_length=20)
    score = models.FloatField()

    class Meta:
        indexes = [
            # Índice composto: a ORDEM das colunas importa
            models.Index(fields=["user", "-criado_em"], name="idx_user_recente"),
            # Índice parcial: menor e mais rápido
            models.Index(
                fields=["status"],
                condition=Q(status="pendente"),
                name="idx_pendentes",
            ),
        ]
        constraints = [
            models.UniqueConstraint(fields=["user", "criado_em"], name="uniq_tentativa")
        ]
```

> [!important] Chave estrangeira ganha índice automático no Django; chave única também
> Você não precisa adicionar `db_index=True` em `ForeignKey` nem em campo `unique=True` — já existe. Adicionar de novo cria um índice duplicado que só custa espaço e escrita.

---

## 🔑 A regra do índice composto

Um índice composto em `(A, B, C)` serve para consultas que filtram por:

```
✅ A
✅ A, B
✅ A, B, C
❌ B          ← não usa o índice
❌ B, C       ← não usa o índice
❌ C          ← não usa o índice
```

É o **princípio do prefixo à esquerda**: o índice só é útil a partir da primeira coluna, em ordem.

> [!tip] Coloque primeiro a coluna de igualdade, depois a de faixa/ordenação
> Para `WHERE user_id = 5 ORDER BY criado_em DESC`, o índice correto é `(user_id, -criado_em)` — nessa ordem. O inverso força o banco a ordenar em memória depois de filtrar.

---

## 💸 O custo dos índices

| Custo | Detalhe |
|---|---|
| **Escrita mais lenta** | Todo `INSERT`/`UPDATE`/`DELETE` atualiza cada índice |
| **Espaço em disco** | Pode ultrapassar o tamanho da própria tabela |
| **Manutenção** | Fragmentação; o planejador precisa de estatísticas atualizadas |
| **Planejamento pior** | Índices demais confundem o otimizador |

> [!warning] Indexar tudo é tão ruim quanto não indexar nada
> Uma tabela com dez índices tem escritas várias vezes mais caras, e o planejador passa a escolher mal. **Índice se adiciona em resposta a uma consulta lenta identificada**, não por precaução.

---

## 🔬 Como saber se o índice está sendo usado

```sql
EXPLAIN ANALYZE
SELECT * FROM quiz_quizattempt WHERE user_id = 5 ORDER BY criado_em DESC LIMIT 10;
```

O que procurar na saída:

| Sinal | Leitura |
|---|---|
| `Index Scan using idx_...` | ✅ o índice está sendo usado |
| `Seq Scan` em tabela grande | 🚩 falta índice, ou ele foi ignorado |
| `Bitmap Heap Scan` | 🟡 índice usado, muitas linhas retornadas |
| `Rows Removed by Filter` alto | 🚩 o índice não é seletivo o bastante |

No Django:

```python
print(QuizAttempt.objects.filter(user_id=5).order_by("-criado_em").explain(analyze=True))
```

---

## 🚫 Quando o índice é ignorado

| Situação | Por quê |
|---|---|
| Função sobre a coluna: `WHERE LOWER(nome) = 'x'` | Índice é sobre `nome`, não sobre `LOWER(nome)` |
| `LIKE '%texto'` | Curinga à esquerda impede navegação na árvore |
| Baixa seletividade | Se 80% das linhas casam, ler tudo é mais rápido |
| Tipos incompatíveis | Comparar `varchar` com número força conversão |
| Tabela pequena | Ler tudo é mais barato que abrir o índice |

**Solução para o primeiro caso** — índice funcional:

```python
models.Index(Lower("nome"), name="idx_nome_lower")
```

---

## 📚 Referências

- **Winand, M.** — *Use The Index, Luke!* — use-the-index-luke.com, o melhor material gratuito sobre o tema
- **Documentação do PostgreSQL** — cap. 11 (Indexes)
- **Kleppmann, M. (2017)** — *Designing Data-Intensive Applications*, cap. 3

---

## 🔗 Conceitos relacionados

- [[dad-otimizacao-consultas|⚡ Otimização de consultas]] · [[dad-normalizacao|🧬 Normalização]]
- [[cs-tree|🌳 Árvores]] · [[cs-hash-table|🔑 Tabelas hash]] · [[cs-big-o|📈 Big O]]
- [[Banco de Dados|Banco de Dados]] · [[ORM|ORM]]

## Veja também

- [[Conceitos|🧠 Conceitos]]

---

**Status:** ✅ Completo
