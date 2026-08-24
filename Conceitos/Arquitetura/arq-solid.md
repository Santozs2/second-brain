---
type: concept
area: Conceitos
status: estavel
difficulty: intermediate
id: arq-solid
category: Arquitetura
tags:
  - arquitetura
  - concept
  - qualidade
created: 2026-08-24
updated: 2026-08-24
---
# 🧱 SOLID

> Cinco princípios de projeto orientado a objetos. Todos atacam a mesma coisa: **acoplamento** — o grau em que mudar uma parte obriga a mudar outra.

---

## 1️⃣ S — Responsabilidade Única (SRP)

> Uma classe deve ter apenas um motivo para mudar.

O critério não é "faz uma coisa só" — é **"responde a um só interessado"**. Uma classe que formata relatório e calcula imposto muda quando o time de design muda e quando a lei muda. Dois motivos, duas classes.

```python
# ❌ Três motivos para mudar
class Recomendador:
    def calcular(self, perfil): ...
    def salvar_no_banco(self, resultado): ...
    def enviar_email(self, resultado): ...

# ✅ Um motivo cada
class Engine:              # muda se a matemática mudar
    def calcular(self, perfil): ...

class RepositorioResultado: # muda se o armazenamento mudar
    def salvar(self, resultado): ...

class NotificadorEmail:    # muda se a comunicação mudar
    def enviar(self, resultado): ...
```

---

## 2️⃣ O — Aberto/Fechado (OCP)

> Aberto para extensão, fechado para modificação.

Adicionar comportamento novo não deveria exigir editar código que já funciona e já foi testado.

```python
# ❌ Cada provedor novo exige editar esta função
def gerar(payload, tipo):
    if tipo == "gemini":
        return chamar_gemini(payload)
    elif tipo == "openai":
        return chamar_openai(payload)
    # ...e assim por diante, para sempre

# ✅ Provedor novo = classe nova; nada aqui muda
class LLMProvider(Protocol):
    def gerar(self, payload: dict) -> dict: ...

def entregar(payload: dict, provider: LLMProvider) -> dict:
    return provider.gerar(payload)
```

---

## 3️⃣ L — Substituição de Liskov (LSP)

> Um subtipo deve poder substituir o tipo base sem quebrar o programa.

Se uma implementação de `LLMProvider` levanta exceção onde as outras retornam valor, quem consome precisa saber qual implementação está usando — e a abstração deixou de servir.

```python
# ❌ Viola LSP: quebra o contrato do tipo base
class ProviderQuebrado:
    def gerar(self, payload):
        raise NotImplementedError("use outro método")

# ✅ Respeita: mesmo contrato, comportamento diferente
class FakeProvider:
    def gerar(self, payload):
        return {"ordem": [c["id"] for c in payload["candidatos"]], "texto": "..."}
```

**Sinais de violação:** o subtipo lança exceção nova, exige pré-condição mais forte, ou devolve menos que o prometido.

---

## 4️⃣ I — Segregação de Interface (ISP)

> Nenhum cliente deve depender de métodos que não usa.

```python
# ❌ Quem só lê é obrigado a conhecer escrita
class Repositorio(Protocol):
    def buscar(self, id): ...
    def salvar(self, obj): ...
    def deletar(self, id): ...
    def exportar_csv(self): ...

# ✅ Interfaces pequenas, compostas quando preciso
class Leitor(Protocol):
    def buscar(self, id): ...

class Escritor(Protocol):
    def salvar(self, obj): ...
```

Interfaces menores geram menos mocks nos testes e menos motivos de recompilação.

---

## 5️⃣ D — Inversão de Dependência (DIP)

> Módulos de alto nível não devem depender de módulos de baixo nível. Ambos dependem de abstrações.

O princípio mais consequente dos cinco.

```python
# ❌ A regra de negócio depende do detalhe concreto
from quiz.llm.gemini import GeminiClient

def entregar(payload):
    return GeminiClient().gerar(payload)   # impossível testar sem rede

# ✅ A regra depende da abstração; o detalhe é injetado
def entregar(payload: dict, provider: LLMProvider) -> dict:
    return provider.gerar(payload)

# produção
entregar(payload, GeminiProvider(api_key=...))
# teste — sem rede, sem chave, sem custo
entregar(payload, FakeProvider())
```

> [!success] DIP é o que torna um sistema testável e o que protege contra dependência externa
> Quando a regra de negócio depende de um protocolo em vez de um cliente HTTP concreto, três coisas ficam possíveis de graça: **testar offline**, **trocar de provedor por configuração** e **implementar fallback** sem tocar na lógica. É a mesma decisão vista de três ângulos. Ver [[tst-mocks-e-dubles|🎭 Mocks e dublês]].

---

## ⚠️ SOLID não é lei

| Risco | Sintoma |
|---|---|
| **Abstração prematura** | Interface com uma única implementação, criada "por precaução" |
| **Explosão de arquivos** | Vinte arquivos de dez linhas para uma funcionalidade |
| **Indireção excessiva** | Cinco saltos para achar onde o trabalho acontece |

> [!tip] Aplique SOLID onde a mudança é esperada
> O ponto do sistema que você **sabe** que vai variar — o provedor de LLM, a forma de pagamento, o canal de notificação — merece abstração. O CRUD de cadastro que nunca mudou em três anos não merece. Abstrair tudo custa mais legibilidade do que ganha em flexibilidade.

---

## 📚 Referências

- **Martin, R. C. (2000)** — *Design Principles and Design Patterns* — onde o acrônimo aparece
- **Martin, R. C. (2017)** — *Clean Architecture*
- **Metz, S. (2012)** — *Practical Object-Oriented Design in Ruby* — a explicação mais acessível

---

## 🔗 Conceitos relacionados

- [[arq-camadas|🏛️ Arquitetura em camadas]] · [[arq-design-patterns|🎨 Padrões de projeto]]
- [[arq-clean-architecture|🧅 Clean Architecture]] · [[tst-mocks-e-dubles|🎭 Mocks e dublês]]
- [[MVC|MVC]]

## Veja também

- [[Conceitos|🧠 Conceitos]]

---

**Status:** ✅ Completo
