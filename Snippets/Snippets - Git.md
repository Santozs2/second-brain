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

---

## Atualizar a branch com o que entrou na main

```bash
git fetch origin && git rebase origin/main
```

**Quando usar:** todo dia, enquanto a feature está aberta. Rebasear cedo transforma um conflito gigante no fim em conflitinhos diários.

#snippet #git

---

## Force push seguro

```bash
git push --force-with-lease
```

**Quando usar:** sempre que precisar sobrescrever a sua branch remota depois de um rebase. Diferente do `--force`, ele recusa o push se alguém tiver enviado algo que você ainda não viu — e nunca apaga o trabalho de outra pessoa em silêncio.

#snippet #git

---

## Commitar só um pedaço do arquivo

```bash
git add -p
```

**Quando usar:** quando você mexeu em duas coisas no mesmo arquivo e quer commits separados. O Git pergunta pedaço por pedaço o que entra.

#snippet #git

---

## Recuperar trabalho depois de um reset --hard

```bash
git reflog
git reset --hard <sha-de-antes-do-erro>
```

**Quando usar:** quando parece que você perdeu commits. O reflog guarda todos os estados do `HEAD` por ~90 dias — commit "perdido" quase sempre está lá.

#snippet #git

---

## Desfazer um Pull Request inteiro já mergeado

```bash
git revert -m 1 <sha-do-commit-de-merge>
```

**Quando usar:** quando uma feature já na `main` precisa sair. O `-m 1` diz qual lado do merge é o "principal" (a `main`). Use `revert`, nunca `reset`, em branch pública.

#snippet #git

---

## Achar o commit que quebrou

```bash
git bisect start
git bisect bad
git bisect good v1.4.0
# testa, responde good/bad até achar
git bisect reset
```

**Quando usar:** quando algo funcionava e você não sabe quando parou. Busca binária: mil commits em ~10 testes.

#snippet #git

---

## Deletar branch local e remota depois do merge

```bash
git branch -d feat/minha-branch
git push origin --delete feat/minha-branch
git fetch --prune
```

**Quando usar:** ao fechar um PR. O `--prune` limpa as referências locais de branches que já sumiram no servidor.

#snippet #git

---

## Criar tag anotada de release

```bash
git tag -a v1.4.2 -m "Release 1.4.2"
git push origin v1.4.2
```

**Quando usar:** ao marcar uma versão. Sempre `-a`: tag leve não guarda autor, data nem mensagem.

#snippet #git

---

## Guardar o trabalho no meio para trocar de contexto

```bash
git stash push -m "meio do formulário"
git stash list
git stash pop
```

**Quando usar:** quando aparece um bug urgente e você está no meio de outra coisa.

#snippet #git

## Veja também

- [[Git|Git]]
- [[Fluxo Empresarial de Git|🏢 Fluxo Empresarial de Git]]
- [[Git Flow|Git Flow]]
- [[Snippets|Todos os snippets]]
