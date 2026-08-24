---
type: concept
area: Conceitos
status: estavel
difficulty: intermediate
id: ia-alucinacao-e-grounding
category: Inteligência Artificial
tags:
  - ia
  - llm
  - concept
created: 2026-08-24
updated: 2026-08-24
---
# 🎭 Alucinação e Grounding

> O modelo produz texto plausível, não texto verdadeiro. Alucinação não é defeito a ser corrigido — é propriedade a ser contida por arquitetura.

---

## 📖 O que é alucinação

Geração de conteúdo fluente, confiante e **factualmente incorreto ou não sustentado** pela informação disponível.

| Tipo | Descrição |
|---|---|
| **Intrínseca** | Contradiz a informação que foi fornecida |
| **Extrínseca** | Afirma algo que a informação fornecida não sustenta |
| **Factual** | Contradiz o mundo real |
| **De fidelidade** | Desvia da tarefa pedida |

```
Enviado ao modelo: lista com 5 cursos
Resposta: recomenda um 6º curso que não estava na lista
          ← alucinação extrínseca, e a mais perigosa em recomendação
```

---

## 🔍 Por que acontece

Não é bug. Decorre diretamente do objetivo de treino:

1. O modelo otimiza **probabilidade do próximo token**, não veracidade
2. Não existe representação interna separada de "eu sei" versus "eu suponho"
3. O treino premia respostas completas — abster-se é penalizado
4. Interpolação entre padrões vistos gera combinações novas que soam certas

> [!important] O modelo é igualmente fluente quando acerta e quando erra
> Não há sinal linguístico de incerteza. Um nome de curso inventado sai com a mesma confiança de um nome real. **Por isso a verificação nunca pode depender de o texto "parecer certo"** — precisa ser programática.

---

## 🛡️ Grounding: ancorar a saída em fonte verificável

**Grounding** é a prática de obrigar o modelo a se apoiar em dados fornecidos e verificáveis, em vez do que ele "lembra".

### Estratégia 1 — Fornecer o contexto (RAG)
Injetar a informação relevante no prompt e instruir a responder apenas com base nela. Ver [[ia-rag|📚 RAG]].

### Estratégia 2 — Restringir o espaço de saída ⭐

A mitigação mais forte. O modelo não escolhe *o que* dizer; ele escolhe **dentro de um conjunto fechado** definido em código.

```
                CÓDIGO (determinístico)          LLM
                       ↓                          ↓
catálogo → seleção dos N candidatos → reordenar/redigir → validar
                       ↑                                      ↓
                       └──── item fora da lista? descarta ─────┘
```

> [!success] Se o modelo só pode reordenar, ele não pode inventar
> Esta é a diferença entre "pedimos ao modelo que não invente" e "o modelo não tem como inventar". A primeira é uma instrução que pode falhar; a segunda é uma propriedade do sistema. Toda decisão que importa deve estar do lado do código. Ver [[rec-sistemas-hibridos|🔀 cascata]].

### Estratégia 3 — Validação programática da saída

```python
def validar_saida(resposta: dict, candidatos_validos: list[int]) -> bool:
    """Rejeita qualquer saída que cite item fora do conjunto enviado."""
    ids = resposta.get("ordem", [])
    if not ids:
        return False
    if set(ids) - set(candidatos_validos):        # inventou item
        return False
    if len(ids) != len(set(ids)):                 # repetiu item
        return False
    return True
```

### Estratégia 4 — Fallback determinístico

Quando a validação falha, o sistema **não** tenta de novo indefinidamente nem entrega o texto suspeito: cai para uma saída calculada em código.

```python
def entregar(payload, provider, tentativas=2):
    for _ in range(tentativas):
        try:
            r = provider.gerar(payload, timeout=10)
            if validar_saida(r, payload["ids_validos"]):
                return r, False                    # ok, sem fallback
        except (TimeoutError, ProviderError):
            pass
    return resposta_deterministica(payload), True  # fallback acionado
```

> [!tip] Registre `usou_fallback` como métrica de produto
> A taxa de fallback é um indicador de saúde do sistema: mede quantas vezes a camada de IA falhou ou produziu saída inválida. Ela também sustenta uma afirmação forte em defesa: *"o sistema responde corretamente mesmo com a API indisponível"* — verificável desligando a chave.

---

## 📊 Como medir alucinação

| Método | Como funciona |
|---|---|
| **Verificação de restrição** | Todo item citado pertence ao conjunto enviado? (automático, barato) |
| **Verificação factual** | Cada afirmação é sustentada pelo contexto? (custoso) |
| **Autoconsistência** | Gerar N vezes; divergência alta sinaliza invenção |
| **LLM como juiz** | Outro modelo avalia a fidelidade → [[ia-avaliacao-de-llm\|📐 nota dedicada]] |
| **Avaliação humana** | Padrão-ouro; caro e não escala |

A **verificação de restrição** é a que mais rende: é automática, roda em toda requisição e cobre o modo de falha mais grave.

---

## ⚠️ O que não resolve

- ❌ **Pedir "não invente" no prompt** — reduz, não elimina
- ❌ **`temperature=0`** — torna a alucinação mais consistente, não menos provável
- ❌ **Modelo maior** — alucina menos, mas alucina
- ❌ **Pedir que o modelo avalie a própria certeza** — a autoavaliação é gerada pelo mesmo processo

---

## 📚 Referências

- **Ji et al. (2023)** — *Survey of Hallucination in Natural Language Generation*, ACM Computing Surveys — a taxonomia de referência
- **Lewis et al. (2020)** — *Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks*
- **Maynez et al. (2020)** — *On Faithfulness and Factuality in Abstractive Summarization*
- **Huang et al. (2023)** — *A Survey on Hallucination in Large Language Models*

---

## 🔗 Conceitos relacionados

- [[ia-llm-fundamentos|🧠 LLM — Fundamentos]] · [[ia-rag|📚 RAG]]
- [[ia-engenharia-de-prompt|📝 Engenharia de prompt]] · [[rec-explicabilidade|💡 Explicabilidade]]
- [[ia-avaliacao-de-llm|📐 Avaliação de LLM]]

## Veja também

- [[Conceitos|🧠 Conceitos]]

---

**Status:** ✅ Completo
