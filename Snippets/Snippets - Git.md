---
type: snippet
area: Snippets
status: estavel
linguagem: Git
tags:
  - snippet
created: 2026-06-30
updated: 2026-06-30
---
# 🧩 Snippets de Git

> [!tip] Como usar
> Cada `###` abaixo é um snippet independente. Adicione novos no final usando o mesmo padrão.

## Desfazer o último commit (mantendo as alterações)

```bash
git reset --soft HEAD~1
```

**Quando usar:** quando você commitou cedo demais e quer ajustar antes de commitar de novo.

#snippet #git

---

## Descartar alterações locais de um arquivo

```bash
git checkout -- nome-do-arquivo.js
```

**Quando usar:** quando você quer voltar um arquivo específico para o estado do último commit.

#snippet #git

---

## Criar e trocar de branch em um comando

```bash
git checkout -b feature/nova-funcionalidade
```

**Quando usar:** ao começar a trabalhar em uma nova funcionalidade isolada da `main`.

#snippet #git

## Veja também

- [[Git|Git]]
- [[Git Flow|Git Flow]]
- [[Snippets|Todos os snippets]]
