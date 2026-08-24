---
type: concept
area: Conceitos
status: estavel
difficulty: intermediate
id: tst-mocks-e-dubles
category: Testes
tags:
  - testes
  - concept
  - qualidade
created: 2026-08-24
updated: 2026-08-24
---
# 🎭 Mocks e Dublês de Teste

> Substituir uma dependência real por uma controlada, para testar sem depender de rede, banco, relógio ou sorte.

---

## 🎬 Os cinco tipos (taxonomia de Meszaros)

| Dublê | O que faz | Uso |
|---|---|---|
| **Dummy** | Só preenche um parâmetro; nunca é usado | Satisfazer assinatura |
| **Stub** | Devolve resposta fixa | Controlar a entrada do teste |
| **Spy** | Stub que registra como foi chamado | Verificar interação |
| **Mock** | Espera chamadas específicas e falha se não vierem | Verificar protocolo |
| **Fake** | Implementação real, porém simplificada | Substituto funcional |

> [!important] Stub controla a entrada; mock verifica a saída
> Use **stub** quando o teste precisa que a dependência devolva algo. Use **mock** quando o comportamento que você está testando **é** a chamada em si (ex.: "o e-mail foi enviado?"). Confundir os dois produz testes que verificam implementação em vez de comportamento.

---

## ⭐ Fake: o mais subestimado

Um *fake* é uma implementação real de uma interface, com atalhos. Diferente de um stub, ele **funciona**.

```python
from typing import Protocol

class LLMProvider(Protocol):
    def gerar(self, payload: dict, timeout: int) -> dict: ...


class FakeProvider:
    """Implementação offline: comportamento determinístico, sem rede."""

    def __init__(self, falhar: bool = False, latencia_ms: int = 0):
        self.falhar = falhar
        self.latencia_ms = latencia_ms
        self.chamadas: list[dict] = []

    def gerar(self, payload: dict, timeout: int = 10) -> dict:
        self.chamadas.append(payload)
        if self.falhar:
            raise TimeoutError("falha simulada")
        ids = [c["id"] for c in payload["candidatos"]]
        return {
            "ordem": ids,
            "texto": f"Recomendação simulada para {len(ids)} candidatos.",
        }
```

> [!success] Um fake bem-feito desbloqueia o desenvolvimento inteiro
> Com `FakeProvider`, toda a lógica de orquestração — validação, cache, timeout, fallback, persistência de métricas — pode ser construída e testada **sem chave de API, sem internet e sem custo**. Quando a credencial real chegar, só a implementação concreta muda; nada mais. Isso remove uma dependência externa do caminho crítico do projeto.

---

## 🐍 `unittest.mock` na prática

```python
from unittest.mock import patch, MagicMock

# Stub — controlar o retorno
@patch("quiz.delivery.buscar_provider")
def test_usa_resposta_do_provider(self, mock_buscar):
    mock_buscar.return_value.gerar.return_value = {"ordem": [1, 2], "texto": "ok"}
    resultado, fallback = entregar(self.payload)
    self.assertFalse(fallback)

# Simular falha
@patch("quiz.delivery.buscar_provider")
def test_cai_para_fallback_em_timeout(self, mock_buscar):
    mock_buscar.return_value.gerar.side_effect = TimeoutError()
    resultado, fallback = entregar(self.payload)
    self.assertTrue(fallback)
    self.assertEqual(len(resultado["ordem"]), 5)   # engine respondeu

# Spy — verificar que NÃO foi chamado
@patch("quiz.delivery.buscar_provider")
def test_cache_evita_segunda_chamada(self, mock_buscar):
    entregar(self.payload)
    entregar(self.payload)
    self.assertEqual(mock_buscar.return_value.gerar.call_count, 1)
```

> [!warning] Faça patch onde o objeto é USADO, não onde é definido
> `@patch("quiz.delivery.buscar_provider")`, não `@patch("quiz.llm.buscar_provider")`. O patch substitui a referência no módulo que importou — se você aplicar no módulo de origem, o código já importou a referência antiga e o mock não tem efeito. Este é o erro nº 1 com `unittest.mock`, e ele falha de forma silenciosa.

---

## ⏰ Controlar tempo e aleatoriedade

Dependências invisíveis que tornam testes instáveis:

```python
# Tempo
from unittest.mock import patch
from datetime import datetime

@patch("meu_modulo.datetime")
def test_janela_de_24h(self, mock_dt):
    mock_dt.now.return_value = datetime(2026, 8, 24, 12, 0)
    self.assertTrue(dentro_da_janela(self.conversa))

# Aleatoriedade — fixe a semente
import random
random.seed(42)
```

> [!tip] Injete o relógio em vez de mocká-lo
> Uma função que recebe `agora: datetime = None` e usa `agora or datetime.now()` é testável sem patch nenhum. **Dependência injetada é mais fácil de testar que dependência mockada** — e o teste fica legível.

---

## 🚨 Quando mock demais vira problema

| Sintoma | Diagnóstico |
|---|---|
| O teste quebra a cada refactor, sem mudança de comportamento | Testa implementação, não comportamento |
| Mais linhas de setup de mock que de asserção | O código sob teste tem dependências demais |
| Todos os colaboradores mockados | O teste prova que o mock funciona, não o código |
| Suíte verde, produção quebrada | Os mocks não refletem a realidade |

> [!important] Mock excessivo é sintoma de acoplamento, não causa
> Quando um teste precisa de seis mocks, a mensagem é sobre o **projeto do código**, não sobre o teste. Extrair a lógica pura para funções sem dependência elimina a maior parte dos mocks — e é a mesma mudança que torna o núcleo testável e defensável. Ver [[arq-camadas|🏛️ Arquitetura em camadas]] e [[arq-solid|🧱 SOLID]].

---

## 📚 Referências

- **Meszaros, G. (2007)** — *xUnit Test Patterns* — a taxonomia dos dublês
- **Fowler, M.** — *Mocks Aren't Stubs*, martinfowler.com
- **Freeman & Pryce (2009)** — *Growing Object-Oriented Software, Guided by Tests*

---

## 🔗 Conceitos relacionados

- [[tst-piramide-de-testes|🔺 Pirâmide de testes]] · [[tst-tdd|🔴 TDD]]
- [[tst-testes-django|🐍 Testes em Django]] · [[arq-solid|🧱 SOLID]]

## Veja também

- [[Conceitos|🧠 Conceitos]]

---

**Status:** ✅ Completo
