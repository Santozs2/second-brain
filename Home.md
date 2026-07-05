---
type: dashboard
tags:
  - dashboard
created: 2026-07-05
updated: 2026-07-05
status: ativo
---

# 🏠 Painel Principal

> Sua base de conhecimento pessoal. Comece por aqui.

## 🗺️ Áreas

| Área | O que tem dentro |
|---|---|
| 📚 [[Estudos]] | Linguagens, frameworks, ferramentas e web — uma nota por tecnologia. |
| 🧠 [[Conceitos]] | Ideias transversais e fundamentos de Ciência da Computação. |
| 🚀 [[Projetos]] | Projetos reais, cada um com sua própria página inicial. |
| 🧩 [[Snippets]] | Trechos de código prontos, por linguagem. |
| 📦 [[Recursos]] | Livros, cursos, vídeos, artigos, documentações e ferramentas. |
| 📔 [[Diário]] | Registro diário de estudo. |

## 🧪 Projetos ativos

```dataview
LIST
FROM "Projetos"
WHERE type = "project" OR type = "goal"
SORT file.name ASC
```

## 🕒 Editado recentemente

```dataview
TABLE updated AS "Atualizado", type AS "Tipo"
FROM ""
WHERE file.name != this.file.name
SORT updated DESC
LIMIT 10
```

## 🧭 Como o Vault funciona

> Tecnologias ensinam Conceitos. Projetos usam Tecnologias e Conceitos. Recursos sustentam tudo.

Os links sempre apontam "para baixo" — um Projeto aponta para as Tecnologias que usa, uma Tecnologia aponta para os Conceitos que explica. Assim o grafo cresce como uma rede, não como uma estrela. Veja o guia completo em [[README]].
