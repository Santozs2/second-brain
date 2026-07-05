---
type: dashboard
id: doc-readme
created: 2026-07-04
updated: 2026-07-04
category: Documentation
tags:
  - type/dashboard
  - documentation
---

# 📖 Como Usar Este Vault

## 🎯 Visão Geral

Este é um **Knowledge OS** — um sistema de organização pessoal do conhecimento preparado para IA (AMEA).

```
Conhecimento = Nó + Contexto + Relacionamentos
```

Cada **nota é um nó** em um grafo de conhecimento com contexto estruturado e relacionamentos explícitos.

---

## 🗺️ Estrutura

```
Second Brain/
├── 🏠 Dashboard/           ← Você está aqui agora
├── 🧠 Knowledge/           ← Todo conhecimento
│   ├── Technologies/       ← Linguagens, frameworks
│   ├── Concepts/           ← Ideias transversais
│   ├── Code/               ← Snippets e padrões
│   ├── Resources/          ← Links externos
│   └── ...
├── 🚀 Projects/            ← Projetos em andamento
└── 🔧 System/              ← Configuração e templates
    ├── Templates/          ← Moldes para notas
    ├── Guides/             ← Guias e convenções
    └── Documentation/      ← Docs (você está aqui)
```

---

## 📚 Tipos de Notas

Existem **15 tipos** de notas, cada uma com sua estrutura:

| Tipo | Pasta | Exemplo |
|------|-------|---------|
| **Technology** | Knowledge/Technologies | React, Python, Docker |
| **Concept** | Knowledge/Concepts | MVC, REST API, ORM |
| **Project** | Projects | Conecta SENAI, AMEA AI |
| **Resource** | Knowledge/Resources | Cursos, Livros, Docs |
| **Snippet** | Knowledge/Code/Snippets | Código reutilizável |
| **MOC** | (anywhere) | Índices e mapa de conteúdo |
| **Dashboard** | Dashboard | Painéis analíticos |
| (+ 8 mais) | | — |

---

## 🏷️ Frontmatter Padrão

Toda nota tem **frontmatter YAML** estruturado:

```yaml
---
type: technology                # Tipo de nota
id: tech-react-001             # ID único
created: 2026-07-04            # Data de criação
updated: 2026-07-04            # Última modificação
category: Frontend              # Categoria
tags:                          # Tags padronizadas
  - type/technology
  - domain/frontend
status: learning               # learning | stable | reference
confidence: 0.85               # 0.0 - 1.0
source: "Official Docs"        # De onde veio
dependencies:                  # O que depende
  - tech-javascript
ai_generated: false            # Criado por IA?
last_reviewed: 2026-07-04      # Última revisão
---
```

---

## 🔗 Como Linkar Notas

Use wikilinks com caminho completo:

```markdown
# ✅ CORRETO
[[Knowledge/Technologies/frontend-react|React]]
[[Knowledge/Concepts/Architecture/concept-mvc|MVC Pattern]]

# Em campos de relacionamento
dependencies:
  - Knowledge/Technologies/frontend-javascript
applied_in:
  - Projects/Conecta-SENAI/INDEX
```

---

## 📝 Criar Nova Nota

### Opção 1: Usar Template (Recomendado)

1. Vá para `System/Templates/`
2. Copie o template apropriado
3. Salve na pasta correta com nome em slug-case
4. Preencha o frontmatter
5. Adicione conteúdo

**Nome correto:** `frontend-react.md` (não `React Advanced.md`)

### Opção 2: Manual

```markdown
---
type: technology
id: tech-nova-001
created: YYYY-MM-DD
updated: YYYY-MM-DD
category: Frontend
tags:
  - type/technology
  - domain/frontend
status: draft
confidence: 0.5
---

# Título da Nota

[Conteúdo...]
```

---

## 📊 Sistema de Tags

Tags devem seguir padrão: `categoria/valor`

```yaml
tags:
  - type/technology           # Sempre incluir tipo
  - domain/frontend           # Domínio (frontend/backend/devops)
  - status/learning           # Status
  - difficulty/intermediate   # Dificuldade
```

---

## 🧬 Relacionamentos

Cada nota pode ter relacionamentos explícitos:

```yaml
dependencies:               # O que preciso saber antes
  - Knowledge/Technologies/frontend-javascript

required_by:               # O que me depende
  - Knowledge/Technologies/frontend-react

related:                   # Relacionado
  - Knowledge/Concepts/component-pattern

applied_in:                # Onde sou usado
  - Projects/Conecta-SENAI

prerequisites:             # Pré-requisitos
  - Knowledge/Concepts/variables
```

---

## 🔍 Navegação

### Dashboard
- [[INDEX|🏠 Painel Principal]] — Aqui agora

### Índices (MOCs)
- [[../../Knowledge/Technologies/INDEX|📚 Tecnologias]]
- [[../../Knowledge/Concepts/INDEX|🧠 Conceitos]]
- [[../../Knowledge/Code/INDEX|🧩 Código]]
- [[../../Knowledge/Resources/INDEX|📖 Recursos]]
- [[../../Projects/INDEX|🚀 Projetos]]

---

## ⚙️ Configuração & Plugins

### Plugins Necessários
- **Dataview** — Queries dinâmicas
- **Obsidian Tasks** — Gestão de tarefas
- **Templater** — Criar notas de templates
- **Calendar** — Notas diárias automáticas

### Plugins Opcionais
- **Excalidraw** — Diagramas
- **Graph Analysis** — Análise do grafo

---

## 🤖 Integração com AMEA

Este vault está 100% pronto para sua IA pessoal (AMEA).

### Capacidades Habilitadas:
- ✅ **Busca Semântica** — Embeddings em banco vetorial
- ✅ **Grafo de Conhecimento** — Relacionamentos tipados
- ✅ **RAG** — Recuperação contextual de informações
- ✅ **Multi-Agent** — AMEA entende a estrutura

### Campos que AMEA Usa:
- `type`, `id`, `category` — Categorização
- `confidence`, `source` — Qualidade da informação
- `dependencies`, `applied_in` — Relacionamentos
- `ai_generated`, `last_reviewed` — Metadados

---

## 📈 Manutenção

### Revisão Semanal
- [ ] Revisar notas estudadas
- [ ] Atualizar status
- [ ] Adicionar novos relacionamentos

### Revisão Mensal
- [ ] Validar links quebrados
- [ ] Atualizar MOCs
- [ ] Consolidar duplicatas

### Revisão Trimestral
- [ ] Analisar lacunas de conhecimento
- [ ] Refatorar estrutura
- [ ] Atualizar documentação

---

## 🆘 Troubleshooting

### Links Quebrados
Use a busca do Obsidian para encontrar referências:
1. Ctrl+P (ou Cmd+P)
2. Search
3. Procure `[[` para encontrar todos os links

### Frontmatter Inválido
Verifique:
- Não há tabs (só espaços)
- Datas no formato `YYYY-MM-DD`
- Valores entre aspas se tiverem caracteres especiais

### Notas Órfãs (Isoladas)
Veja no Graph View e adicione links:
- Qual domínio pertence?
- Qual index (MOC) deve conter?
- Qual projeto usa?

---

## 📚 Próximos Passos

1. **Explore:** Navegue pela nova estrutura
2. **Estude:** Leia as notas organizadas
3. **Escreva:** Crie novas notas seguindo padrões
4. **Integre:** Conecte com AMEA quando pronto

---

**Status:** Knowledge OS v1.0 ✅  
**Última Atualização:** 2026-07-04  
**Pronto para AMEA:** Sim ✅

---
