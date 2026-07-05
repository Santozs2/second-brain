---
type: dashboard
id: dashboard-progress
created: 2026-07-04
updated: 2026-07-04
category: Analytics
tags:
  - type/dashboard
  - learning-progress
---

# 📈 Progresso de Aprendizado

> Acompanhamento do seu progresso em tecnologias

## 🎓 Tecnologias por Nível

### 🟢 Dominada (90%+)
- Estrutura sólida
- Aplicada em projetos
- Conhecimento profundo

```dataview
LIST file.name
FROM "Knowledge/Technologies"
WHERE confidence >= 0.9 AND status = "stable"
```

### 🟡 Aprendendo (30-90%)
- Conceitos compreendidos
- Em prática
- Ainda aprendendo detalhes

```dataview
LIST file.name
FROM "Knowledge/Technologies"
WHERE confidence BETWEEN 0.3 0.9 AND status = "learning"
```

### 🔵 Iniciante (0-30%)
- Começando agora
- Conhecimento superficial
- Precisa praticar mais

```dataview
LIST file.name
FROM "Knowledge/Technologies"
WHERE confidence < 0.3 AND status = "learning"
```

## 📊 Distribuição por Tipo

| Status | Quantidade |
|--------|-----------|
| Dominada | 0 |
| Aprendendo | 12 |
| Referência | 0 |
| Ideia | 0 |
| Draft | 0 |

## 🎯 Próximas Metas

- [ ] React: Dominar hooks avançados
- [ ] Django: Completar DRF
- [ ] Docker: Usar em projeto real
- [ ] TypeScript: Aplicar em Next.js

## 📚 Caminhos de Aprendizado Recomendados

### Path 1: Full-Stack
1. HTML → CSS → JavaScript
2. React → Next.js
3. Python → Django → Django REST Framework
4. PostgreSQL

### Path 2: Backend
1. Python fundamentals
2. Django basics
3. Django REST Framework
4. Database design
5. Docker & DevOps

### Path 3: Frontend
1. HTML → CSS
2. JavaScript fundamentals
3. React
4. TypeScript
5. Next.js

---
