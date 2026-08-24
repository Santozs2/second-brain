---
type: concept
area: Conceitos
status: estavel
difficulty: intermediate
id: met-revisao-bibliografica
category: Metodologia Científica
tags:
  - metodologia
  - concept
  - tcc
created: 2026-08-24
updated: 2026-08-24
---
# 📖 Revisão Bibliográfica

> Não é listar o que outros escreveram. É construir, com fontes, o argumento de que a sua escolha era a escolha certa.

---

## 🎯 As três funções da revisão

| Função | O que faz |
|---|---|
| **Fundamentar** | Sustentar cada afirmação técnica com fonte |
| **Posicionar** | Mostrar onde seu trabalho se encaixa no que já existe |
| **Justificar** | Demonstrar que a lacuna que você preenche é real |

---

## 🔍 Onde buscar

| Base | Perfil |
|---|---|
| **Google Scholar** | Cobertura ampla; bom ponto de partida |
| **Portal de Periódicos CAPES** | Acesso a bases pagas via instituição |
| **SciELO** | Produção brasileira e latino-americana |
| **IEEE Xplore / ACM DL** | Computação, o padrão internacional |
| **arXiv** | Pré-prints; rápido, mas **sem revisão por pares** |
| **BDTD** | Teses e dissertações brasileiras |

> [!warning] Pré-print não é artigo revisado
> arXiv publica sem revisão por pares. É legítimo citar — muito da literatura recente de IA vive lá — mas identifique como pré-print e prefira a versão publicada quando existir.

---

## 🧭 Avaliar a qualidade de uma fonte

| Critério | Sinal bom | Sinal ruim |
|---|---|---|
| **Veículo** | Periódico ou conferência conhecida | Blog, site de empresa, conteúdo de marketing |
| **Revisão por pares** | Sim | Não informado |
| **Citações** | Consistente com a idade do trabalho | Zero após muitos anos |
| **Autoria** | Vinculação institucional identificável | Autor anônimo |
| **Método** | Descrito e reproduzível | Ausente |

**Hierarquia de preferência:**

```
Livro-texto consagrado  >  Periódico revisado  >  Conferência
  >  Dissertação/tese  >  Pré-print  >  Documentação oficial  >  Blog técnico
```

> [!tip] Documentação oficial é fonte legítima para fato técnico
> Citar a documentação do Django para afirmar como o ORM se comporta é apropriado — é a fonte primária. O que ela não sustenta é afirmação **comparativa ou avaliativa** ("é o melhor framework"): para isso é preciso literatura ou experimento próprio.

---

## 🧱 Como estruturar o capítulo

O erro mais comum é organizar por autor. A revisão deve ser organizada por **conceito**, com os autores aparecendo como sustentação.

```
❌ Por autor (vira lista, não argumento)
   2.1 Salton (1975)
   2.2 Burke (2002)
   2.3 Ricci (2011)

✅ Por conceito (vira argumento)
   2.1 Sistemas de recomendação: definição e taxonomia
   2.2 Abordagens baseadas em conteúdo
   2.3 O problema do cold start
   2.4 Medidas de similaridade
   2.5 Recomendação explicável
```

Cada seção termina fechando um elo do argumento. Ao fim do capítulo, a escolha técnica do trabalho deve parecer **inevitável**.

---

## ✍️ Citação: os três modos

### Indireta (paráfrase) — o modo padrão
> A similaridade de cosseno é amplamente adotada em recuperação de informação por desconsiderar a magnitude dos vetores (SALTON; MCGILL, 1983).

### Direta curta (até 3 linhas) — entre aspas, no corpo
> Segundo Gil (2002, p. 42), a pesquisa aplicada "objetiva gerar conhecimentos para aplicação prática dirigidos à solução de problemas específicos".

### Direta longa (mais de 3 linhas) — recuo de 4 cm, fonte menor, sem aspas

> [!important] Use paráfrase na esmagadora maioria das vezes
> Citação direta demais sinaliza que o autor não digeriu o material. Reserve-a para definições formais e frases que perderiam precisão se reescritas. **A regra prática: se você consegue dizer com suas palavras, diga.**

---

## 🚨 Plágio

| Forma | Descrição |
|---|---|
| **Direto** | Copiar sem aspas e sem fonte |
| **Mosaico** | Trocar palavras mantendo a estrutura original |
| **De autoplágio** | Reaproveitar texto próprio já entregue sem indicar |
| **De ideia** | Apresentar conceito de outro como seu |
| **De fonte secundária** | Citar A tendo lido apenas B, que citava A |

> [!warning] Texto gerado por IA sem revisão e sem verificação é risco duplo
> Além da questão de autoria — que varia conforme a política da instituição, e **precisa ser consultada** — modelos de linguagem produzem referências inexistentes com formatação perfeita. **Toda referência precisa ser aberta e verificada.** Citar um artigo que não existe é o tipo de erro que compromete a credibilidade do trabalho inteiro. Ver [[ia-alucinacao-e-grounding|🎭 Alucinação]].

Ferramentas de verificação: Turnitin, Plagius, CopySpider.

---

## 🗂️ Organização prática

Use um gerenciador desde o primeiro dia: **Zotero** (livre, o mais usado), Mendeley ou JabRef. Eles geram as referências em ABNT automaticamente e evitam a noite de formatação manual.

**Fluxo que funciona:**

```
Encontrou → salva no Zotero (com PDF)
          → lê e anota a ideia central em uma frase própria
          → registra a página exata das passagens úteis
          → escreve já citando; nunca "cito depois"
```

> [!tip] Uma linha por fonte, escrita no dia da leitura
> "O que este texto me permite afirmar?" — respondido em uma frase, no momento da leitura. Sem isso, você chega na escrita com 40 PDFs e nenhuma lembrança de por que baixou cada um.

---

## 📚 Referências

- **Gil, A. C. (2002)** — *Como Elaborar Projetos de Pesquisa*, Atlas
- **Booth, Colomb & Williams** — *The Craft of Research*
- **Kitchenham & Charters (2007)** — *Guidelines for Performing Systematic Literature Reviews in Software Engineering*
- **ABNT NBR 10520** — Citações em documentos

---

## 🔗 Conceitos relacionados

- [[met-normas-abnt|📏 Normas ABNT]] · [[met-estrutura-monografia|📄 Estrutura da monografia]]
- [[met-etica-em-pesquisa|🤝 Ética em pesquisa]] · [[met-tipos-de-pesquisa|🔬 Tipos de pesquisa]]

## Veja também

- [[Conceitos|🧠 Conceitos]]
- [[Artigos|Artigos]] · [[Livros|Livros]]

---

**Status:** ✅ Completo
