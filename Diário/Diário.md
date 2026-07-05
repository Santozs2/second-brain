---
type: journal
area: Diário
status: ativo
tags:
  - journal
created: 2026-06-30
updated: 2026-06-30
---
# 📔 Diário

> [!tip] Sobre esta página
> Use o ícone do **Calendar** na barra lateral para abrir ou criar a nota de hoje a partir do [[Template - Diário]]. Esta página só lista as últimas entradas.

```dataview
TABLE data AS "Data"
FROM "Diário" AND -"Diário/Diário"
SORT data DESC
LIMIT 14
```

## Veja também

- [[Home|Painel Principal]]
