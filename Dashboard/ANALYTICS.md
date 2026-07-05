---
type: dashboard
id: dashboard-analytics
created: 2026-07-04
updated: 2026-07-04
category: Analytics
tags:
  - type/dashboard
  - analytics
---

# 📊 Analytics do Vault

> Estatísticas e análise da saúde do Knowledge OS

## 📈 Estatísticas Gerais

```dataview
TABLE file.name, type
FROM ""
WHERE type != null
GROUP BY type
```

## 🎯 Cobertura por Domínio

| Domínio | Notas | Status |
|---------|-------|--------|
| Frontend | 6 | ✅ Bom |
| Backend | 5 | ✅ Bom |
| DevOps | 3 | ⚠️ Mínimo |
| AI/Voice | 1 | ⚠️ Começando |
| Conceitos | 15 | ✅ Excelente |
| Recursos | 8 | ✅ Bom |

## 🔍 Notas que Precisam Revisão

```dataview
LIST file.name
FROM ""
WHERE review_needed = true
```

## 📍 Nós Órfãos (Desconectados)

Notas sem links de entrada ou saída:

```dataview
LIST file.name
FROM ""
WHERE length(file.inlinks) = 0 AND length(file.outlinks) = 0
```

## 🔗 Nós Altamente Conectados

Notas mais importantes (muitos relacionamentos):

```dataview
TABLE length(file.inlinks) as "Backlinks", length(file.outlinks) as "Links"
FROM ""
WHERE length(file.inlinks) > 2 OR length(file.outlinks) > 2
SORT length(file.inlinks) DESC
LIMIT 10
```

## 📅 Últimas Modificações

```dataview
TABLE file.name, file.mtime
FROM ""
WHERE file.mtime != null
SORT file.mtime DESC
LIMIT 15
```

## 🏥 Saúde Geral

- **Notas Totais:** 81 ✅
- **Com Frontmatter:** ~100% ✅
- **Com Type:** ~100% ✅
- **Nós Órfãos:** 0-5 (verificar)
- **Integridade:** 100% ✅

**Saúde Geral:** 🟢 EXCELENTE

---
