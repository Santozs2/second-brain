---
type: concept
area: Conceitos
status: estavel
difficulty: beginner
id: ia-tokens-e-custo
category: Inteligência Artificial
tags:
  - ia
  - llm
  - concept
created: 2026-08-24
updated: 2026-08-24
---
# 💰 Tokens, Custo e Latência

> Toda decisão de arquitetura com LLM tem um preço em tokens e um preço em segundos. Estimar antes de construir evita surpresa e sustenta escolha de projeto.

---

## 🔤 Como o token vira conta

Provedores cobram por token, com preços **diferentes** para entrada e saída — saída costuma custar de 3 a 5 vezes mais.

```
custo = (tokens_entrada × preço_entrada) + (tokens_saída × preço_saída)
```

**Estimativa rápida para português:** `tokens ≈ caracteres / 3,5`

```
Prompt de 2.000 caracteres  ≈  570 tokens de entrada
Resposta de 800 caracteres  ≈  230 tokens de saída
```

> [!note] Português custa mais que inglês pelo mesmo conteúdo
> Tokenizadores foram otimizados majoritariamente sobre corpus em inglês. O mesmo texto em português costuma gastar de 20% a 40% mais tokens. Em estimativa de orçamento, considere isso.

---

## 🧮 Modelo de estimativa

```python
def estimar_custo(
    n_requisicoes: int,
    chars_entrada: int,
    chars_saida: int,
    preco_entrada_por_milhao: float,
    preco_saida_por_milhao: float,
) -> dict:
    t_in = (chars_entrada / 3.5) * n_requisicoes
    t_out = (chars_saida / 3.5) * n_requisicoes
    custo = (t_in / 1_000_000) * preco_entrada_por_milhao \
          + (t_out / 1_000_000) * preco_saida_por_milhao
    return {
        "tokens_entrada": round(t_in),
        "tokens_saida": round(t_out),
        "custo_usd": round(custo, 4),
    }
```

Faça a conta em **três cenários** — piloto, uso esperado e pico — e registre os três. Uma tabela de custo estimado é o tipo de evidência que transforma "escolhemos o provedor X" em decisão justificada.

---

## 🎚️ As alavancas de redução

| Alavanca | Efeito | Custo colateral |
|---|---|---|
| **Cache de respostas** | Elimina chamadas repetidas | Precisa de chave de cache estável |
| **Prompt mais curto** | Reduz entrada | Pode perder instrução necessária |
| **`max_tokens` menor** | Reduz saída (a parte cara) | Risco de corte no meio |
| **Modelo menor** | Reduz preço unitário | Perde qualidade |
| **Batelada** | Desconto em processamento assíncrono | Não serve para tempo real |
| **Não chamar** | Redução de 100% | — |

> [!success] A maior economia é arquitetural, não de configuração
> Se o cálculo é determinístico e a entrada se repete, **cachear pela entrada canônica** elimina a maior parte das chamadas. Em um sistema onde muitas pessoas produzem o mesmo perfil, a segunda pessoa com o mesmo perfil não precisa de chamada nenhuma.

```python
def chave_cache(payload: dict) -> str:
    """Chave estável: mesmo conteúdo → mesma chave, independente da ordem."""
    import hashlib, json
    canonico = json.dumps(payload, sort_keys=True, ensure_ascii=False)
    return hashlib.sha256(canonico.encode()).hexdigest()
```

---

## ⏱️ Latência

O custo em segundos costuma importar mais que o custo em reais.

| Fator | Impacto |
|---|---|
| **Tokens de saída** | O dominante — geração é sequencial, token a token |
| Tamanho do modelo | Modelo maior, mais lento por token |
| Tokens de entrada | Impacto menor (processados em paralelo) |
| Rede e região | Dezenas a centenas de ms |
| Fila do provedor | Variável, imprevisível |

**Faixa realista:** 1 a 5 segundos para uma resposta de algumas centenas de tokens.

### Padrões para lidar com a espera

| Padrão | Quando |
|---|---|
| **Streaming** | Interface conversacional — o texto aparece enquanto gera |
| **Processamento assíncrono** | Resultado não precisa ser imediato |
| **Timeout + fallback** | O sistema não pode travar esperando |
| **Resultado progressivo** | Mostrar o resultado calculado, enriquecer depois |

```python
TIMEOUT_S = 10

def entregar_com_prazo(payload, provider):
    try:
        return provider.gerar(payload, timeout=TIMEOUT_S), False
    except (TimeoutError, ProviderError):
        return resposta_deterministica(payload), True   # nunca trava
```

> [!important] Timeout com fallback é requisito, não otimização
> Um sistema que depende de uma API externa para responder **fica indisponível quando a API fica**. Com fallback determinístico, a indisponibilidade externa vira uma degradação de qualidade, não uma queda. Ver [[ia-alucinacao-e-grounding|🎭 fallback]].

---

## 📊 O que registrar em cada chamada

```python
@dataclass
class MetricasChamada:
    modelo: str
    prompt_version: str
    tokens_entrada: int
    tokens_saida: int
    latencia_ms: int
    custo_estimado: float
    usou_cache: bool
    usou_fallback: bool
    timestamp: datetime
```

Sem esses campos persistidos, não há como responder "quanto custou", "está mais lento?" ou "com que frequência a IA falha" — nem em produção, nem em capítulo de resultados.

---

## 🔗 Conceitos relacionados

- [[ia-llm-fundamentos|🧠 LLM — Fundamentos]] · [[ia-engenharia-de-prompt|📝 Engenharia de prompt]]
- [[Cache|Cache]] · [[ia-avaliacao-de-llm|📐 Avaliação de LLM]]

## Veja também

- [[Conceitos|🧠 Conceitos]]

---

**Status:** ✅ Completo
