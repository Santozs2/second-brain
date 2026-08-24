---
type: concept
area: Conceitos
status: estavel
difficulty: intermediate
id: arq-monolito-vs-microservicos
category: Arquitetura
tags:
  - arquitetura
  - concept
  - devops
created: 2026-08-24
updated: 2026-08-24
---
# 🧊 Monólito × Microsserviços

> A escolha não é sobre qual arquitetura é melhor. É sobre qual problema você tem — e a maioria dos projetos tem o problema que o monólito resolve.

---

## 📖 As duas formas

### Monólito
Uma aplicação, um processo, um deploy. Os módulos se comunicam por chamada de função.

```
┌──────────────────────────────┐
│         Aplicação            │
│  ┌────┐ ┌────┐ ┌────┐        │
│  │quiz│ │auth│ │cat.│        │  chamadas diretas
│  └────┘ └────┘ └────┘        │
└──────────────┬───────────────┘
           ┌───▼───┐
           │  BD   │
           └───────┘
```

### Microsserviços
Serviços independentes, cada um com seu deploy e seu banco, comunicando por rede.

```
┌──────┐   ┌──────┐   ┌──────┐
│ quiz │──▶│ auth │──▶│ cat. │    HTTP / fila
└──┬───┘   └──┬───┘   └──┬───┘
 ┌─▼─┐     ┌──▼┐      ┌──▼┐
 │BD │     │BD │      │BD │
 └───┘     └───┘      └───┘
```

---

## ⚖️ Comparação honesta

| Critério | Monólito | Microsserviços |
|---|:---:|:---:|
| Complexidade inicial | 🟢 baixa | 🔴 alta |
| Deploy | 🟢 um | 🔴 N pipelines |
| Depuração | 🟢 stack trace única | 🔴 rastreamento distribuído |
| Transação entre módulos | 🟢 ACID nativo | 🔴 saga / consistência eventual |
| Escala independente | 🔴 escala tudo junto | 🟢 escala só o gargalo |
| Times independentes | 🔴 coordenação alta | 🟢 autonomia |
| Falha isolada | 🔴 cai tudo | 🟢 degrada em parte |
| Custo de infraestrutura | 🟢 baixo | 🔴 alto |
| Latência interna | 🟢 nanossegundos | 🔴 milissegundos |

---

## 🎯 O que realmente decide

> [!important] Microsserviços resolvem um problema organizacional antes de resolver um problema técnico
> A motivação original é permitir que **times deployem sem coordenar entre si**. Com um time — ou com quatro pessoas em um TCC — esse problema não existe. Adotar microsserviços aí importa todo o custo operacional e nenhum benefício.

**A Lei de Conway** (*Conway, 1968*): a arquitetura de um sistema tende a espelhar a estrutura de comunicação da organização que o constrói. Um time único produz naturalmente um sistema único — e forçar o contrário gera atrito permanente.

| Se você tem... | Escolha |
|---|---|
| 1 time, < 10 pessoas | **Monólito** |
| Carga uniforme | **Monólito** |
| Domínio ainda em descoberta | **Monólito** |
| Prazo curto | **Monólito** |
| Múltiplos times autônomos | Microsserviços |
| Um componente com escala muito diferente | Microsserviços (ou extração pontual) |
| Requisito de isolamento de falha | Microsserviços |

---

## ⭐ Monólito modular: o meio-termo que quase sempre vence

Um monólito com fronteiras internas bem definidas — módulos que se comunicam por interfaces explícitas, não por acesso direto às tabelas uns dos outros.

```
projeto/
├── quiz/           ← módulo: expõe uma API interna clara
├── catalog/        ← módulo: dono das suas tabelas
├── accounts/       ← módulo
└── shared/         ← contratos e tipos comuns

Regra: nenhum módulo consulta as tabelas de outro diretamente.
```

> [!success] O monólito modular preserva a opção de extrair depois
> Você paga o custo baixo do monólito hoje e mantém a possibilidade de extrair um módulo em serviço quando — e **se** — houver motivo real. O contrário não é verdade: um monólito emaranhado não vira microsserviço sem reescrita.

---

## 🚨 O antipadrão: monólito distribuído

O pior dos dois mundos. Serviços separados que **precisam ser deployados juntos** porque estão acoplados.

**Sinais:** mudar um serviço obriga a mudar outro; serviços compartilham banco; uma requisição atravessa seis serviços; nenhum serviço funciona sozinho em ambiente local.

---

## 📐 O caminho recomendado

```
1. Comece monolítico
2. Mantenha fronteiras internas limpas
3. Meça onde dói de verdade
4. Extraia SÓ o que tem motivo demonstrável
```

*Martin Fowler* chama isto de **MonolithFirst**: quase todo sistema de microsserviços bem-sucedido começou como um monólito que ficou grande demais, e quase todo sistema que já nasceu em microsserviços teve problemas sérios.

> [!tip] Em contexto acadêmico, escolher monólito e justificar é mais forte que escolher microsserviços e sofrer
> Uma banca não premia complexidade — premia **adequação justificada**. "Adotou-se arquitetura monolítica modular, uma vez que o contexto de time único e carga uniforme não apresenta os fatores que motivam a distribuição (NEWMAN, 2021), enquanto o custo operacional seria incompatível com o prazo do trabalho" é uma decisão defendida. Ver [[met-defesa-banca|🎤 Defesa de banca]].

---

## 📚 Referências

- **Newman, S. (2021)** — *Building Microservices*, 2ª ed., O'Reilly
- **Fowler, M.** — *MonolithFirst* e *MicroservicePremium*, martinfowler.com
- **Conway, M. (1968)** — *How Do Committees Invent?*
- **Richardson, C.** — *Microservices Patterns*

---

## 🔗 Conceitos relacionados

- [[arq-camadas|🏛️ Arquitetura em camadas]] · [[arq-event-driven|📡 Arquitetura orientada a eventos]]
- [[arq-clean-architecture|🧅 Clean Architecture]] · [[cs-load-balancing|⚖️ Load balancing]]
- [[Docker|Docker]] · [[CI-CD|CI/CD]]

## Veja também

- [[Conceitos|🧠 Conceitos]]

---

**Status:** ✅ Completo
