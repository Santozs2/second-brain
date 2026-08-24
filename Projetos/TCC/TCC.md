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
  - ia
  - llm
created: 2026-08-17
updated: 2026-08-20
---
# 🎓 TCC — Quiz de Perfil e Recomendação de Cursos

> [!abstract] Visão geral
> Sistema web que aplica um **quiz de perfil profissional** e recomenda cursos com base nas respostas. O núcleo é uma **engine de matching por similaridade de cosseno** que compara o perfil da pessoa com o vetor de cada curso, devolvendo um ranking **com justificativa** — não uma caixa-preta.
> Repositório: `github.com/Santozs2/tcc-train` · Pasta local: `C:\Users\USER\Documents\vs code\tcc-treine`

## 🗺️ Páginas do projeto

### 📐 Escopo e equipe

- [[escopo-fluxo-educmatch|🗺️ Fluxo EducMatch e recorte de escopo]] — o quadro do grupo, o que entra e o que fica de fora
- [[divisao-de-trabalho-tcc|👥 Divisão de trabalho (4 frentes)]] — quem faz o quê e as fronteiras entre as partes
- [[spec-motor-e-ia-frentes-1-2|🧭 Spec das Frentes 1 e 2]] — backlog, contratos e cronograma do motor e da camada de IA
- [[passo-a-passo-f1-01-f1-02|🔧 Passo a passo de F1-01 e F1-02]] — o código das duas primeiras tarefas, arquivo por arquivo

### 🛠️ Técnico

- [[guia-tcc-quiz-perfil|🏗️ Guia de implementação]] — os 6 passos, do setup ao site no ar
- [[modelagem-dados-quiz|🗃️ Modelagem de dados]] — áreas, cursos, pesos e tentativas
- [[engine-matching-cosseno|🧮 Engine de matching (cosseno)]] — o coração do trabalho
- [[catalogo-areas-e-cursos|📚 Catálogo de áreas, cursos e perguntas]] — a base populada pelos seeds
- [[api-quiz-drf|🔌 API REST do quiz]] — endpoints DRF
- [[front-templates-django|🎨 Front em templates Django]] — wizard e página de resultado
- [[testes-e-validacao-tcc|✅ Testes e validação]] — 12 testes automatizados + smoke test
- [[bugs-e-licoes-tcc|🐛 Bugs e lições aprendidas]] — o que quebrou e por quê
- [[fundamentacao-teorica-recomendacao|📖 Fundamentação teórica]] — cold start, VSM e por que cosseno
- [[artigo-secao-calculo-cosseno|✍️ Artigo: seção do cálculo de cosseno]] — como a engine vira metodologia escrita
- [[defesa-monografia-tcc|🎤 Defesa e monografia]] — argumentos para a banca e roadmap

### 🤖 Camada de IA (Passos 7 a 11)

- [[decisao-camada-ia|🤖 Decisão da camada de IA]] — desenhos A/B/C, orçamento e a decisão registrada
- [[camada-ia-plano-implementacao|🧩 Plano de implementação]] — contratos, arquivos, os 5 passos e o DoD de cada um
- [[prompt-padrao-recomendacao|📝 Prompt padrão de entrega (v1)]] — o prompt único, regra a regra

## 🎯 Estado atual

> [!success] Fim a fim funcionando
> Cadastro no admin → quiz → engine → ranking explicado. Base populada com 7 áreas, 12 cursos e 6 perguntas. 12 testes automatizados passando. Site em templates Django com wizard e página de resultado.

> [!info] Camada de IA decidida em 2026-08-20 — **Desenho B modificado**
> A engine seleciona o top-5; a **LLM entrega os cursos** por um **prompt padrão**, podendo reordenar dentro desses 5. A tela passa a mostrar **1 curso principal + 4 alternativas**. Provedor padrão: **Gemini Flash free tier** (custo previsto: R$ 0), plugável por `.env`. O sistema continua funcionando com a IA desligada.

> [!todo] Próximos passos
> **Passo 7** — protocolo `LLMProvider` + `FakeProvider` + contratos, tudo offline ([[camada-ia-plano-implementacao|🧩 plano]]). Depois: scraping do catálogo real de cursos (alimenta o `catalog`) e tela de histórico de tentativas.

## 🧱 Stack

- **Backend:** [[Django|Django]] 5.2 + [[Django REST Framework|DRF]]
- **Banco:** SQLite ([[Banco de Dados|Banco de Dados]] · [[ORM|ORM]])
- **Front:** Templates Django + [[CSS|CSS]] puro + [[JavaScript|JavaScript]] vanilla
- **Testes:** `unittest` nativo do Django (`manage.py test`)
- **IA:** Gemini Flash (free tier) atrás de um protocolo `LLMProvider` — trocável por `.env`, com `FakeProvider` para desenvolver offline
- **Versionamento:** [[Git|Git]]

## 🧠 Decisões que definem o projeto

| Decisão | Por quê |
|---|---|
| Similaridade de **cosseno**, não soma de pontos | Neutraliza o "curso gordo" que pesa muitas áreas |
| Pesos em **tabela relacional**, não JSON | Integridade referencial, consultável e editável no admin |
| Engine em **funções puras** separadas do ORM | Testável sem banco e defensável como arquitetura |
| Campo `explanation` em cada recomendação | Recomendação **explicável** — o diferencial do trabalho |
| **Sem** multi-tenant, sem JWT, SQLite | O valor está na engine, não na infraestrutura |
| LLM **entrega**, engine **decide os candidatos** | A IA nunca escolhe fora do top-5 — sem alucinação de curso |
| **Um** prompt padrão para todos os perfis | Sem prompt fixo, o experimento comparativo não teria validade |
| Fallback para a engine em qualquer falha | O TCC responde com a internet caída — a IA é incremento, não dependência |

## 🔗 Relacionado no Vault

- **Backend:** [[Django|Django]] · [[Django REST Framework|DRF]] · [[Python|Python]] · [[REST API|REST API]] · [[Serializers|Serializers]] · [[Views|Views]]
- **Dados:** [[Models|Models]] · [[Migrations|Migrations]] · [[ORM|ORM]] · [[Banco de Dados|Banco de Dados]] · [[CRUD|CRUD]]
- **Conceitos:** [[cs-linear-algebra|Álgebra linear]] · [[MVC|MVC]] · [[HTTP|HTTP]]
- **Outros projetos:** [[ChatBot|💬 ChatBot]]

## Veja também

- [[Projetos|🚀 Projetos]]
- [[Home|Painel Principal]]
