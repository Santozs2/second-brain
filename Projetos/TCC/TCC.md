---
type: project
area: Projetos
status: 🟢 Em Desenvolvimento
priority: 🔥 Alta
tecnologia: Django
tags:
  - project
  - tcc
  - quiz
  - django
  - recomendacao
created: 2026-08-17
updated: 2026-08-17
---
# 🎓 TCC — Quiz de Perfil e Recomendação de Cursos

> [!abstract] Visão geral
> Sistema web que aplica um **quiz de perfil profissional** e recomenda cursos com base nas respostas. O núcleo é uma **engine de matching por similaridade de cosseno** que compara o perfil da pessoa com o vetor de cada curso, devolvendo um ranking **com justificativa** — não uma caixa-preta.
> Repositório: `github.com/Santozs2/tcc-train` · Pasta local: `C:\Users\USER\Documents\vs code\tcc-treine`

## 🗺️ Páginas do projeto

- [[guia-tcc-quiz-perfil|🏗️ Guia de implementação]] — os 6 passos, do setup ao site no ar
- [[modelagem-dados-quiz|🗃️ Modelagem de dados]] — áreas, cursos, pesos e tentativas
- [[engine-matching-cosseno|🧮 Engine de matching (cosseno)]] — o coração do trabalho
- [[catalogo-areas-e-cursos|📚 Catálogo de áreas, cursos e perguntas]] — a base populada pelos seeds
- [[api-quiz-drf|🔌 API REST do quiz]] — endpoints DRF
- [[front-templates-django|🎨 Front em templates Django]] — wizard e página de resultado
- [[testes-e-validacao-tcc|✅ Testes e validação]] — 12 testes automatizados + smoke test
- [[bugs-e-licoes-tcc|🐛 Bugs e lições aprendidas]] — o que quebrou e por quê
- [[defesa-monografia-tcc|🎤 Defesa e monografia]] — argumentos para a banca e roadmap

## 🎯 Estado atual

> [!success] Fim a fim funcionando
> Cadastro no admin → quiz → engine → ranking explicado. Base populada com 7 áreas, 12 cursos e 6 perguntas. 12 testes automatizados passando. Site em templates Django com wizard e página de resultado.

> [!todo] Próximos passos
> Scraping do catálogo real de cursos (alimenta o `catalog`) e tela de histórico de tentativas.

## 🧱 Stack

- **Backend:** [[Django|Django]] 5.2 + [[Django REST Framework|DRF]]
- **Banco:** SQLite ([[Banco de Dados|Banco de Dados]] · [[ORM|ORM]])
- **Front:** Templates Django + [[CSS|CSS]] puro + [[JavaScript|JavaScript]] vanilla
- **Testes:** `unittest` nativo do Django (`manage.py test`)
- **Versionamento:** [[Git|Git]]

## 🧠 Decisões que definem o projeto

| Decisão | Por quê |
|---|---|
| Similaridade de **cosseno**, não soma de pontos | Neutraliza o "curso gordo" que pesa muitas áreas |
| Pesos em **tabela relacional**, não JSON | Integridade referencial, consultável e editável no admin |
| Engine em **funções puras** separadas do ORM | Testável sem banco e defensável como arquitetura |
| Campo `explanation` em cada recomendação | Recomendação **explicável** — o diferencial do trabalho |
| **Sem** multi-tenant, sem JWT, SQLite | O valor está na engine, não na infraestrutura |

## 🔗 Relacionado no Vault

- **Backend:** [[Django|Django]] · [[Django REST Framework|DRF]] · [[Python|Python]] · [[REST API|REST API]] · [[Serializers|Serializers]] · [[Views|Views]]
- **Dados:** [[Models|Models]] · [[Migrations|Migrations]] · [[ORM|ORM]] · [[Banco de Dados|Banco de Dados]] · [[CRUD|CRUD]]
- **Conceitos:** [[cs-linear-algebra|Álgebra linear]] · [[MVC|MVC]] · [[HTTP|HTTP]]
- **Outros projetos:** [[ChatBot|💬 ChatBot]]

## Veja também

- [[Projetos|🚀 Projetos]]
- [[Home|Painel Principal]]
