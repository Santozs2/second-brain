---
type: moc
area: Conceitos
status: estavel
aliases: ["Fluxo empresarial de Git", "Git empresarial", "Workflow de Git", "Fluxo de trabalho Git"]
tags:
  - concept
  - moc
  - git
  - devops
  - processo
created: 2026-08-20
updated: 2026-08-20
---
# 🏢 Fluxo Empresarial de Git

> [!abstract] O que é este conjunto de notas
> Como equipes de verdade usam Git no dia a dia: branches, features, pull requests, review, merge, release e hotfix. Não é sobre **comandos** ([[Git|Git]] já cobre isso) — é sobre o **processo** em volta deles, que é o que separa um repositório pessoal de um repositório onde 10 pessoas mexem sem quebrar produção.

## 🎯 O que muda quando o repositório é de uma empresa

| No projeto pessoal | Na empresa |
|---|---|
| Você lembra por que fez aquilo | Alguém vai ler seu commit daqui a 2 anos, sem você por perto |
| Quebrou? Você conserta | Quebrou? Está fora do ar para clientes pagantes |
| Commit "ajustes" resolve | O histórico é evidência: auditoria, `bisect`, rollback |
| Merge quando quiser | Merge passa por review, CI e permissão |
| Uma pessoa, um contexto | Onboarding: alguém novo precisa entender o fluxo em um dia |

> [!important] O histórico do Git é documentação, não burocracia
> Toda regra deste conjunto de notas existe para responder uma pergunta futura: *"quem mudou isso, quando, por quê, e como eu volto atrás?"*. Um fluxo que não responde essas quatro perguntas é cerimônia sem valor — e um que responde compensa cada minuto gasto.

## 🔄 O ciclo diário

```
git switch main && git pull          1. sincroniza
        ↓
git switch -c feat/PROJ-142-...      2. branch a partir da main
        ↓
commits pequenos e atômicos          3. trabalho
        ↓
git push -u origin <branch>          4. publica
        ↓
abre Pull Request                    5. pede review
        ↓
CI roda: lint + testes + build       6. o robô revisa primeiro
        ↓
review humano → ajustes              7. a pessoa revisa depois
        ↓
merge (squash) na main               8. entra
        ↓
deploy + branch deletada             9. fecha o ciclo
```

## 🗺️ As notas deste conjunto

| Nota | Responde |
|---|---|
| [[git-modelos-de-branching\|🌳 Modelos de branching]] | Git Flow, GitHub Flow, Trunk-Based — qual escolher e por quê |
| [[git-convencao-de-branches\|🏷️ Convenção de branches]] | Como nomear, quanto tempo pode viver, quando deletar |
| [[git-commits-e-mensagens\|✍️ Commits e mensagens]] | Conventional Commits, commit atômico, o corpo da mensagem |
| [[git-pull-request-e-code-review\|🔍 Pull Request e code review]] | Tamanho do PR, template, CODEOWNERS, como revisar |
| [[git-merge-rebase-e-historico\|🔀 Merge, rebase e histórico]] | Squash × merge commit × rebase, e a regra de ouro do rebase |
| [[git-protecao-e-permissoes\|🔒 Proteção e permissões]] | Branch protection, checks obrigatórios, segredos vazados |
| [[git-release-e-versionamento\|🚀 Release e versionamento]] | SemVer, tags, changelog, code freeze, rollback |
| [[git-hotfix-e-recuperacao\|🚑 Hotfix e recuperação]] | Produção quebrada, `revert`, `reflog`, `bisect` |
| [[git-fluxo-aplicado-tcc\|🎓 Fluxo aplicado ao TCC]] | O mesmo processo, no tamanho de um grupo de 4 |

## ⚖️ As seis regras que quase toda empresa tem

1. **A `main` está sempre deployável.** Se não está, o time inteiro para até estar.
2. **Ninguém commita direto na `main`.** Tudo entra por Pull Request.
3. **Branch é curta e descartável.** Nasce da `main`, vive dias, morre no merge.
4. **CI verde é condição de merge**, não sugestão.
5. **Ninguém aprova o próprio PR.** Pelo menos um par revisa.
6. **Segredo não entra no repositório.** Nunca, nem em branch, nem "só para testar".

> [!warning] Processo demais mata tanto quanto processo de menos
> Git Flow completo (`develop` + `release/*` + `hotfix/*`) num time de 3 pessoas que faz deploy toda semana é overhead puro: mais merge, mais conflito, mais espera. As seis regras acima cabem em qualquer tamanho; o resto é proporcional ao risco. **Escolha o fluxo pela frequência de deploy e pelo custo de um erro em produção** — não pelo tamanho da empresa nem pelo que está na moda.

## 🧩 Conceitos relacionados

- [[Git|Git]] — os comandos
- [[Git Flow|Git Flow]] — o modelo específico de 2010
- [[CI-CD|CI/CD]] — o robô que roda a cada push
- [[Snippets - Git|🧩 Snippets de Git]]

## Veja também

- [[Conceitos|🧠 Conceitos]]
- [[Home|Painel Principal]]
