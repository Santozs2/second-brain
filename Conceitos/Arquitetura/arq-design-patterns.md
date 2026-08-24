---
type: concept
area: Conceitos
status: estavel
difficulty: intermediate
id: arq-design-patterns
category: Arquitetura
tags:
  - arquitetura
  - concept
created: 2026-08-24
updated: 2026-08-24
---
# 🎨 Padrões de Projeto

> Soluções nomeadas para problemas recorrentes. O valor principal não é o código — é o **vocabulário compartilhado**: dizer "isso é um Strategy" comunica em duas palavras o que levaria um parágrafo.

---

## 🗂️ As três famílias (GoF)

| Família | Resolve |
|---|---|
| **Criacionais** | Como objetos são criados |
| **Estruturais** | Como objetos se compõem |
| **Comportamentais** | Como objetos colaboram |

---

## 🏭 Criacionais

### Factory
Centraliza a decisão de qual implementação instanciar.

```python
def criar_provider(nome: str) -> LLMProvider:
    """Um só lugar decide; o resto do sistema não sabe qual é."""
    providers = {
        "gemini": GeminiProvider,
        "fake": FakeProvider,
    }
    if nome not in providers:
        raise ValueError(f"provider desconhecido: {nome}")
    return providers[nome]()

provider = criar_provider(os.getenv("LLM_PROVIDER", "fake"))
```

> [!tip] Factory + variável de ambiente é o que torna um componente plugável
> Com esse par, trocar de provedor vira uma linha no `.env` — e desenvolver offline vira o padrão, não a exceção.

### Singleton
Uma única instância global. **Use com parcimônia** — dificulta teste e esconde dependência. Em Python, um módulo já é um singleton.

### Builder
Construção passo a passo de objeto complexo. Útil para montar payloads longos.

---

## 🧩 Estruturais

### Adapter
Traduz uma interface para outra. É o padrão que permite trocar de fornecedor.

```python
class GeminiAdapter:
    """Adapta a API do Gemini ao protocolo interno LLMProvider."""
    def __init__(self, client): self._client = client

    def gerar(self, payload: dict, timeout: int = 10) -> dict:
        bruto = self._client.generate_content(
            self._montar_prompt(payload), request_options={"timeout": timeout}
        )
        return self._normalizar(bruto)     # o resto do sistema não muda
```

### Facade
Uma interface simples sobre um subsistema complexo.

### Decorator
Adiciona comportamento sem alterar o objeto. Em Python, é sintaxe nativa.

```python
@cache_resultado
@com_timeout(10)
@registrar_metricas
def entregar(payload, provider): ...
```

### Proxy
Um substituto que controla acesso — cache, lazy loading, permissão.

---

## 🔄 Comportamentais

### Strategy ⭐
Encapsula algoritmos intercambiáveis. **O padrão mais útil em sistema com cálculo.**

```python
class Similaridade(Protocol):
    def __call__(self, a: list[float], b: list[float]) -> float: ...

def rank(perfil, itens, metrica: Similaridade = cosine_similarity):
    return sorted(itens, key=lambda i: metrica(perfil, i.vetor), reverse=True)

# Trocar a métrica é passar outra função — e vira experimento comparativo
rank(perfil, itens, metrica=euclidean_similarity)
```

> [!success] Strategy transforma uma decisão de projeto em variável de experimento
> Se a métrica de similaridade é um parâmetro, comparar cosseno contra euclidiana custa uma linha — e vira um resultado publicável. Padrão de projeto e desenho experimental coincidem aqui. Ver [[rec-metricas-similaridade|📏 Métricas de similaridade]].

### Observer
Notifica interessados quando algo acontece. Os *signals* do Django são Observer.

### Template Method
Define o esqueleto do algoritmo, deixando passos para subclasses. As *generic views* do Django são Template Method.

### Chain of Responsibility
Uma cadeia de manipuladores; cada um trata ou repassa. O middleware do Django é exatamente isso.

### Repository
Abstrai o acesso a dados atrás de uma interface de coleção.

```python
class RepositorioCursos(Protocol):
    def todos_com_vetores(self) -> list[CourseVec]: ...

class RepositorioDjango:
    def todos_com_vetores(self) -> list[CourseVec]:
        return [CourseVec(c.id, montar_vetor(c))
                for c in Course.objects.prefetch_related("pesos")]

class RepositorioMemoria:
    """Para teste: sem banco."""
    def __init__(self, cursos): self._cursos = cursos
    def todos_com_vetores(self): return self._cursos
```

---

## ⚠️ O antipadrão: caçar padrões

| Sintoma | Diagnóstico |
|---|---|
| `AbstractFactoryBuilderStrategy` | O padrão virou o objetivo |
| Interface com uma implementação eterna | Abstração sem motivo |
| Cinco arquivos para uma soma | Indireção que só atrapalha |

> [!warning] Padrão é resposta a um problema, não enfeite
> Aplicar Strategy onde existe **um** algoritmo que nunca vai mudar adiciona indireção sem entregar flexibilidade. A pergunta antes de aplicar qualquer padrão: *"qual mudança futura isto torna barata, e essa mudança é realista?"*. Sem resposta concreta, não aplique.

---

## 🐍 Padrões em Python são mais simples

Muitos padrões do GoF existem para contornar limitações de linguagens sem funções de primeira classe.

| Padrão GoF | Em Python |
|---|---|
| Strategy | Passar uma função |
| Command | Passar uma função |
| Factory | Um dicionário ou uma função |
| Singleton | Um módulo |
| Decorator | Sintaxe `@` nativa |
| Iterator | Protocolo nativo / geradores |

---

## 📚 Referências

- **Gamma, Helm, Johnson & Vlissides (1994)** — *Design Patterns* — o livro "Gang of Four"
- **Freeman & Robson** — *Head First Design Patterns* — o mais didático
- **Ramalho, L. (2022)** — *Fluent Python*, 2ª ed. — padrões idiomáticos em Python

---

## 🔗 Conceitos relacionados

- [[arq-solid|🧱 SOLID]] · [[arq-camadas|🏛️ Arquitetura em camadas]]
- [[MVC|MVC]] · [[arq-clean-architecture|🧅 Clean Architecture]]

## Veja também

- [[Conceitos|🧠 Conceitos]]

---

**Status:** ✅ Completo
