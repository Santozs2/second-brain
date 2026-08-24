---
type: concept
area: Conceitos
status: estavel
difficulty: intermediate
id: dad-normalizacao
category: Dados
tags:
  - dados
  - concept
  - backend
created: 2026-08-24
updated: 2026-08-24
---
# 🧬 Normalização de Banco de Dados

> Organizar tabelas para que cada fato seja armazenado **em um só lugar**. Redundância não é desperdício de espaço — é oportunidade de contradição.

---

## 📖 O problema que ela resolve

```
❌ Tabela não normalizada

| aluno | curso              | carga | instrutor      |
|-------|--------------------|-------|----------------|
| Ana   | Mecânica Industrial| 160h  | Carlos Pereira |
| Bruno | Mecânica Industrial| 160h  | Carlos Pereira |
| Carla | Mecânica Industrial| 180h  | Carlos Pereyra |  ← 🚩
```

Três anomalias nascem daqui:

| Anomalia | O que acontece |
|---|---|
| **De atualização** | Corrigir a carga exige achar todas as linhas; esquecer uma gera contradição |
| **De inserção** | Não dá para cadastrar um curso que ainda não tem aluno |
| **De exclusão** | Apagar o último aluno apaga a informação do curso |

---

## 📐 As formas normais

### 1FN — Primeira Forma Normal
Valores atômicos; nada de lista dentro de célula.

```
❌ | curso | areas                        |
   | X     | "mecânica, elétrica, TI"     |

✅ tabela de associação com uma linha por par
```

> [!warning] Um `JSONField` com lista viola a 1FN — e às vezes tudo bem
> Guardar `["mecanica", "eletrica"]` em JSON é prático e o Postgres até indexa. O custo: você perde integridade referencial (nada impede `"mecancia"` digitado errado), perde consulta relacional natural e perde a edição no admin. **Para dado que participa de cálculo ou de relatório, use tabela. Para configuração opaca, JSON serve.**

### 2FN — Segunda Forma Normal
Está em 1FN **e** nenhum atributo depende de apenas parte de uma chave composta.

### 3FN — Terceira Forma Normal
Está em 2FN **e** nenhum atributo não-chave depende de outro atributo não-chave (sem dependência transitiva).

```
❌ Curso(id, nome, instrutor_id, instrutor_nome)
              instrutor_nome depende de instrutor_id, não do curso

✅ Curso(id, nome, instrutor_id) + Instrutor(id, nome)
```

### BCNF e além
Refinamentos para casos com múltiplas chaves candidatas. **Na prática, 3FN resolve quase tudo.**

---

## 🎯 O modelo que resolve N:N com atributo

O padrão mais útil no dia a dia: quando a relação entre duas entidades **carrega um dado próprio**.

```python
class Area(models.Model):
    nome = models.CharField(max_length=80, unique=True)
    slug = models.SlugField(unique=True)

class Course(models.Model):
    nome = models.CharField(max_length=120)
    areas = models.ManyToManyField(Area, through="CourseAreaWeight")

class CourseAreaWeight(models.Model):
    """O peso pertence à RELAÇÃO, não ao curso nem à área."""
    course = models.ForeignKey(Course, on_delete=models.CASCADE, related_name="pesos")
    area = models.ForeignKey(Area, on_delete=models.PROTECT)
    peso = models.PositiveSmallIntegerField(
        validators=[MinValueValidator(0), MaxValueValidator(5)]
    )

    class Meta:
        constraints = [
            models.UniqueConstraint(fields=["course", "area"], name="uniq_curso_area")
        ]
```

> [!success] Tabela relacional versus JSON: o argumento que se defende
> A tabela de associação entrega quatro coisas que um `JSONField` não entrega: **integridade referencial** (não existe peso para área inexistente), **validação no banco** (0–5 garantido), **unicidade** (não há peso duplicado para o mesmo par) e **editabilidade no admin** sem código. Em contexto acadêmico, isso transforma uma escolha de implementação em decisão de modelagem justificável.

---

## 🔀 Desnormalização deliberada

Nem sempre 3FN é o objetivo. Desnormalizar é aceitar redundância **em troca de performance de leitura** — desde que seja decisão consciente.

| Técnica | Quando |
|---|---|
| **Campo calculado** | Total de um pedido, recalculado a cada mudança |
| **Contador materializado** | `num_respostas` em vez de `COUNT(*)` a cada leitura |
| **Cópia histórica** | Preço no momento da compra (o preço atual **não** serve) |
| **View materializada** | Relatório caro, atualizado periodicamente |

> [!important] Cópia histórica não é desnormalização — é modelagem correta
> Guardar o preço dentro do item do pedido parece redundante, mas o preço do produto **muda** e o do pedido **não pode mudar**. São dois fatos diferentes que por acaso coincidem no momento da compra. Confundir os dois é o bug mais comum em sistema de vendas.

---

## ⚖️ O trade-off

| | Normalizado | Desnormalizado |
|---|:---:|:---:|
| Integridade | 🟢 alta | 🔴 depende da aplicação |
| Escrita | 🟢 um lugar | 🔴 vários lugares |
| Leitura | 🔴 exige JOIN | 🟢 direta |
| Espaço | 🟢 menor | 🔴 maior |
| Evolução | 🟢 fácil | 🔴 arriscada |

**Regra prática:** normalize por padrão; desnormalize quando houver **medição** mostrando que o JOIN é o gargalo. Ver [[dad-otimizacao-consultas|⚡ Otimização de consultas]].

---

## 📚 Referências

- **Codd, E. F. (1970)** — *A Relational Model of Data for Large Shared Data Banks* — o artigo fundador
- **Date, C. J.** — *An Introduction to Database Systems*
- **Elmasri & Navathe** — *Sistemas de Banco de Dados*

---

## 🔗 Conceitos relacionados

- [[dad-modelagem-relacional|🗺️ Modelagem relacional]] · [[dad-indices|🔎 Índices]]
- [[dad-transacoes-acid|🔒 Transações e ACID]] · [[Models|Models]] · [[ORM|ORM]]
- [[Banco de Dados|Banco de Dados]]

## Veja também

- [[Conceitos|🧠 Conceitos]]

---

**Status:** ✅ Completo
