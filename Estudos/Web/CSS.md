---
type: tech
area: Estudos
status: aprendendo
tecnologia: CSS
tags:
  - tech
  - estudo
  - frontend
created: 2026-06-30
updated: 2026-06-30
---
# CSS

> [!tip] Status
> 🟢 Confortável

## 📝 Resumo

CSS (Cascading Style Sheets) define a aparência visual de páginas HTML: cores, layout, espaçamento, responsividade e animações.

## 🧠 Conceitos principais

- **Seletores**: classe, id, elemento, pseudo-classes (`:hover`), pseudo-elementos (`::before`)
- **Box model**: `margin`, `border`, `padding`, `content`
- **Layout**: `Flexbox`, `Grid`, `position`
- **Responsividade**: `media queries`, unidades relativas (`rem`, `%`, `vw/vh`)
- **Especificidade e cascata**
- **Variáveis CSS**: `:root { --cor-principal: #000 }`
- **Pré-processadores**: Sass/SCSS (opcional)

## 💻 Exemplos

```css
.card {
  display: flex;
  flex-direction: column;
  gap: 1rem;
  padding: 1.5rem;
  border-radius: 0.5rem;
}

@media (max-width: 600px) {
  .card {
    padding: 1rem;
  }
}
```

## 🔗 Links úteis

- [MDN - CSS](https://developer.mozilla.org/pt-BR/docs/Web/CSS)
- [CSS-Tricks - A Complete Guide to Flexbox](https://css-tricks.com/snippets/css/a-guide-to-flexbox/)
- [Grid Garden (jogo para praticar Grid)](https://cssgridgarden.com/)

## ✅ Checklist de aprendizado

- [x] Box model
- [x] Flexbox
- [ ] CSS Grid
- [ ] Responsividade (mobile-first)
- [ ] Animações e transições

## 🗒️ Notas pessoais


## 🔗 Veja também

- [[HTML|HTML]]
- [[React|React]]
