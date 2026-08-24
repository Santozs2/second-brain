---
type: concept
area: Conceitos
status: estavel
difficulty: beginner
id: met-estrutura-monografia
category: Metodologia Científica
tags:
  - metodologia
  - concept
  - tcc
created: 2026-08-24
updated: 2026-08-24
---
# 📄 Estrutura da Monografia

> Cada capítulo responde a **uma** pergunta. Se você não sabe qual pergunta o capítulo responde, ele ainda não tem motivo para existir.

---

## 🗺️ A estrutura canônica

| Capítulo | A pergunta que ele responde | Extensão típica |
|---|---|---|
| **1. Introdução** | Qual é o problema e por que ele importa? | 4–8 pág. |
| **2. Fundamentação teórica** | O que já se sabe sobre isto? | 10–20 pág. |
| **3. Metodologia** | Como vocês fizeram, e por que assim? | 8–15 pág. |
| **4. Desenvolvimento** | O que foi construído? | 10–20 pág. |
| **5. Resultados** | O que aconteceu quando testaram? | 8–15 pág. |
| **6. Considerações finais** | O que aprendemos e o que fica para depois? | 3–6 pág. |

---

## 1️⃣ Introdução

Cinco elementos, nesta ordem:

```
Contextualização  → o cenário onde o problema vive
Problema          → uma frase, em forma de pergunta
Objetivo geral    → um verbo no infinitivo, um resultado
Objetivos específicos → 3 a 5, cada um verificável
Justificativa     → por que isto merece existir
```

> [!important] O objetivo geral e a conclusão precisam se espelhar
> Se o objetivo é *"desenvolver um sistema de recomendação de cursos baseado em similaridade vetorial"*, a conclusão precisa dizer se isso foi feito. Trabalho que promete uma coisa na página 3 e entrega outra na página 60 perde nota — mesmo quando a coisa entregue é boa.

**Verbos que funcionam em objetivo específico** (verificáveis): implementar, avaliar, comparar, mensurar, validar, documentar.
**Verbos que não funcionam** (não são verificáveis): entender, explorar, estudar, abordar.

---

## 2️⃣ Fundamentação teórica

Não é resumo de livro. É a construção do **argumento de que sua escolha técnica é a escolha certa** para o seu contexto.

A estrutura que funciona:

```
1. O campo geral        → o que é a área
2. As alternativas      → quais abordagens existem
3. O critério           → o que decide entre elas no seu caso
4. A escolha            → qual você adotou e por quê
5. As limitações        → o que essa escolha custa
```

> [!tip] Cada afirmação técnica precisa de uma fonte ou de um experimento
> "Cosseno é melhor para este caso" sem citação é opinião. "Cosseno neutraliza o efeito da magnitude (Salton & McGill, 1983), o que é desejável aqui porque..." é fundamentação. A diferença entre as duas frases é a diferença entre relatório e monografia.

Ver [[met-revisao-bibliografica|📖 Revisão bibliográfica]].

---

## 3️⃣ Metodologia

O capítulo que mais separa trabalhos. Precisa permitir que **outra pessoa reproduza** o que você fez.

Componentes:

- **Classificação da pesquisa** — natureza, objetivo, abordagem, procedimento → [[met-tipos-de-pesquisa|🔬 Tipos de pesquisa]]
- **Materiais** — tecnologias, versões, ambiente
- **Procedimento** — o passo a passo do que foi feito
- **Instrumentos** — questionários, roteiros, scripts
- **Método de análise** — como os dados coletados viraram conclusão

> [!warning] Descreva o método, não a cronologia
> "Primeiro tentamos X, não deu certo, então fomos para Y" é diário de bordo. O capítulo de metodologia descreve **o procedimento adotado**, na forma como ele deve ser reproduzido. As tentativas frustradas, quando ensinam algo, vão para os resultados ou para uma seção de lições aprendidas.

---

## 4️⃣ Desenvolvimento

O que foi construído: arquitetura, modelagem de dados, decisões de implementação, telas.

- Cada decisão relevante vem com a alternativa descartada e o motivo
- Diagramas > parágrafos descritivos
- Código só quando ele **é** o argumento (o algoritmo central, não o CRUD)

---

## 5️⃣ Resultados

Duas partes que não devem se misturar:

| | O que vai |
|---|---|
| **Apresentação** | Os números, as tabelas, os gráficos. Sem interpretação. |
| **Discussão** | O que os números significam, e o que eles não permitem concluir. |

> [!success] A discussão dos casos que deram errado vale mais que a dos que deram certo
> Um trabalho que mostra 100% de acerto levanta suspeita de método frouxo. Um que mostra 78% de acerto **e analisa os 22%** demonstra domínio. Divergência analisada é resultado; divergência escondida é risco.

---

## 6️⃣ Considerações finais

- Retomar o objetivo e dizer se foi atingido
- Sintetizar as contribuições (o que existe no mundo que não existia antes)
- Declarar as limitações → [[met-validade-e-limitacoes|🎯 Validade e limitações]]
- Propor trabalhos futuros — específicos, não genéricos

**Genérico (fraco):** "pretende-se aprimorar o sistema"
**Específico (forte):** "expandir o catálogo de 18 para 180 cursos, o que exige um método escalável de atribuição de pesos — possivelmente derivação automática a partir das ementas"

---

## 📐 Elementos pré e pós-textuais (ABNT)

```
Capa · Folha de rosto · Folha de aprovação · Dedicatória* · Agradecimentos*
Resumo (PT) + palavras-chave · Abstract (EN) + keywords
Listas de figuras/tabelas/siglas* · Sumário
    ── TEXTO ──
Referências (obrigatório) · Apêndices* (feitos por você) · Anexos* (de terceiros)
                                                    * opcional
```

Ver [[met-normas-abnt|📏 Normas ABNT]].

---

## ✍️ Regras de escrita acadêmica

| Regra | Exemplo |
|---|---|
| Impessoalidade | ~~"eu fiz"~~ → "foi desenvolvido" / "o trabalho apresenta" |
| Tempo verbal | Metodologia e resultados no passado; conclusões no presente |
| Sigla | Nome por extenso na primeira ocorrência, sigla entre parênteses |
| Figura | Toda figura é citada no texto antes de aparecer |
| Afirmação | Toda afirmação não-óbvia tem fonte ou dado |

---

## 🔗 Conceitos relacionados

- [[met-metodo-cientifico|🔬 Método científico]] · [[met-tipos-de-pesquisa|🔬 Tipos de pesquisa]]
- [[met-revisao-bibliografica|📖 Revisão bibliográfica]] · [[met-normas-abnt|📏 Normas ABNT]]
- [[met-defesa-banca|🎤 Defesa de banca]] · [[met-validade-e-limitacoes|🎯 Validade e limitações]]

## Veja também

- [[Conceitos|🧠 Conceitos]]

---

**Status:** ✅ Completo
