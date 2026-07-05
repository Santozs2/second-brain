---
type: snippet
tags:
  - snippet
linguagem: CSS
created: 2026-06-30
updated: 2026-06-30
status: estavel
---

# 🧩 Snippets de CSS

> [!tip] Como usar
> Cada `###` abaixo é um snippet independente. Adicione novos no final usando o mesmo padrão.

## Centralizar com Flexbox

```css
.container {
  display: flex;
  align-items: center;
  justify-content: center;
}
```

**Quando usar:** centralizar conteúdo vertical e horizontalmente — o caso mais comum de centralização.

#snippet #css

---

## Grid responsivo automático

```css
.grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 1rem;
}
```

**Quando usar:** criar grids de cards que se ajustam sozinhos ao tamanho da tela, sem media queries.

#snippet #css

---

## Reset básico

```css
*, *::before, *::after {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}
```

**Quando usar:** no início de qualquer projeto, para eliminar inconsistências de espaçamento entre navegadores.

#snippet #css

## Veja também

- [[CSS|CSS]]
- [[Snippets|Todos os snippets]]
