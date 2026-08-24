---
type: concept
area: Conceitos
status: estavel
difficulty: intermediate
id: rec-explicabilidade
category: Recomendação
tags:
  - recomendacao
  - concept
  - etica
created: 2026-08-24
updated: 2026-08-24
---
# 💡 Recomendação Explicável

> Entregar junto com a recomendação o **motivo** dela. Deixa de ser um veredito e vira um argumento que a pessoa pode aceitar ou recusar.

---

## 📖 Definição

Um sistema de recomendação é **explicável** quando, além do item, produz uma justificativa compreensível para um humano.

```
Recomendação opaca:
  → "Curso de Mecânica Industrial"

Recomendação explicável:
  → "Curso de Mecânica Industrial (87% de afinidade)
     porque suas respostas indicaram forte interesse em
     trabalho manual de precisão e processos produtivos."
```

A diferença não é cosmética. Muda a relação da pessoa com o sistema: de **obediência** para **decisão informada**.

---

## 🎯 Por que explicar importa

*Tintarev & Masthoff (2007)* catalogaram sete objetivos distintos — e eles **competem entre si**:

| Objetivo | O que busca |
|---|---|
| **Transparência** | Explicar como o sistema funciona |
| **Escrutabilidade** | Permitir que a pessoa corrija o sistema |
| **Confiança** | Aumentar a credibilidade percebida |
| **Eficácia** | Ajudar a decidir melhor |
| **Persuasão** | Aumentar a chance de aceitar |
| **Eficiência** | Ajudar a decidir mais rápido |
| **Satisfação** | Melhorar a experiência |

> [!warning] Persuasão e eficácia puxam em direções opostas
> Uma explicação otimizada para **persuadir** aumenta a aceitação mesmo quando a recomendação é ruim. Uma explicação otimizada para **eficácia** ajuda a pessoa a recusar quando não serve. Em contexto educacional ou de saúde, otimizar por persuasão é manipulação. **Declare qual objetivo você escolheu.**

---

## 🧱 Tipos de explicação

### Baseada em atributo (*feature-based*)
"Recomendado porque é forte em X, e você demonstrou interesse em X."

Sai de graça em [[rec-filtragem-conteudo|filtragem baseada em conteúdo]]: basta identificar quais dimensões mais contribuíram para o resultado.

```python
def explain(perfil, item_vetor, nomes_dimensoes, top=2):
    """Devolve as dimensões que mais contribuíram para a similaridade."""
    contribuicoes = [
        (nomes_dimensoes[i], perfil[i] * item_vetor[i])
        for i in range(len(perfil))
    ]
    contribuicoes.sort(key=lambda c: c[1], reverse=True)
    return [nome for nome, valor in contribuicoes[:top] if valor > 0]
```

> [!tip] A contribuição por dimensão é o produto termo a termo do numerador
> No cosseno, o numerador é `Σ (Aᵢ × Bᵢ)`. Cada parcela dessa soma **é** a contribuição daquela dimensão. Ordenar as parcelas e pegar as maiores dá a explicação matematicamente correta — não uma narrativa inventada por cima do resultado.

### Baseada em vizinhança
"Pessoas com perfil parecido escolheram isto." Típica de [[rec-filtragem-colaborativa|filtragem colaborativa]] — mais fraca, porque a pessoa não pode verificar.

### Baseada em caso
"É parecido com X, que você já escolheu."

### Contrafactual
"Se você tivesse respondido diferente na pergunta 3, o resultado seria outro." A mais informativa e a mais cara de produzir.

---

## 🚨 A armadilha: explicação plausível ≠ explicação verdadeira

Existe uma diferença crítica entre:

| | |
|---|---|
| **Explicação fiel** | Descreve o que o sistema **realmente** fez |
| **Racionalização** | Texto plausível produzido **depois**, que não reflete o cálculo |

Gerar a recomendação por um método e pedir a um modelo de linguagem que "explique" o resultado produz **racionalização**, não explicação. O texto soa convincente e pode estar completamente desconectado da causa real.

> [!important] A arquitetura que preserva fidelidade
> Se um LLM participa da entrega, ele deve **receber a justificativa já calculada** e apenas traduzi-la para linguagem natural — nunca inventá-la. O cálculo é a fonte da verdade; o modelo é o redator. Qualquer desenho em que o modelo decide *e* explica quebra a fidelidade. Ver [[ia-alucinacao-e-grounding|🎭 Alucinação e grounding]].

---

## ⚖️ Explicabilidade como exigência legal

Não é só boa prática. A **LGPD (Lei 13.709/2018), art. 20** garante ao titular o direito de solicitar revisão de decisões automatizadas que afetem seus interesses, e o direito a informações claras sobre os critérios utilizados. O **GDPR** europeu tem dispositivo análogo (art. 22).

Em sistemas que orientam trajetória educacional ou profissional, isso deixa de ser diferencial e vira **conformidade**.

---

## 📊 Como avaliar uma explicação

| Dimensão | Pergunta | Como medir |
|---|---|---|
| **Fidelidade** | Reflete o cálculo real? | Auditoria técnica |
| **Compreensibilidade** | A pessoa entende? | Teste com usuários |
| **Utilidade** | Ajuda a decidir? | Taxa de decisão informada |
| **Confiança** | Aumenta credibilidade? | Escala Likert pós-uso |

Instrumento clássico: o questionário de *Pu, Chen & Hu (2011)* — **ResQue**, framework de avaliação centrada no usuário para sistemas de recomendação.

---

## 📚 Referências fundamentais

- **Tintarev & Masthoff (2007)** — *A Survey of Explanations in Recommender Systems*, ICDE Workshop — os sete objetivos
- **Zhang & Chen (2020)** — *Explainable Recommendation: A Survey and New Perspectives*
- **Pu, Chen & Hu (2011)** — *A User-Centric Evaluation Framework for Recommender Systems* (ResQue), RecSys
- **Ribeiro, Singh & Guestrin (2016)** — *"Why Should I Trust You?"* (LIME) — explicabilidade em ML geral

---

## 🔗 Conceitos relacionados

- [[rec-filtragem-conteudo|📄 Filtragem baseada em conteúdo]] — de onde a explicação sai de graça
- [[rec-vieses-e-etica|⚖️ Vieses e ética]] · [[ia-alucinacao-e-grounding|🎭 Alucinação e grounding]]
- [[rec-metricas-avaliacao|📊 Métricas de avaliação]]

## Veja também

- [[Conceitos|🧠 Conceitos]]

---

**Status:** ✅ Completo
