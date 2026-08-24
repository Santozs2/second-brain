---
type: concept
area: Conceitos
status: estavel
difficulty: intermediate
id: rec-cold-start
category: Recomendação
tags:
  - recomendacao
  - concept
created: 2026-08-24
updated: 2026-08-24
---
# 🥶 Cold Start

> O sistema precisa recomendar para quem acabou de chegar — e não sabe nada sobre essa pessoa. É o problema que decide a arquitetura inteira.

---

## 📖 Definição

**Cold start** é a incapacidade de gerar recomendações úteis por falta de dados históricos. O termo foi formalizado por *Schein et al. (2002)*. Existem três variantes, com dificuldades muito diferentes:

| Tipo | Situação | Gravidade |
|---|---|---|
| **Usuário novo** | Pessoa acabou de se cadastrar, sem histórico | 🔴 Alta — é o caso mais comum |
| **Item novo** | Item entrou no catálogo, ninguém interagiu | 🟡 Média — resolvida por atributos |
| **Sistema novo** | O produto acabou de nascer, ninguém usou nada | 🔴 Crítica — mata filtragem colaborativa |

---

## 💀 Por que isso mata a filtragem colaborativa

A [[rec-filtragem-colaborativa|filtragem colaborativa]] funciona por vizinhança: encontra pessoas parecidas com você e recomenda o que elas gostaram. Sem histórico:

```
Usuário novo → nenhuma interação → nenhum vizinho computável
             → nenhuma recomendação possível
```

Não é uma limitação de implementação. É **estrutural**: o método precisa de um dado que, por definição, não existe no primeiro acesso.

> [!important] O cold start de sistema é o que decide a arquitetura
> Um sistema institucional que nasce do zero **não tem histórico de ninguém**. Isso elimina filtragem colaborativa como opção — não por preferência técnica, mas por indisponibilidade de dados. É por isso que **abordagens baseadas em conteúdo são a escolha correta no dia zero**, e essa é uma justificativa metodológica forte, não um plano B.

---

## 🛠️ As estratégias de mitigação

### 1. Elicitação explícita de preferências (*onboarding*)

Perguntar diretamente. Um quiz, um formulário, uma seleção inicial de interesses constroem o vetor de perfil na hora, sem esperar comportamento.

- ✅ Resolve o cold start de usuário **completamente**
- ✅ Gera perfil interpretável — a pessoa sabe o que declarou
- ❌ Custa atrito: cada pergunta perde participantes
- ❌ O que a pessoa declara nem sempre é o que ela faz

> [!tip] O equilíbrio entre número de perguntas e qualidade do perfil é uma decisão de produto
> Poucas perguntas = perfil pobre. Muitas perguntas = ninguém termina. A literatura de *preference elicitation* trata isso como um problema de otimização: **maximizar informação obtida por pergunta feita**. Perguntas que quase todo mundo responde igual carregam pouca informação e podem ser cortadas.

### 2. Recomendação baseada em conteúdo

Usar atributos do item em vez de comportamento. Funciona no primeiro acesso porque a informação está no catálogo, não no usuário.

### 3. Estratégias de popularidade (*fallback*)

Recomendar o mais popular enquanto não há dados. É fraco, mas é melhor que tela vazia.

### 4. Dados demográficos

Inferir preferência por idade, região, escolaridade.

> [!warning] Demografia como proxy de preferência é um risco ético
> Recomendar cursos de uma área para um gênero e de outra área para outro, com base em correlação estatística, **reproduz e amplifica desigualdade existente**. É tecnicamente eficaz e eticamente indefensável em contexto educacional. Ver [[rec-vieses-e-etica|⚖️ Vieses e ética]].

### 5. Abordagem híbrida em cascata

Conteúdo no dia zero, migrando para colaborativa conforme o histórico acumula. Ver [[rec-sistemas-hibridos|🔀 Sistemas híbridos]].

---

## 📊 Comparação das estratégias

| Estratégia | Resolve usuário novo | Resolve item novo | Custo de implementação |
|---|:---:|:---:|---|
| Elicitação (quiz) | ✅ | ❌ | Médio |
| Conteúdo | ✅ | ✅ | Médio |
| Popularidade | 🟡 parcial | ❌ | Baixo |
| Demografia | 🟡 parcial | ❌ | Baixo (mas risco ético) |
| Híbrido cascata | ✅ | ✅ | Alto |

---

## 🎓 Como isto vira argumento de metodologia

O cold start transforma uma escolha que pareceria simplista em uma escolha **justificada pela literatura**. A estrutura do argumento:

1. O contexto é de sistema novo, sem base histórica de interações
2. *Schein et al. (2002)* estabelecem que filtragem colaborativa é inviável nesse regime
3. Portanto, adota-se abordagem baseada em conteúdo com elicitação explícita
4. A limitação decorrente (não capta preferência revelada, só declarada) fica **declarada**
5. Trabalho futuro: migração para híbrido conforme a base cresce

> [!success] Isto responde à pergunta mais provável da banca
> *"Por que vocês não usaram machine learning / redes neurais / filtragem colaborativa?"* — a resposta não é "era difícil". É **"o regime de dados não permite, e aqui está a citação que sustenta isso"**. Ver [[met-defesa-banca|🎤 Defesa de banca]].

---

## 📚 Referências fundamentais

- **Schein, Popescul, Ungar & Pennock (2002)** — *Methods and Metrics for Cold-Start Recommendations*, SIGIR — o artigo que formalizou o problema
- **Bobadilla et al. (2012)** — *A Collaborative Filtering Approach to Mitigate the New User Cold Start Problem*
- **Aggarwal, C. C. (2016)** — *Recommender Systems: The Textbook*, seção 1.3.1

---

## 🔗 Conceitos relacionados

- [[rec-filtragem-conteudo|📄 Filtragem baseada em conteúdo]] — a resposta padrão ao cold start
- [[rec-filtragem-colaborativa|👥 Filtragem colaborativa]] — o método que o cold start inviabiliza
- [[rec-sistemas-hibridos|🔀 Sistemas híbridos]] · [[rec-sistemas-de-recomendacao|🎯 Sistemas de Recomendação]]

## Veja também

- [[Conceitos|🧠 Conceitos]]

---

**Status:** ✅ Completo
