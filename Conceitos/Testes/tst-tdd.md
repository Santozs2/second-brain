---
type: concept
area: Conceitos
status: estavel
difficulty: intermediate
id: tst-tdd
category: Testes
tags:
  - testes
  - concept
  - qualidade
created: 2026-08-24
updated: 2026-08-24
---
# 🔴 TDD — Test-Driven Development

> Escrever o teste antes do código. O ganho principal não é a suíte de testes — é que você projeta a interface a partir de quem vai usá-la.

---

## 🔄 O ciclo Red-Green-Refactor

```
  🔴 RED        Escreve um teste que falha
      ↓         (falhar é obrigatório — prova que o teste testa algo)
  🟢 GREEN      Escreve o mínimo de código para passar
      ↓         (feio é permitido nesta fase)
  🔵 REFACTOR   Melhora o código com o teste protegendo
      ↓
   repete
```

> [!important] Ver o teste falhar é uma etapa, não uma formalidade
> Um teste que passa antes de o código existir está testando a coisa errada — ou não está testando nada. **A falha inicial é a prova de que o teste é capaz de detectar o problema.** Pular esta etapa produz suítes verdes que não protegem nada.

---

## 💡 O ganho real: projeto de interface

Ao escrever o teste primeiro, você é forçado a usar a função antes de escrevê-la — e a experiência de uso aparece antes da implementação.

```python
# Escrevendo o teste primeiro, você decide a assinatura como CLIENTE:
def test_recomendacao_devolve_top_n_com_explicacao(self):
    resultado = recommend(perfil=[5, 0, 2], cursos=self.cursos, n=3)

    self.assertEqual(len(resultado), 3)
    self.assertGreaterEqual(resultado[0].score, resultado[1].score)
    self.assertTrue(resultado[0].explanation)
```

Antes de escrever uma linha de implementação, três decisões de projeto já foram tomadas: o retorno é ordenado, cada item tem `score` e `explanation`, e `n` é parâmetro. Se a assinatura ficasse desconfortável de usar no teste, ela seria desconfortável de usar em produção.

---

## 🧱 O ciclo duplo

Para funcionalidades maiores, TDD opera em dois níveis:

```
┌── Teste de aceitação (falhando) ─────────────────┐
│                                                   │
│   ┌── unidade → verde → refatora ──┐              │
│   └────────── repete ──────────────┘              │
│                                                   │
└── aceitação fica verde quando a feature termina ──┘
```

---

## ✅ Onde TDD compensa muito

| Contexto | Por quê |
|---|---|
| **Lógica algorítmica** | Entrada e saída bem definidas; propriedades verificáveis |
| **Regras de negócio** | Muitos casos-limite a cobrir |
| **Correção de bug** | O teste reproduz o bug antes da correção |
| **Refatoração** | A suíte é a rede de segurança |
| **Cálculo matemático** | Você consegue verificar o esperado à mão |

## ❌ Onde TDD atrapalha

| Contexto | Por quê |
|---|---|
| **Exploração / protótipo** | Você ainda não sabe qual é a interface |
| **UI visual** | O critério é "está bonito", não é assertivo |
| **Integração externa** | O comportamento real só aparece contra o serviço |
| **Código descartável** | Custo sem retorno |

> [!tip] TDD não é tudo ou nada
> A prática mais produtiva na maioria dos projetos é **TDD no núcleo, testes depois nas bordas**. O algoritmo central nasce por TDD; o CRUD de admin recebe teste quando estabiliza. Adotar TDD como dogma em toda linha de código é o caminho mais rápido para abandoná-lo por completo.

---

## 🧪 Testes baseados em propriedade

Uma evolução útil para código matemático: em vez de casos específicos, declare **propriedades que devem valer sempre**, e deixe a ferramenta gerar centenas de entradas.

```python
from hypothesis import given, strategies as st

@given(st.lists(st.floats(min_value=0, max_value=100), min_size=3, max_size=3))
def test_cosseno_consigo_mesmo_e_um(v):
    """Propriedade: sim(v, v) == 1 para qualquer v não-nulo."""
    if any(v):
        assert math.isclose(cosine_similarity(v, v), 1.0, rel_tol=1e-9)

@given(
    st.lists(st.floats(min_value=0, max_value=100), min_size=3, max_size=3),
    st.lists(st.floats(min_value=0, max_value=100), min_size=3, max_size=3),
)
def test_cosseno_e_simetrico(a, b):
    """Propriedade: sim(a,b) == sim(b,a) sempre."""
    assert cosine_similarity(a, b) == cosine_similarity(b, a)
```

> [!success] Teste de propriedade encontra o caso-limite que você não imaginou
> Ferramentas como **Hypothesis** (Python) geram entradas adversariais e reduzem automaticamente o caso que falhou ao menor exemplo possível. Para código matemático, isso costuma revelar em minutos bugs que casos escritos à mão não pegariam — e as propriedades verificadas rendem um parágrafo forte de metodologia.

---

## 📚 Referências

- **Beck, K. (2002)** — *Test-Driven Development: By Example* — o livro fundador
- **Freeman & Pryce (2009)** — *Growing Object-Oriented Software, Guided by Tests* — o ciclo duplo
- **MacIver, D.** — documentação do *Hypothesis*

---

## 🔗 Conceitos relacionados

- [[tst-piramide-de-testes|🔺 Pirâmide de testes]] · [[tst-mocks-e-dubles|🎭 Mocks e dublês]]
- [[tst-cobertura|📊 Cobertura]] · [[arq-solid|🧱 SOLID]]

## Veja também

- [[Conceitos|🧠 Conceitos]]

---

**Status:** ✅ Completo
