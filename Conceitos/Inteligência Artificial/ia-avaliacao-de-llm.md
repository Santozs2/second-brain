---
type: concept
area: Conceitos
status: estavel
difficulty: advanced
id: ia-avaliacao-de-llm
category: Inteligência Artificial
tags:
  - ia
  - llm
  - concept
  - metricas
created: 2026-08-24
updated: 2026-08-24
---
# 📐 Avaliação de LLM

> Como provar que a saída de um modelo é boa, quando a saída é texto livre, não-determinística e sem gabarito único.

---

## 📖 O problema central

Software tradicional se testa por igualdade: entrada X → saída esperada Y. Com LLM isso quebra em três pontos:

1. **Não existe saída única correta** — muitas redações diferentes são igualmente boas
2. **Não é determinístico** — a mesma entrada pode gerar saídas diferentes
3. **A qualidade é multidimensional** — correção, tom, formato, concisão e fidelidade são coisas distintas

Portanto: **não se testa igualdade; testa-se propriedade.**

---

## 1️⃣ Verificações determinísticas (as mais valiosas)

Nem tudo precisa de julgamento. Estas rodam em toda requisição, custam quase nada e pegam os piores modos de falha.

```python
def verificacoes_estruturais(resposta: dict, candidatos: list[int]) -> list[str]:
    """Retorna a lista de violações. Lista vazia = passou."""
    falhas = []
    if not isinstance(resposta.get("ordem"), list):
        falhas.append("campo 'ordem' ausente ou de tipo errado")
    ids = resposta.get("ordem", [])
    if set(ids) - set(candidatos):
        falhas.append("citou item fora do conjunto enviado")
    if len(ids) != len(set(ids)):
        falhas.append("item repetido")
    texto = resposta.get("texto", "")
    if not (50 <= len(texto) <= 800):
        falhas.append(f"comprimento fora da faixa: {len(texto)}")
    return falhas
```

| Verificação | Pega |
|---|---|
| JSON válido e esquema correto | Formato quebrado |
| Itens ∈ conjunto enviado | [[ia-alucinacao-e-grounding\|Alucinação]] extrínseca |
| Sem repetição | Degeneração |
| Comprimento na faixa | Resposta truncada ou prolixa |
| Ausência de termos proibidos | Vazamento de instrução |

> [!success] Comece por aqui, sempre
> Verificação estrutural é barata, determinística e cobre o modo de falha mais grave. Muita gente pula direto para "LLM como juiz" e deixa passar respostas que nem eram JSON válido.

---

## 2️⃣ Métricas automáticas de texto

| Métrica | Mede | Uso |
|---|---|---|
| **BLEU** | Sobreposição de n-gramas | Tradução; ruim para texto livre |
| **ROUGE** | Cobertura de n-gramas da referência | Sumarização |
| **BERTScore** | Similaridade semântica via embedding | Melhor que BLEU para paráfrase |
| **Perplexidade** | Quão "esperado" é o texto | Diagnóstico, não qualidade |

> [!warning] Estas métricas exigem uma referência e punem paráfrase legítima
> Duas respostas igualmente boas com palavras diferentes recebem nota baixa. Para tarefas abertas, métricas de sobreposição medem semelhança superficial, não qualidade.

---

## 3️⃣ LLM como juiz (*LLM-as-a-judge*)

Usar um modelo para avaliar a saída de outro, com rubrica explícita.

```
Avalie a resposta abaixo em três critérios, de 1 a 5:

1. FIDELIDADE — todas as afirmações são sustentadas pelos dados fornecidos?
2. CLAREZA — o texto é compreensível para alguém sem conhecimento técnico?
3. UTILIDADE — a resposta ajuda a pessoa a decidir?

<dados>{contexto}</dados>
<resposta>{saida}</resposta>

Responda em JSON: {"fidelidade": n, "clareza": n, "utilidade": n, "justificativa": "..."}
```

✅ Escala, é barato, correlaciona razoavelmente com julgamento humano
❌ **Viés de posição** — tende a preferir a primeira opção apresentada
❌ **Viés de verbosidade** — tende a preferir respostas longas
❌ **Autopreferência** — tende a preferir texto do próprio modelo

**Mitigações:** alternar a ordem das opções entre execuções, usar rubrica com âncoras concretas, e **calibrar contra uma amostra avaliada por humanos**. Referência: *Zheng et al. (2023)*.

---

## 4️⃣ Avaliação humana (padrão-ouro)

| Desenho | Quando usar |
|---|---|
| **Likert por critério** | Medir dimensões separadas |
| **Comparação pareada A/B** | Mais confiável que nota absoluta |
| **Preferência cega** | Avaliador não sabe qual sistema é qual |
| **Anotação de erro** | Categorizar o tipo de falha |

> [!important] Comparação pareada cega é o desenho mais forte e o mais barato de justificar
> Humanos são ruins em dar notas absolutas consistentes e bons em dizer "prefiro este". Apresentar duas saídas sem identificação e perguntar qual é melhor produz dado mais confiável — e um resultado direto de reportar: *"em N=40 comparações, o caminho B foi preferido em 62% dos casos"*.

Reporte sempre: **N**, número de avaliadores, e concordância entre eles (Kappa de Cohen ou alfa de Krippendorff).

---

## 🔬 Desenho de experimento comparativo

```
Conjunto fixo de N entradas
        ↓
   ┌────┴────┐
Sistema A   Sistema B          ← prompt e modelo CONGELADOS
   └────┬────┘
        ↓
  taxa de divergência
  concordância no topo
  preferência humana cega
  latência e custo por chamada
```

**Controle mínimo para o experimento ter validade:**

- [ ] Prompt versionado e imutável durante a coleta
- [ ] Identificador exato do modelo registrado
- [ ] Parâmetros de geração fixos e anotados
- [ ] Data e hora de cada execução (modelos mudam sem aviso)
- [ ] Todas as saídas brutas persistidas, não só as métricas
- [ ] Ordem de apresentação randomizada na avaliação humana

> [!warning] Modelos comerciais mudam sem versionar
> O mesmo identificador de modelo pode se comportar diferente em datas diferentes. **Registre a data da coleta e guarde as saídas brutas.** Sem isso, o experimento é irreprodutível — e essa limitação precisa estar declarada. Ver [[met-validade-e-limitacoes|🎯 Validade e limitações]].

---

## 📚 Referências

- **Zheng et al. (2023)** — *Judging LLM-as-a-Judge with MT-Bench and Chatbot Arena*, NeurIPS
- **Liang et al. (2022)** — *Holistic Evaluation of Language Models* (HELM), Stanford
- **Zhang et al. (2020)** — *BERTScore: Evaluating Text Generation with BERT*
- **Chang et al. (2023)** — *A Survey on Evaluation of Large Language Models*

---

## 🔗 Conceitos relacionados

- [[ia-llm-fundamentos|🧠 LLM — Fundamentos]] · [[ia-alucinacao-e-grounding|🎭 Alucinação e grounding]]
- [[ia-engenharia-de-prompt|📝 Engenharia de prompt]] · [[rec-metricas-avaliacao|📊 Métricas de avaliação de recomendação]]
- [[met-tipos-de-pesquisa|🔬 Tipos de pesquisa]]

## Veja também

- [[Conceitos|🧠 Conceitos]]

---

**Status:** ✅ Completo
