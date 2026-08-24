---
type: concept
area: Conceitos
status: estavel
difficulty: advanced
id: arq-clean-architecture
category: Arquitetura
tags:
  - arquitetura
  - concept
created: 2026-08-24
updated: 2026-08-24
---
# 🧅 Clean Architecture

> Uma regra só: as dependências apontam **para dentro**. O núcleo de regras de negócio não sabe que existe banco, web ou framework.

---

## 🎯 A regra da dependência

```
        ┌──────────────────────────────────┐
        │  Frameworks & Drivers            │  Django, React, Postgres
        │  ┌────────────────────────────┐  │
        │  │  Adaptadores de Interface  │  │  views, serializers, repos
        │  │  ┌──────────────────────┐  │  │
        │  │  │  Casos de Uso        │  │  │  serviços de aplicação
        │  │  │  ┌────────────────┐  │  │  │
        │  │  │  │   Entidades    │  │  │  │  regras de negócio
        │  │  │  └────────────────┘  │  │  │
        │  │  └──────────────────────┘  │  │
        │  └────────────────────────────┘  │
        └──────────────────────────────────┘
              dependências apontam ──►  para DENTRO
```

Nada de uma camada interna pode mencionar nada de uma camada externa. Uma entidade **não importa** Django. Um caso de uso **não sabe** se a entrada veio de HTTP ou de um script.

---

## 🔄 Como inverter a dependência do banco

O caso de uso precisa de dados, mas não pode depender do ORM. A solução é o **protocolo definido na camada de dentro e implementado na de fora**.

```python
# ═══ DENTRO — casos de uso definem o que precisam ═══
class CursoRepo(Protocol):
    def todos_com_vetores(self) -> list[CourseVec]: ...

def recomendar(perfil: list[float], repo: CursoRepo, n: int = 5) -> list[Rec]:
    """Caso de uso puro: não importa Django, não sabe de HTTP."""
    return rank_courses(perfil, repo.todos_com_vetores())[:n]


# ═══ FORA — a infraestrutura implementa o protocolo ═══
class DjangoCursoRepo:
    def todos_com_vetores(self) -> list[CourseVec]:
        return [CourseVec(c.id, montar_vetor(c))
                for c in Course.objects.prefetch_related("pesos")]
```

> [!important] A seta se inverte: agora a infraestrutura depende do domínio
> Sem o protocolo, `recomendar` importaria `Course` e a dependência apontaria para fora. Com ele, quem depende é `DjangoCursoRepo` — que precisa se conformar ao contrato do domínio. **Essa inversão é o coração da arquitetura**, e é o D do [[arq-solid|SOLID]] aplicado em escala de sistema.

---

## ✅ O que a arquitetura entrega

| Ganho | Como se manifesta |
|---|---|
| **Testável sem infraestrutura** | Caso de uso testado com repositório em memória |
| **Framework adiável** | O núcleo funciona antes de escolher a stack |
| **Banco substituível** | Trocar Postgres por outra coisa não toca o domínio |
| **Regra de negócio localizável** | Está em um lugar, não espalhada em views |
| **Domínio legível** | Lê-se como o problema, não como o framework |

---

## ⚠️ O custo, honestamente

| Custo | Realidade |
|---|---|
| **Muito mais arquivos** | Uma funcionalidade toca 5–6 arquivos |
| **Mapeamento repetitivo** | Converter model ↔ entidade ↔ DTO |
| **Briga com o framework** | Django foi feito para *fat models*, não para isto |
| **Curva de equipe** | Pessoas novas demoram a achar as coisas |

> [!warning] Clean Architecture completa raramente compensa em projeto pequeno
> Em um CRUD com regra simples, ela produz cinco camadas de indireção para salvar um formulário. O ganho aparece quando existe **regra de negócio complexa e estável** que sobrevive a mudanças de tecnologia — não quando o sistema é majoritariamente entrada e saída de dados.

---

## 🎯 O meio-termo que quase sempre é o certo

Aplique a inversão **apenas onde há valor**, e siga o framework no resto:

```
quiz/
├── engine.py       ← DOMÍNIO puro: sem imports de Django ✅
├── services.py     ← casos de uso: orquestram engine + repos
├── models.py       ← Django normal
├── views.py        ← Django normal
└── serializers.py  ← Django normal
```

> [!success] Um módulo puro já entrega a maior parte do benefício
> `engine.py` sem nenhum import de framework é testável, portável, verificável à mão e apresentável em uma monografia sem que o leitor precise conhecer Django. **Isso custa uma decisão de organização, não uma reengenharia.** O resto do sistema pode continuar sendo Django idiomático.

---

## 🧭 Arquiteturas da mesma família

| Nome | Autor | Ideia central |
|---|---|---|
| **Hexagonal / Ports & Adapters** | Alistair Cockburn (2005) | Núcleo com portas; adaptadores nas bordas |
| **Onion Architecture** | Jeffrey Palermo (2008) | Camadas concêntricas em volta do domínio |
| **Clean Architecture** | Robert C. Martin (2012) | Síntese das anteriores |

São essencialmente a mesma ideia com vocabulários diferentes: **isolar o domínio e inverter as dependências**.

---

## 📚 Referências

- **Martin, R. C. (2017)** — *Clean Architecture: A Craftsman's Guide to Software Structure and Design*
- **Cockburn, A. (2005)** — *Hexagonal Architecture*
- **Percival & Gregory (2020)** — *Architecture Patterns with Python* — o mais aplicável ao ecossistema Python

---

## 🔗 Conceitos relacionados

- [[arq-camadas|🏛️ Arquitetura em camadas]] · [[arq-solid|🧱 SOLID]]
- [[arq-design-patterns|🎨 Padrões de projeto]] · [[MVC|MVC]]
- [[tst-piramide-de-testes|🔺 Pirâmide de testes]]

## Veja também

- [[Conceitos|🧠 Conceitos]]

---

**Status:** ✅ Completo
