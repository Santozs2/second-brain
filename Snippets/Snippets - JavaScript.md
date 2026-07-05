---
type: snippet
area: Snippets
status: estavel
linguagem: JavaScript
tags:
  - snippet
created: 2026-06-30
updated: 2026-06-30
---
# 🧩 Snippets de JavaScript

> [!tip] Como usar
> Cada `###` abaixo é um snippet independente. Adicione novos no final usando o mesmo padrão.

## Debounce de função

```js
function debounce(fn, delay = 300) {
  let timer;
  return (...args) => {
    clearTimeout(timer);
    timer = setTimeout(() => fn(...args), delay);
  };
}
```

**Quando usar:** evitar disparar uma função várias vezes seguidas (ex: campo de busca enquanto o usuário digita).

#snippet #javascript

---

## Fetch com tratamento de erro

```js
async function getDados(url) {
  try {
    const res = await fetch(url);
    if (!res.ok) throw new Error(`Erro HTTP: ${res.status}`);
    return await res.json();
  } catch (erro) {
    console.error("Falha ao buscar dados:", erro);
    return null;
  }
}
```

**Quando usar:** padrão básico para consumir uma API com segurança.

#snippet #javascript

---

## Copiar objeto/array sem mutar o original

```js
const copiaArray = [...arrayOriginal];
const copiaObjeto = { ...objetoOriginal };
```

**Quando usar:** sempre que precisar alterar dados sem afetar o estado original (muito comum em React).

#snippet #javascript

## Veja também

- [[JavaScript|JavaScript]]
- [[Snippets|Todos os snippets]]
