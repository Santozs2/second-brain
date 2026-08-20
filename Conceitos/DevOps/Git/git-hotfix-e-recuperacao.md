---
type: concept
area: Conceitos
status: estavel
aliases: ["Hotfix", "git reflog", "git bisect", "Recuperar commit perdido", "Cherry-pick"]
tags:
  - concept
  - git
  - devops
  - incidente
  - recuperacao
created: 2026-08-20
updated: 2026-08-20
---
> [!info] Conjunto: [[Fluxo Empresarial de Git|🏢 Fluxo Empresarial de Git]] · Relacionado: [[git-release-e-versionamento|🚀 Release]] · [[git-merge-rebase-e-historico|🔀 Histórico]]

# 🚑 Hotfix e recuperação

> [!abstract] Duas situações diferentes
> **Hotfix**: produção está quebrada e o conserto não pode esperar o fluxo normal. **Recuperação**: alguém errou no Git e acha que perdeu trabalho — quase sempre não perdeu.

---

## 🔥 Fluxo de hotfix

```
tag v2.4.1 (o que está em produção)
     │
     └─► hotfix/PROJ-203-pagamento-500
              │  correção mínima + teste que prova
              ▼
         PR expresso → CI → 1 aprovação
              │
              ├─► main → tag v2.4.2 → deploy
              └─► back-merge para develop/main (se houver duas)
```

> [!warning] Hotfix nasce da tag de produção, não da `main`
> É o erro mais comum sob pressão. A `main` já andou desde o último deploy: ela tem funcionalidades ainda não liberadas. Ramificar dali leva tudo isso junto para produção, no meio de um incidente — trocando um problema conhecido por um desconhecido. **Ramifique da tag exata que está no ar.**

### Regras do hotfix

| Regra | Motivo |
|---|---|
| Correção **mínima** | Não é hora de refatorar nem de "já que estou aqui" |
| Com teste que reproduz a falha | Sem ele, volta na próxima release |
| PR mesmo assim, com revisor único | Pressa não justifica ninguém olhar |
| CI roda igual | O único gate que não se pula |
| Back-merge no mesmo dia | Senão a correção some na próxima release |

> [!danger] O hotfix que não volta para a linha principal
> Clássico: corrige em produção, todo mundo comemora, ninguém faz o back-merge. Duas semanas depois a release nova sobe **sem a correção** e o mesmo incidente acontece de novo — agora com o time convencido de que "já tinha sido resolvido". Back-merge é parte do hotfix, não tarefa seguinte.

### Cherry-pick

```bash
git switch main
git cherry-pick <sha-do-hotfix>
```

Copia um commit específico para outra branch. Útil para levar o hotfix de volta — e perigoso como hábito: cria commits duplicados com hashes diferentes, e o Git deixa de saber que são a mesma mudança. Use pontualmente, nunca como estratégia de integração.

---

## 🧰 Recuperação — o que fazer quando dá errado

### "Commitei na branch errada"
```bash
git reset HEAD~1              # desfaz o commit, mantém as alterações
git stash
git switch -c feat/branch-certa
git stash pop
```

### "Preciso desfazer algo que já está na `main`"
```bash
git revert <sha>              # commit novo que desfaz — seguro em branch pública
git revert -m 1 <sha-merge>   # desfaz um merge inteiro (PR completo)
```

### "Fiz reset --hard e perdi tudo"
```bash
git reflog                    # lista TODOS os estados por onde o HEAD passou
git reset --hard <sha>        # volta para o estado antes do erro
```

> [!success] `reflog` é a rede de segurança que quase ninguém conhece
> O Git registra localmente cada movimento do `HEAD` — commit, reset, rebase, checkout — por 90 dias por padrão. Commit "perdido" por `reset --hard`, rebase mal resolvido ou branch deletada quase sempre está lá inteiro. **Antes de aceitar que o trabalho sumiu, rode `git reflog`.** A ressalva importante: é local. O que nunca foi commitado (só editado) não está no reflog — está perdido de verdade.

### "Deletei a branch sem mergear"
```bash
git reflog                    # ache o último commit dela
git switch -c feat/recuperada <sha>
```

### "Preciso guardar o trabalho no meio para trocar de contexto"
```bash
git stash push -m "meio do formulário"
git stash list
git stash pop                 # aplica e remove da pilha
```

### "Qual commit quebrou isso?"
```bash
git bisect start
git bisect bad                # o estado atual está quebrado
git bisect good v2.4.0        # aqui funcionava
# testa, responde good/bad, repete
git bisect reset              # ao terminar
```

Busca binária no histórico: 1.000 commits em ~10 testes. Ver [[git-merge-rebase-e-historico|🔀 Histórico]].

### "Quem escreveu esta linha e por quê?"
```bash
git blame -L 40,60 arquivo.py
git show <sha>                # o commit inteiro, com a mensagem
git log -S "texto" --oneline  # quando esse trecho apareceu ou sumiu
```

---

## 📄 Depois do incidente: post-mortem

Um hotfix bem-feito termina em documento, não em alívio:

- **O que quebrou** e desde quando (o `bisect` responde)
- **Por que passou** pelo review e pelo CI
- **Qual teste** foi adicionado para não repetir
- **O que muda no processo** — se a resposta for "ter mais atenção", nada mudou

> [!important] A pergunta certa não é "quem errou"
> É *"por que o processo deixou passar?"*. Todo incidente que chega em produção passou por review, CI e deploy — a falha é do conjunto, e é ali que a correção rende. Post-mortem que procura culpado ensina o time a esconder erro, que é o oposto do que se quer.

## 🧩 Conceitos relacionados

- [[Fluxo Empresarial de Git|🏢 Fluxo Empresarial de Git]]
- [[git-release-e-versionamento|🚀 Release e versionamento]]
- [[git-merge-rebase-e-historico|🔀 Merge, rebase e histórico]]
- [[Snippets - Git|🧩 Snippets de Git]]
