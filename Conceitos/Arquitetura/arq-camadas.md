---
type: concept
area: Conceitos
status: estavel
difficulty: intermediate
id: arq-camadas
category: Arquitetura
tags:
  - arquitetura
  - concept
created: 2026-08-24
updated: 2026-08-24
---
# 🏛️ Arquitetura em Camadas

> Separar o sistema em faixas horizontais onde cada uma só conhece a de baixo. É a arquitetura mais simples que ainda resolve o problema do acoplamento.

---

## 📚 As camadas clássicas

```
┌─────────────────────────────────────┐
│  Apresentação    views, templates,  │  ← fala com o usuário
│                  serializers        │
├─────────────────────────────────────┤
│  Aplicação       casos de uso,      │  ← orquestra
│                  serviços           │
├─────────────────────────────────────┤
│  Domínio         regras, entidades, │  ← o valor do sistema
│                  algoritmos         │
├─────────────────────────────────────┤
│  Infraestrutura  ORM, HTTP, arquivo,│  ← detalhes técnicos
│                  fila, cache        │
└─────────────────────────────────────┘
```

**A regra:** cada camada só chama a camada imediatamente abaixo. Uma view não fala com o banco direto; um algoritmo não conhece HTTP.

---

## 🎯 O caso concreto: separar o cálculo do framework

O ganho mais imediato dessa separação aparece quando existe lógica não-trivial.

```python
# ═══ DOMÍNIO — não conhece Django, banco nem HTTP ═══
# quiz/engine.py
def cosine_similarity(a: list[float], b: list[float]) -> float:
    na, nb = norm(a), norm(b)
    return dot(a, b) / (na * nb) if na and nb else 0.0

def rank_courses(perfil: list[float], cursos: list[CourseVec]) -> list[Rec]:
    """Recebe dados simples, devolve dados simples."""
    return sorted(
        (Rec(c.id, cosine_similarity(perfil, c.vetor), explain(perfil, c))
         for c in cursos),
        key=lambda r: r.score, reverse=True,
    )


# ═══ APLICAÇÃO — orquestra, conhece o domínio e o repositório ═══
# quiz/services.py
def recomendar_para(attempt_id: int, n: int = 5) -> list[Rec]:
    attempt = QuizAttempt.objects.get(pk=attempt_id)
    perfil = montar_perfil(attempt)
    cursos = carregar_vetores_de_cursos()      # infraestrutura
    return rank_courses(perfil, cursos)[:n]    # domínio


# ═══ APRESENTAÇÃO — só traduz HTTP ═══
# quiz/views.py
class RecomendacaoView(APIView):
    def get(self, request, attempt_id):
        recs = recomendar_para(attempt_id)
        return Response(RecSerializer(recs, many=True).data)
```

> [!success] Três benefícios que vêm da mesma decisão
> **1. Testabilidade** — `rank_courses` roda em `SimpleTestCase`, sem banco, em milissegundos.
> **2. Portabilidade** — o mesmo módulo serve API, comando de terminal e script de experimento.
> **3. Defensabilidade** — em uma monografia, o algoritmo pode ser apresentado e verificado isoladamente, sem que o leitor precise entender Django. Ver [[tst-piramide-de-testes|🔺 Pirâmide de testes]].

---

## 🚧 Como as camadas vazam

| Vazamento | Sintoma |
|---|---|
| **Lógica na view** | A view tem 200 linhas e um `if` de regra de negócio |
| **Domínio importando ORM** | O algoritmo faz `Model.objects.filter(...)` |
| **Model gordo** | O model tem regra de negócio, envio de e-mail e formatação |
| **Salto de camada** | A view acessa o banco pulando o serviço |
| **Objeto do ORM cruzando tudo** | O domínio recebe instância de model, não dado puro |

> [!warning] O vazamento mais comum em Django é a lógica dentro do model
> Django incentiva *fat models*, e para regra simples isso é adequado. O limite aparece quando o método precisa de banco, de rede ou de configuração para ser testado. **Quando o teste do algoritmo exige criar registros no banco, a camada já vazou.**

---

## ⚖️ Quando camadas são exagero

| Contexto | Recomendação |
|---|---|
| CRUD simples, sem regra | Django MVT direto; não crie camada de serviço |
| Algoritmo central não-trivial | **Separe o domínio** — é aqui que compensa |
| Protótipo descartável | Sem camadas |
| Sistema com múltiplas entradas | Separe — API, CLI e worker compartilham o domínio |

> [!tip] Separe onde há valor, não em todo lugar
> Um projeto não precisa ser inteiramente em camadas para se beneficiar delas. **Extrair o núcleo algorítmico para um módulo puro** já entrega a maior parte do ganho, com um custo próximo de zero. O resto do sistema pode seguir o padrão do framework.

---

## 🔀 Camadas versus Clean Architecture

| | Camadas | [[arq-clean-architecture\|Clean Architecture]] |
|---|---|---|
| Direção da dependência | De cima para baixo | **Sempre para dentro** |
| Domínio conhece infra | 🟡 às vezes | ❌ nunca |
| Inversão de dependência | Opcional | Obrigatória |
| Complexidade | Baixa | Média-alta |

Camadas é o degrau anterior. Para a maioria dos projetos, é o degrau suficiente.

---

## 📚 Referências

- **Fowler, M. (2002)** — *Patterns of Enterprise Application Architecture*
- **Evans, E. (2003)** — *Domain-Driven Design*
- **Richards, M.** — *Software Architecture Patterns*, O'Reilly (livre)

---

## 🔗 Conceitos relacionados

- [[arq-solid|🧱 SOLID]] · [[arq-clean-architecture|🧅 Clean Architecture]]
- [[MVC|MVC]] · [[arq-design-patterns|🎨 Padrões de projeto]]
- [[tst-piramide-de-testes|🔺 Pirâmide de testes]]

## Veja também

- [[Conceitos|🧠 Conceitos]]

---

**Status:** ✅ Completo
