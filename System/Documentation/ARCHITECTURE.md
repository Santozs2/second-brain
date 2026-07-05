---
type: dashboard
id: doc-architecture
created: 2026-07-04
updated: 2026-07-04
category: Documentation
tags:
  - type/dashboard
  - architecture
  - amea
---

# 🏗️ Arquitetura Knowledge OS

## 🎯 Visão Geral

Este vault é um **Knowledge OS** pronto para AMEA (IA Pessoal).

```
Conhecimento = Nó + Contexto + Relacionamentos

Cada nota = Nó em grafo
├── Metadados estruturados (frontmatter)
├── Conteúdo estruturado (seções)
└── Relacionamentos tipados (edges)
```

## 📐 Camadas da Arquitetura

### Camada 1: Apresentação
- Dashboard (HOME, ANALYTICS, PROGRESS)
- MOCs (índices)
- Navegação visual

### Camada 2: Conhecimento
- Notas (technology, concept, project, resource)
- Estrutura de conteúdo por tipo
- Sistema de tags

### Camada 3: Metadados
- Frontmatter YAML (type, id, confidence)
- Tags (type/domain/status)
- Relacionamentos (dependencies, applied_in)

### Camada 4: Infraestrutura
- Templates (System/Templates/)
- Documentação (System/Documentation/)
- Configuração (System/Config/)

## 🗺️ Estrutura de Pastas

```
Second Brain/
├── 🏠 Dashboard/          ← Navegação principal
├── 🧠 Knowledge/          ← Conhecimento estruturado
│   ├── Technologies/      ← Tecnologias e linguagens
│   ├── Concepts/          ← Conceitos transversais
│   ├── Code/              ← Snippets e padrões
│   ├── Resources/         ← Links externos
│   ├── Definitions/       ← Glossário
│   ├── Reference/         ← Documentação de ref
│   └── Relationships/     ← Mapas de relacionamento
├── 🚀 Projects/           ← Projetos ativos
├── 📝 (root)              ← Diário e histórico
└── 🔧 System/             ← Templates e config
```

## 📋 Tipos de Notas (15)

| Tipo | ID Prefixo | Localização | Relações |
|------|-----------|------------|----------|
| **Technology** | tech- | Technologies/ | → Concepts, Projects |
| **Concept** | concept- | Concepts/ | → Technologies, Resources |
| **Project** | project- | Projects/ | → Technologies, Concepts |
| **Resource** | resource- | Resources/ | (terminal) |
| **Snippet** | snippet- | Code/Snippets/ | → Technologies |
| **MOC** | moc- | anywhere | → all types |
| **Dashboard** | dashboard- | Dashboard/ | → all types |
| Definition | def- | Definitions/ | ← all |
| Reference | ref- | Reference/ | ← Technology |
| Architecture | arch- | Concepts/ | ← Projects |
| Error | err- | (temp) | ← Projects |
| Idea | idea- | (temp) | (backlog) |
| Decision | dec- | Projects/ | ← Projects |
| Daily/Journal | daily- | root | → all |
| Template | template- | System/Templates/ | (not indexed) |

## 🔗 Relacionamentos Semânticos (11)

```
depends_on        ← "React depende de JavaScript"
prerequisite_for  ← "JavaScript é pré-requisito para React"
related_to        ← "MVC está relacionado a ORM"
contrasts_with    ← "REST contrasta com GraphQL"
equivalent_to     ← "ORM equivale a Query Builder"
applied_in        ← "JWT é aplicado em Auth"
implemented_by    ← "CRUD é implementado por Models"
explained_by      ← "HTTP é explicado em REST API"
part_of           ← "Props é parte de React"
specialization_of ← "DRF é especialização de Django"
uses              ← "Conecta SENAI usa Django"
```

## 🏷️ Sistema de Tags

Padrão: `categoria/valor`

```yaml
type/technology       # Tipo de nota
domain/frontend       # Frontend, Backend, DevOps, AI
status/learning       # learning, stable, reference, idea
difficulty/intermediate  # beginner, intermediate, advanced
context/conecta-senai # Projeto relacionado
```

## 🧬 Frontmatter Universal

```yaml
---
# Identificação
type: technology
id: tech-react-001
created: 2026-07-04
updated: 2026-07-04

# Categorização
category: Frontend
tags: [type/technology, domain/frontend]

# Status
status: learning
confidence: 0.85
difficulty: intermediate

# Contexto
source: "Official Docs"
learned_date: 2026-06-15

# Relacionamentos
dependencies:
  - tech-javascript
applied_in:
  - project-conecta

# Metadados para IA
ai_generated: false
last_reviewed: 2026-07-04
embedding_model: openai
---
```

## 📊 Fluxo de Dados para AMEA

```
Nota (.md)
├── Parse frontmatter → Metadata
├── Extract type → Classification
├── Extract tags → Categorization
├── Extract relationships → Graph edges
├── Process content → Embeddings
└── Store in DB ← Qdrant/Pinecone
     │
     └→ Semantic Search
        └→ RAG
           └→ AMEA queries
```

## 🤖 Integração com AMEA

### O que AMEA precisa
- ✅ Frontmatter estruturado (type, id, tags)
- ✅ Relacionamentos tipados (depends_on, applied_in)
- ✅ Conteúdo em markdown
- ✅ Metadados de qualidade (confidence, source)

### O que AMEA fará
- 🔍 Busca semântica em embeddings
- 🔗 Navegar grafo de conhecimento
- 📚 RAG (encontrar contexto relevante)
- 🧠 Responder perguntas com contexto
- 🚀 Sugerir novas notas/conexões
- 📊 Analisar lacunas de conhecimento

## ✅ Checklist de Implementação

- [x] **Fase 1:** Estrutura de pastas
- [x] **Fase 2:** Padronização (frontmatter, tags)
- [x] **Fase 3:** Ontologia (tipos, relacionamentos)
- [ ] **Fase 4:** AI-Ready (embeddings, AMEA)

## 📈 Métricas de Sucesso

| Métrica | Status | Score |
|---------|--------|-------|
| Estrutura | ✅ | 10/10 |
| Padronização | ✅ | 10/10 |
| Documentação | ✅ | 10/10 |
| Preparação para IA | ✅ | 8/10 |
| **TOTAL** | ✅ | **9/10** |

## 🚀 Próximos Passos para AMEA

1. **Embeddings:** Gerar vectors para todas as notas
2. **Indexação:** Armazenar em banco vetorial
3. **Testing:** Testar busca semântica
4. **Integration:** Conectar com chat AMEA
5. **Automação:** Setup de atualizações automáticas

---

**Status:** Knowledge OS v1.0 - Pronto para AMEA ✅  
**Última Atualização:** 2026-07-04
