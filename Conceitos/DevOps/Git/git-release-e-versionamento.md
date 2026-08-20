---
type: concept
area: Conceitos
status: estavel
aliases: ["SemVer", "Versionamento semântico", "Release", "Tags no Git", "Changelog"]
tags:
  - concept
  - git
  - devops
  - release
  - versionamento
created: 2026-08-20
updated: 2026-08-20
---
> [!info] Conjunto: [[Fluxo Empresarial de Git|🏢 Fluxo Empresarial de Git]] · Relacionado: [[git-commits-e-mensagens|✍️ Commits]] · [[git-hotfix-e-recuperacao|🚑 Hotfix]]

# 🚀 Release e versionamento

> [!abstract] O que é uma release
> Um ponto do histórico marcado, nomeado e reproduzível. Não é "o código que está no ar" — é **um commit específico que dá para reconstruir byte a byte**, meses depois, para investigar um bug ou voltar atrás.

## 🔢 SemVer — versionamento semântico

```
   2  .  4  .  1
   │     │     └── PATCH: correção compatível
   │     └──────── MINOR: funcionalidade nova, compatível
   └────────────── MAJOR: quebra compatibilidade
```

| Mudança | Versão | Vem do commit |
|---|---|---|
| Corrigiu bug sem mudar contrato | `2.4.0` → `2.4.1` | `fix:` |
| Adicionou endpoint novo | `2.4.1` → `2.5.0` | `feat:` |
| Removeu ou mudou um campo da API | `2.5.0` → `3.0.0` | `feat!:` / `BREAKING CHANGE:` |

Pré-lançamentos: `3.0.0-beta.1`, `3.0.0-rc.2`.

> [!important] SemVer é uma promessa a quem consome, não um número bonito
> A pergunta que decide o dígito é sempre a mesma: **quem usa isso precisa mudar alguma coisa?** Se sim, é major — mesmo que a mudança tenha sido "só" renomear um campo do JSON. Subir minor numa quebra de contrato é o jeito mais rápido de perder a confiança de quem integra com você.

## 🏷️ Tags

```bash
# tag anotada (sempre esta, em release)
git tag -a v2.4.1 -m "Release 2.4.1"
git push origin v2.4.1

# ver o que entrou desde a última
git log --oneline v2.4.0..v2.4.1

# voltar ao código exato de uma versão
git checkout v2.4.0
```

> [!warning] Tag leve × tag anotada
> `git tag v2.4.1` cria uma tag **leve**: um apelido para o commit, sem autor, sem data, sem mensagem, e que alguns comandos ignoram. `-a` cria um objeto de verdade no repositório, com metadados e assinável. Em release, use sempre `-a` — a diferença aparece na auditoria, quando alguém pergunta quem publicou aquela versão.

## 📦 Fluxo de release

### Sem release branch (deploy contínuo)
```
main ──●──●──●──●──► tag v2.5.0 → deploy
```
Marca-se a tag no commit que foi para produção. É o modelo do [[git-modelos-de-branching|GitHub Flow]] e serve para a maioria das aplicações web.

### Com release branch (estabilização)
```
main    ──●──●──●──────────────●──  ← back-merge
              \               /
release/2.5    ●──●──●──► v2.5.0
                  ↑ só correções
```
A branch congela o escopo: dali em diante, só entra correção. O trabalho novo continua na `main` sem esperar.

> [!tip] Release branch existe para desacoplar "parar de adicionar" de "parar de trabalhar"
> Sem ela, congelar a versão congela o time inteiro. Com ela, quem está estabilizando corrige na `release/*` enquanto o resto segue na `main`. Se o seu time faz deploy toda semana e nunca precisa disso, é sinal de que você não precisa da branch — precisa continuar como está.

## 📜 Changelog

Com [[git-commits-e-mensagens|Conventional Commits]], o changelog é gerado:

```bash
npx standard-version        # bump + CHANGELOG.md + tag
npx semantic-release        # o mesmo, dentro do CI, sem passo manual
```

Escrito à mão, o changelog envelhece em duas releases. Gerado, ele é subproduto de uma disciplina que o time já tem.

## ❄️ Code freeze

Período antes de uma entrega crítica em que só entra correção de bug — típico de véspera de evento, fechamento fiscal ou auditoria.

- Anunciado com data e hora, não "a partir de agora"
- Implementado por regra de proteção, não por confiança
- Com critério explícito de exceção e quem pode autorizar

## 🎚️ Deploy ≠ release

Duas decisões que costumam ser confundidas:

| | Deploy | Release |
|---|---|---|
| O quê | Código vai para o servidor | Funcionalidade fica visível ao usuário |
| Risco | Técnico | De produto |
| Reversão | Redeploy da versão anterior | Desligar a feature flag |

Com feature flags dá para deployar dez vezes por dia e liberar a funcionalidade quando o produto quiser — **o Git para de ser o gargalo da decisão de negócio.**

## ⏪ Rollback

| Situação | Ação |
|---|---|
| Deploy quebrou, código está ok | Redeploy da tag anterior |
| Commit específico causou o problema | `git revert <sha>` |
| Um PR inteiro precisa sair | `git revert -m 1 <sha-do-merge>` |
| Feature nova causando problema | Desligar a flag (segundos, sem deploy) |

> [!danger] `git reset` nunca em branch pública
> `reset` reescreve a história; `revert` cria um commit novo que desfaz. Em `main`, só `revert` — ele preserva o registro de que o erro existiu e foi corrigido, que é exatamente o que a auditoria e o post-mortem precisam ver.

## 🧩 Conceitos relacionados

- [[Fluxo Empresarial de Git|🏢 Fluxo Empresarial de Git]]
- [[git-hotfix-e-recuperacao|🚑 Hotfix e recuperação]]
- [[git-commits-e-mensagens|✍️ Commits e mensagens]]
- [[CI-CD|CI/CD]]
