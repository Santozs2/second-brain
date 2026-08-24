---
type: concept
area: Conceitos
status: estavel
difficulty: intermediate
id: dad-transacoes-acid
category: Dados
tags:
  - dados
  - concept
  - backend
created: 2026-08-24
updated: 2026-08-24
---
# 🔒 Transações e ACID

> Um conjunto de operações que acontece **inteiro ou não acontece**. É a garantia que impede o sistema de ficar em um estado que não deveria existir.

---

## 🧱 As quatro propriedades

| Propriedade | Garante |
|---|---|
| **A**tomicidade | Tudo ou nada; falha no meio desfaz o que já foi feito |
| **C**onsistência | O banco vai de um estado válido a outro estado válido |
| **I**solamento | Transações concorrentes não enxergam estados intermediários |
| **D**urabilidade | Depois do commit, sobrevive a queda de energia |

---

## 💻 Transações no Django

```python
from django.db import transaction

# Como decorator — a view inteira é uma transação
@transaction.atomic
def registrar_tentativa(request):
    attempt = QuizAttempt.objects.create(user=request.user)
    for resp in request.data["respostas"]:
        Answer.objects.create(attempt=attempt, **resp)
    gerar_recomendacoes(attempt)
    # Exceção em qualquer ponto → rollback de tudo

# Como bloco — escopo menor, preferível
def processar(dados):
    validar(dados)                      # fora da transação
    with transaction.atomic():
        obj = Model.objects.create(**dados)
        atualizar_contadores(obj)
    notificar(obj)                      # fora — não deve segurar a transação
```

> [!warning] Nunca faça chamada de rede dentro de uma transação
> Uma chamada HTTP com timeout de 10 segundos dentro de `atomic()` mantém locks no banco por 10 segundos. Sob concorrência, isso vira fila e depois *deadlock*. **Chamada externa, envio de e-mail e tarefa assíncrona vão para fora do bloco** — ou para depois do commit.

### Executar só depois do commit

```python
from django.db import transaction

with transaction.atomic():
    attempt = QuizAttempt.objects.create(...)
    # Só enfileira se a transação realmente for confirmada
    transaction.on_commit(lambda: gerar_entrega.delay(attempt.id))
```

Sem `on_commit`, a tarefa assíncrona pode rodar antes do commit e não encontrar o registro — uma corrida clássica e difícil de reproduzir.

---

## 🔀 Níveis de isolamento

| Nível | Leitura suja | Leitura não repetível | Leitura fantasma |
|---|:---:|:---:|:---:|
| Read Uncommitted | ✅ ocorre | ✅ | ✅ |
| **Read Committed** | ❌ | ✅ | ✅ |
| Repeatable Read | ❌ | ❌ | ✅ |
| Serializable | ❌ | ❌ | ❌ |

**Padrão do PostgreSQL:** Read Committed. **Padrão do MySQL/InnoDB:** Repeatable Read.

### Os fenômenos

- **Leitura suja** — você lê algo que outra transação ainda não confirmou (e pode desfazer)
- **Leitura não repetível** — a mesma linha, lida duas vezes, devolve valores diferentes
- **Leitura fantasma** — a mesma consulta devolve um número diferente de linhas

---

## 🏁 Condições de corrida

O cenário clássico: duas requisições simultâneas lendo e escrevendo o mesmo registro.

```python
# ❌ Corrida: as duas leem 10, as duas escrevem 11 — perdeu-se uma
vaga = Vaga.objects.get(pk=1)
vaga.inscritos += 1
vaga.save()
```

### Solução 1 — Operação atômica no banco (a mais simples)

```python
from django.db.models import F

Vaga.objects.filter(pk=1).update(inscritos=F("inscritos") + 1)
# Vira UPDATE ... SET inscritos = inscritos + 1 — atômico no SQL
```

### Solução 2 — Lock pessimista

```python
with transaction.atomic():
    vaga = Vaga.objects.select_for_update().get(pk=1)   # trava a linha
    if vaga.inscritos < vaga.limite:
        vaga.inscritos += 1
        vaga.save()
```

`select_for_update()` bloqueia as outras transações até o commit. Correto, porém serializa o acesso — use quando a lógica for complexa demais para um `F()`.

### Solução 3 — Constraint no banco (a defesa final)

```python
class Meta:
    constraints = [
        models.CheckConstraint(check=Q(inscritos__lte=F("limite")), name="ck_limite"),
        models.UniqueConstraint(fields=["user", "vaga"], name="uniq_inscricao"),
    ]
```

> [!important] Validação na aplicação não substitui constraint no banco
> Entre o `if` da aplicação e o `save()` existe uma janela em que outro processo pode agir. **Só o banco consegue garantir a regra sob concorrência.** A validação na aplicação existe para dar mensagem de erro amigável; a constraint existe para tornar o estado inválido impossível.

---

## 🌐 Transações distribuídas

Quando a operação atravessa serviços ou bancos, ACID deixa de estar disponível. As alternativas:

| Padrão | Como funciona |
|---|---|
| **Saga** | Sequência de transações locais com compensação em caso de falha |
| **Outbox** | Grava o evento na mesma transação; publica depois |
| **Consistência eventual** | Aceita divergência temporária, converge depois |

Ver [[arq-event-driven|📡 Arquitetura orientada a eventos]] e [[arq-monolito-vs-microservicos|🧊 Monólito × Microsserviços]].

> [!tip] Manter tudo em um banco é uma vantagem que se perde rápido
> ACID entre módulos é de graça em um monólito com um banco só. No momento em que o sistema é dividido em serviços com bancos separados, essa garantia desaparece e precisa ser reconstruída à mão, com saga e compensação. É um dos custos mais subestimados da distribuição.

---

## 📚 Referências

- **Kleppmann, M. (2017)** — *Designing Data-Intensive Applications*, cap. 7 — a melhor explicação de isolamento
- **Bailis et al. (2013)** — *Highly Available Transactions*
- **Documentação do PostgreSQL** — cap. 13 (Concurrency Control)
- **Documentação do Django** — `django.db.transaction`

---

## 🔗 Conceitos relacionados

- [[dad-normalizacao|🧬 Normalização]] · [[dad-indices|🔎 Índices]]
- [[cs-synchronization|🔄 Sincronização]] · [[arq-event-driven|📡 Eventos]]
- [[Banco de Dados|Banco de Dados]] · [[ORM|ORM]]

## Veja também

- [[Conceitos|🧠 Conceitos]]

---

**Status:** ✅ Completo
