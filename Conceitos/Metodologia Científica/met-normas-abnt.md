---
type: concept
area: Conceitos
status: estavel
difficulty: beginner
id: met-normas-abnt
category: Metodologia Científica
tags:
  - metodologia
  - concept
  - tcc
created: 2026-08-24
updated: 2026-08-24
---
# 📏 Normas ABNT

> Formatação não melhora o conteúdo, mas formatação errada custa nota em quase toda rubrica. É o ponto de maior retorno por hora investida no trabalho inteiro.

---

## 📚 As normas que importam em TCC

| Norma | Assunto |
|---|---|
| **NBR 14724** | Trabalhos acadêmicos — estrutura e apresentação |
| **NBR 10520** | Citações |
| **NBR 6023** | Referências |
| **NBR 6024** | Numeração progressiva das seções |
| **NBR 6027** | Sumário |
| **NBR 6028** | Resumo |
| **NBR 6034** | Índice |

> [!important] A norma da sua instituição tem precedência
> Quase toda instituição publica um manual próprio que adapta a ABNT. **Onde houver conflito, o manual institucional vence.** Consiga esse documento antes de escrever a primeira página — reformatar 60 páginas depois é trabalho perdido.

---

## 📐 Formatação do corpo (NBR 14724)

| Elemento | Especificação |
|---|---|
| Papel | A4 branco |
| Fonte | Arial ou Times New Roman, tamanho 12 |
| Margens | Esquerda e superior 3 cm; direita e inferior 2 cm |
| Espaçamento | 1,5 entre linhas |
| Parágrafo | Recuo de 1,25 cm na primeira linha |
| Alinhamento | Justificado |
| Paginação | Canto superior direito, contando da folha de rosto, **exibindo** a partir da introdução |

**Exceções com espaçamento simples e fonte menor (10):** citações longas, notas de rodapé, legendas de figuras e tabelas, ficha catalográfica.

---

## 🔢 Numeração de seções (NBR 6024)

```
1 SEÇÃO PRIMÁRIA           ← caixa alta, negrito
1.1 Seção secundária       ← negrito
1.1.1 Seção terciária      ← sem negrito
1.1.1.1 Seção quaternária
```

Regras que costumam ser violadas:

- **Sem ponto** após o último número (`1.1 Título`, nunca `1.1. Título`)
- **Sem ponto final** no título da seção
- Alinhado à margem esquerda, **sem** recuo
- Evite passar da quaternária
- Elementos sem numeração: resumo, sumário, referências, agradecimentos

---

## 📝 Citações (NBR 10520)

### Sistema autor-data — o mais usado

**No corpo do texto** (autor como parte da frase, só a inicial maiúscula):
> Segundo Salton e McGill (1983), a similaridade de cosseno...

**Entre parênteses** (autor fora da frase, **tudo em caixa alta**):
> A similaridade de cosseno neutraliza a magnitude (SALTON; MCGILL, 1983).

### Número de autores

| Autores | No corpo | Entre parênteses |
|---|---|---|
| 1 | Gil (2002) | (GIL, 2002) |
| 2 | Salton e McGill (1983) | (SALTON; MCGILL, 1983) |
| 3 | Pu, Chen e Hu (2011) | (PU; CHEN; HU, 2011) |
| 4+ | Hevner *et al.* (2004) | (HEVNER *et al.*, 2004) |

> [!note] Ponto e vírgula separa autores dentro dos parênteses
> `(SALTON; MCGILL, 1983)` — nunca vírgula entre os sobrenomes. E `et al.` vai em itálico.

### Citação longa (mais de 3 linhas)
Recuo de **4 cm** da margem esquerda, fonte 10, espaçamento simples, **sem aspas**.

---

## 📖 Referências (NBR 6023)

Em ordem alfabética, alinhadas à **esquerda** (não justificadas), espaçamento simples, separadas por uma linha em branco.

### Livro
```
SOBRENOME, Nome. Título da obra: subtítulo. Edição.
Cidade: Editora, ano.
```
> GIL, Antonio Carlos. **Como elaborar projetos de pesquisa**. 4. ed. São Paulo: Atlas, 2002.

### Artigo de periódico
```
SOBRENOME, Nome. Título do artigo. Título do Periódico,
cidade, v. X, n. Y, p. inicial-final, ano.
```
> KOREN, Yehuda; BELL, Robert; VOLINSKY, Chris. Matrix factorization techniques for recommender systems. **Computer**, v. 42, n. 8, p. 30-37, 2009.

### Trabalho em evento
```
SOBRENOME, Nome. Título. In: NOME DO EVENTO, número, ano,
cidade. Anais [...]. Cidade: Editora, ano. p. inicial-final.
```

### Documento eletrônico
Acrescente ao final:
> Disponível em: <URL>. Acesso em: 24 ago. 2026.

**Abreviação dos meses:** jan. fev. mar. abr. **maio** jun. jul. ago. set. out. nov. dez.
(maio é o único que não abrevia)

---

## 🖼️ Figuras e tabelas

| | Figura | Tabela |
|---|---|---|
| **Legenda** | **Acima** | **Acima** |
| **Fonte** | Abaixo, fonte 10 | Abaixo, fonte 10 |
| **Bordas laterais** | — | **Não** tem |
| Citação no texto | Obrigatória antes de aparecer | Obrigatória antes de aparecer |

```
Figura 1 — Arquitetura do sistema
        [ imagem ]
Fonte: elaborada pelo autor (2026).
```

> [!tip] "Fonte: elaborada pelo autor" é obrigatório mesmo no que você criou
> Figura sem fonte é erro de norma, inclusive quando a figura é sua. Toda figura e toda tabela precisam de legenda acima e fonte abaixo, sem exceção.

---

## 🛠️ Ferramentas

| Ferramenta | Uso |
|---|---|
| **Zotero + plugin ABNT** | Gera e mantém as referências automaticamente |
| **Template institucional** | Peça à coordenação — economiza horas |
| **Sumário automático do Word/LaTeX** | Nunca digite sumário à mão |
| **abnTeX2** | Classe LaTeX que já implementa a norma |
| **Mais que Nada / MoreThanABNT** | Geradores online, para conferência pontual |

> [!success] Formate desde o primeiro parágrafo
> Escrever 60 páginas sem estilo e formatar no fim é o caminho mais lento possível. Configure os estilos de título, o sumário automático e o gerenciador de referências **antes** de escrever — e a formatação deixa de existir como tarefa.

---

## 🔗 Conceitos relacionados

- [[met-estrutura-monografia|📄 Estrutura da monografia]] · [[met-revisao-bibliografica|📖 Revisão bibliográfica]]
- [[met-defesa-banca|🎤 Defesa de banca]]

## Veja também

- [[Conceitos|🧠 Conceitos]]
- [[Ferramentas|Ferramentas]]

---

**Status:** ✅ Completo
