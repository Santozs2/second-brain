---
type: concept
area: Conceitos
status: estavel
difficulty: beginner
id: tst-piramide-de-testes
category: Testes
tags:
  - testes
  - concept
  - qualidade
created: 2026-08-24
updated: 2026-08-24
---
# 🔺 Pirâmide de Testes

> Muitos testes rápidos e baratos na base; poucos testes lentos e caros no topo. Inverter a proporção produz uma suíte que ninguém roda.

---

## 📖 O modelo

Proposto por *Mike Cohn (2009)*, popularizado por *Martin Fowler*:

```
              /\
             /  \      E2E          poucos · lentos · frágeis
            /────\                  minutos
           /      \
          /        \   Integração   alguns · médios
         /──────────\               segundos
        /            \
       /              \ Unidade     muitos · rápidos · estáveis
      /────────────────\            milissegundos
```

| Nível | O que testa | Velocidade | Proporção sugerida |
|---|---|---|---|
| **Unidade** | Uma função ou classe isolada | ms | ~70% |
| **Integração** | Componentes conversando (com banco, API) | s | ~20% |
| **E2E** | O sistema inteiro pela interface | min | ~10% |

---

## 1️⃣ Testes de unidade

Testam uma unidade de lógica **sem** banco, rede ou arquivo.

```python
def test_cosseno_vetores_identicos(self):
    """Vetores idênticos têm similaridade 1."""
    v = [3.0, 1.0, 4.0]
    self.assertAlmostEqual(cosine_similarity(v, v), 1.0, places=6)

def test_cosseno_vetor_nulo_nao_estoura(self):
    """Vetor nulo retorna 0, não ZeroDivisionError."""
    self.assertEqual(cosine_similarity([0, 0, 0], [1, 2, 3]), 0.0)

def test_cosseno_ignora_escala(self):
    """Vetores proporcionais são idênticos em direção."""
    self.assertAlmostEqual(
        cosine_similarity([1, 2], [2, 4]), 1.0, places=6
    )
```

> [!success] Lógica pura é o que torna a base da pirâmide barata
> Uma função que só recebe números e devolve número roda em microssegundos, não precisa de banco e nunca fica *flaky*. **Separar a lógica de negócio do framework é a decisão que faz a pirâmide funcionar** — e é a mesma decisão que torna o algoritmo defensável em uma monografia, porque ele pode ser verificado à mão. Ver [[arq-camadas|🏛️ Arquitetura em camadas]].

---

## 2️⃣ Testes de integração

Verificam que as peças se encaixam: view + serializer + banco, ou serviço + API externa mockada.

```python
class TestAPIQuiz(TestCase):
    def test_envio_de_respostas_gera_ranking(self):
        resp = self.client.post("/api/quiz/respostas/", self.payload, format="json")
        self.assertEqual(resp.status_code, 201)
        self.assertEqual(len(resp.data["ranking"]), 5)
        self.assertIn("explanation", resp.data["ranking"][0])
```

---

## 3️⃣ Testes E2E

Simulam o usuário real no navegador (Playwright, Cypress, Selenium).

✅ Únicos que provam que o sistema realmente funciona ponta a ponta
❌ Lentos, quebram por mudança de CSS, difíceis de depurar

**Regra prática:** cubra apenas os **caminhos críticos** — aqueles cuja quebra inviabiliza o produto. Para um sistema de quiz: responder o quiz até ver o resultado. Só isso.

---

## 🔄 Os antipadrões

### 🍦 Cone de sorvete (a pirâmide invertida)

```
    \                    /
     \    E2E           /    ← muitos
      \────────────────/
       \  Integração  /
        \────────────/
         \ Unidade  /        ← poucos
          \────────/
           \  🍦  /
```

Suíte que leva 40 minutos, quebra sem motivo e que a equipe aprende a ignorar. É como quase todo projeto acaba quando ninguém decide a proporção.

### ⌛ Ampulheta
Muitos unitários, muitos E2E, quase nenhum de integração. Os bugs de contrato entre camadas passam direto.

---

## 🎯 O que testar quando o tempo é curto

| Prioridade | O quê |
|---|---|
| 1 | **Lógica de negócio central** — o algoritmo que justifica o projeto |
| 2 | **Casos-limite** — zero, vazio, nulo, negativo, máximo |
| 3 | **Bugs já corridos** — teste de regressão impede o retorno |
| 4 | **Fronteiras de contrato** — o formato que outra parte consome |
| 5 | Caminho feliz E2E |

> [!tip] Todo bug corrigido merece um teste antes da correção
> Escreva o teste que **falha** reproduzindo o bug, depois corrija até ele passar. Isso garante que o bug foi de fato entendido — e que ele não volta silenciosamente em um refactor futuro. Ver [[tst-tdd|🔴 TDD]].

---

## 📊 Testes em trabalho acadêmico

Uma suíte de testes não é só engenharia — é **evidência de corretude** em uma monografia.

> "A implementação do cálculo foi verificada por 12 testes automatizados, cobrindo propriedades matemáticas (identidade, ortogonalidade, invariância a escala), casos-limite (vetor nulo) e reprodutibilidade (mesma entrada produz mesma saída em execuções sucessivas)."

Isso responde antecipadamente a "como vocês sabem que o cálculo está certo?" — uma das perguntas mais prováveis de banca. Ver [[met-defesa-banca|🎤 Defesa de banca]].

---

## 📚 Referências

- **Cohn, M. (2009)** — *Succeeding with Agile* — origem da pirâmide
- **Fowler, M.** — *TestPyramid* e *Practical Test Pyramid*, martinfowler.com
- **Meszaros, G. (2007)** — *xUnit Test Patterns*

---

## 🔗 Conceitos relacionados

- [[tst-tdd|🔴 TDD]] · [[tst-mocks-e-dubles|🎭 Mocks e dublês]]
- [[tst-cobertura|📊 Cobertura de testes]] · [[tst-testes-django|🐍 Testes em Django]]
- [[CI-CD|CI/CD]]

## Veja também

- [[Conceitos|🧠 Conceitos]]

---

**Status:** ✅ Completo
