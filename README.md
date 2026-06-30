---
type: dashboard
tags:
  - dashboard
created: 2026-06-30
updated: 2026-06-30
status: ativo
---

# 📖 Como usar este Vault

Bem-vindo(a). Este Vault foi pensado para ser simples no dia a dia e, ao mesmo tempo, bem conectado — como uma pequena enciclopédia pessoal do que você está aprendendo e construindo.

## 🗺️ A ideia em uma frase

> Tecnologias ensinam Conceitos. Projetos usam Tecnologias e Conceitos. Recursos sustentam tudo.

Isso significa que os links sempre vão "para baixo": um Projeto aponta para as Tecnologias que usa, uma Tecnologia aponta para os Conceitos que explica, e um Conceito aponta para os Recursos onde aprender mais. Nunca o contrário — assim o grafo cresce como uma rede, não como uma estrela em volta do Dashboard.

## 📁 As pastas

| Pasta | O que tem dentro |
|---|---|
| 🏠 **Dashboard** | A `Home` — sua porta de entrada. Só navegação, nada de conteúdo. |
| 📚 **Estudos** | Uma nota por tecnologia (HTML, React, Django...). Resumo, conceitos, exemplos, checklist. |
| 🧠 **Conceitos** | Ideias transversais (REST API, JWT, ORM, MVC...) que aparecem em várias tecnologias e projetos. |
| 🚀 **Projetos** | Uma pasta por projeto, com sua própria `Home` interna. |
| 🧩 **Snippets** | Trechos de código prontos, por linguagem. |
| 📔 **Diário** | Uma nota por dia de estudo. |
| 📦 **Recursos** | Livros, cursos, vídeos, artigos, documentações e ferramentas. |
| 🧰 **Templates** | Moldes para criar notas novas. Fica isolada do Graph de propósito. |

Cada pasta de conteúdo (Estudos, Conceitos, Projetos, Snippets, Recursos, Diário) tem uma nota índice com o mesmo nome da pasta (ex: `Estudos/Estudos.md`) — é por ela que o Dashboard navega até o conteúdo.

## 🔁 Fluxo diário

1. Abra o Obsidian — ele já abre direto na `Home` do Dashboard.
2. Marque o que vai estudar hoje em **🎯 Objetivos da semana** (ou confira o que já estava marcado).
3. Use o ícone do **Calendar** na barra lateral para criar/abrir a nota do dia em `Diário/`.
4. Preencha o diário ao final do estudo: o que estudou, o que aprendeu, dificuldades, o que vem amanhã.

## 🗓️ Fluxo semanal

1. No domingo (ou no dia que preferir), abra a `Home` e atualize **🎯 Objetivos da semana** e **📖 Tecnologia em foco**.
2. Dê uma olhada nas **✅ Tarefas em aberto** (lista automática, vem do plugin Tasks) e limpe o que já não faz sentido.
3. Se começou ou avançou um projeto, atualize o link de **🧪 Projeto atual**.

## ✍️ Como criar notas novas

Use os templates em `Templates/` — com o plugin **Templater**, basta criar uma nota vazia na pasta certa e rodar o template (ou configurar para inserir automaticamente). Cada template já vem com as seções certas:

- **Nova tecnologia** → `Template - Nota de Estudo` (em `Estudos/`)
- **Novo projeto** → `Template - Projeto` (crie uma pasta em `Projetos/<nome>/` com uma `Home.md` de índice, como em `Conecta Senai`)
- **Novo snippet** → `Template - Snippet` (ou adicione uma seção no arquivo da linguagem já existente em `Snippets/`)
- **Novo recurso** → `Template - Recurso`
- **Novo erro/bug** → `Template - Erro`
- **Nova ideia** → `Template - Ideia`
- **Diário** → `Template - Diário` (automático pelo Calendar)

> [!tip] Antes de criar, procure
> Se já existe uma nota parecida, prefira editá-la em vez de criar outra. Isso evita duplicatas como aconteceu antes (ex: duas notas de "Banco de Dados").

## 🧠 Como estudar usando o sistema

1. Toda tecnologia nova vira uma nota em `Estudos/`.
2. Sempre que um conceito se repete em mais de uma tecnologia (ex: ORM aparece em Django e em outros ORMs), ele vira uma nota em `Conceitos/` — e a tecnologia linka para lá.
3. Quando você usa a tecnologia em um projeto de verdade, o projeto linka de volta para a tecnologia (nunca o contrário).

## 🚀 Como organizar um projeto

Cada projeto é uma pasta em `Projetos/<Nome do Projeto>/` com uma `Home.md` que navega para:

`Sobre` · `Arquitetura` (com seções `#Backend` e `#Frontend`) · `Banco de Dados` · `API` · `Roadmap` · `Deploy` · `Aprendizados` · `Bugs` · `Melhorias`

Veja `Projetos/Conecta Senai/Home.md` como exemplo real já preenchido.

## 🌱 Como evoluir o Vault com o tempo

- **Não crie pastas novas** sem necessidade — sub-pastas dentro das 8 existentes resolvem quase tudo.
- **Prefira editar** uma nota existente a criar uma parecida.
- Quando perceber uma ideia se repetindo em vários lugares, **transforme em um Conceito**.
- De vez em quando, abra o **Graph View** e veja se alguma nota ficou isolada (órfã) — se ficou, adicione um link em "Veja também".
- Mantenha o frontmatter (`type`, `tags`, `created`, `updated`, `status`) em notas novas — ajuda a filtrar e a manter o Dataview funcionando.
