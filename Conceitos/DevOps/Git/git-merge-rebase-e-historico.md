---
type: concept
area: Conceitos
status: estavel
aliases: ["Merge vs rebase", "Squash merge", "Histórico linear", "Rebase"]
tags:
  - concept
  - git
  - devops
  - merge
  - historico
created: 2026-08-20
updated: 2026-08-20
---
> [!info] Conjunto: [[Fluxo Empresarial de Git|🏢 Fluxo Empresarial de Git]] · Relacionado: [[git-commits-e-mensagens|✍️ Commits]] · [[git-hotfix-e-recuperacao|🚑 Recuperação]]

# 🔀 Merge, rebase e histórico

> [!abstract] A decisão real
> As três estratégias produzem o **mesmo código**. O que muda é o **histórico** — e o histórico é a ferramenta de investigação de quando algo quebra. Escolher aqui é escolher o que você vai conseguir descobrir daqui a seis meses.

## 🧭 As três estratégias

### 1. Merge commit (`--no-ff`)

```
main    ──●───────────●──  ← commit de merge
           \         /
feature     ●──●──●──
```
Preserva todos os commits da branch **e** registra que ali houve uma integração.

- ✅ Histórico fiel ao que aconteceu; fácil reverter a feature inteira (`revert -m 1`)
- ❌ Histórico "cabeludo" com muitas branches paralelas; polui com commits de `wip`

### 2. Squash merge

```
main    ──●──●──●──  ← um commit só, com tudo da branch
feature (descartada)
```
Todos os commits da branch viram **um** commit na `main`.

- ✅ Histórico linear e legível: 1 PR = 1 commit = 1 unidade lógica
- ✅ `git log` da main vira a lista de features entregues
- ❌ Perde os passos intermediários (ficam no PR, não no Git)
- ❌ `bisect` fica mais grosso — o commit culpado pode ter 400 linhas

### 3. Rebase + fast-forward

```
main    ──●──●──●──●──  ← commits da branch reescritos por cima
```
Reaplica os commits da branch no topo da `main`, sem commit de merge.

- ✅ Histórico perfeitamente linear preservando commits individuais
- ❌ Reescreve hashes; exige commits limpos desde o começo
- ❌ Some o registro de que aquilo foi uma feature única

## 📊 Qual usar

| Situação | Estratégia |
|---|---|
| Feature branch → `main` (caso comum) | **Squash** |
| Branch com commits já bem escritos e atômicos | Rebase + ff |
| `release/*` ou `hotfix/*` → `main` | **Merge commit** (o registro da integração importa) |
| Atualizar sua branch com a `main` durante o trabalho | **Rebase** (`git pull --rebase`) |
| Branch compartilhada com outra pessoa | **Merge**, nunca rebase |

> [!success] Recomendação empresarial padrão
> **Squash na entrada e merge commit nas releases.** A `main` fica com uma linha por entrega — legível, revertível e alinhada com o PR. Basta uma disciplina em troca: o título do PR precisa seguir [[git-commits-e-mensagens|Conventional Commits]], porque é ele que vira a mensagem final.

## ⛔ A regra de ouro do rebase

> [!warning] Nunca rebaseie uma branch que outra pessoa já puxou
> Rebase **reescreve commits**: mesmo conteúdo, hash novo. Se alguém já tinha a versão antiga, o Git dela e o seu passam a contar histórias diferentes do mesmo trabalho — e a "correção" costuma ser um merge duplicando tudo. Rebase é seguro em uma branch que só você usa. Na `main` ou em branch compartilhada, é acidente.

Quando o rebase é inevitável numa branch já publicada (só sua):

```bash
git push --force-with-lease
```

`--force-with-lease` recusa o push se alguém tiver enviado algo que você não viu. `--force` puro sobrescreve o trabalho dessa pessoa sem avisar — **em repositório de empresa, deveria ser proibido por regra de proteção.**

## 🕵️ Por que o histórico importa

O histórico só serve se for consultável. Estas ferramentas dependem dele:

```bash
# quem mudou esta linha e por quê
git blame -L 40,60 arquivo.py

# encontrar o commit que quebrou, por busca binária
git bisect start
git bisect bad                 # o atual está quebrado
git bisect good v1.4.0         # esta versão funcionava
# o Git faz checkout no meio; você testa e responde good/bad até achar

# ver só o que entrou entre duas versões
git log --oneline v1.4.0..v1.5.0

# encontrar quando um trecho de código apareceu ou sumiu
git log -S "cosine_similarity" --oneline
```

> [!important] `bisect` é o argumento mais forte a favor de commits pequenos
> Ele encontra o commit exato que introduziu um bug em log(n) passos — 10 tentativas em mil commits. Mas o que ele devolve é **um commit**: se aquele commit tiver 2.000 linhas de "várias coisas", você descobriu o dia, não a causa. Commit atômico ([[git-commits-e-mensagens|✍️]]) é o que transforma `bisect` de curiosidade em ferramenta.

## 🧯 Conflito de merge

```bash
git switch feat/minha-branch
git fetch origin
git rebase origin/main
# conflito → editar os arquivos marcados
git add <arquivos-resolvidos>
git rebase --continue
# desistiu no meio?
git rebase --abort
```

Reduzir conflito é questão de hábito, não de ferramenta:

- Rebaseie na `main` **todo dia**, não no fim
- Branch curta ([[git-convencao-de-branches|🏷️]])
- Duas pessoas na mesma área do código → combinar antes, não descobrir no merge
- Formatador automático: metade dos conflitos reais é reformatação, não lógica

## 🧩 Conceitos relacionados

- [[Fluxo Empresarial de Git|🏢 Fluxo Empresarial de Git]]
- [[git-commits-e-mensagens|✍️ Commits e mensagens]]
- [[git-hotfix-e-recuperacao|🚑 Hotfix e recuperação]]
- [[git-pull-request-e-code-review|🔍 Pull Request]]
