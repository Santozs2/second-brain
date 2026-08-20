---
type: concept
area: Conceitos
status: estavel
aliases: ["Convenção de branches", "Nomenclatura de branch", "Feature branch"]
tags:
  - concept
  - git
  - devops
  - branching
  - convencao
created: 2026-08-20
updated: 2026-08-20
---
> [!info] Conjunto: [[Fluxo Empresarial de Git|🏢 Fluxo Empresarial de Git]] · Relacionado: [[git-modelos-de-branching|🌳 Modelos de branching]] · [[git-commits-e-mensagens|✍️ Commits]]

# 🏷️ Convenção de branches

> [!abstract] Por que padronizar o nome
> Numa empresa há dezenas de branches abertas ao mesmo tempo. O nome é a **única** informação visível na lista: sem padrão, `teste2`, `arrumando` e `joao-final-final` convivem e ninguém sabe o que pode deletar. Com padrão, dá para filtrar, automatizar e rastrear até o ticket.

## 🧬 Anatomia

```
feat/PROJ-142-login-com-google
└┬─┘ └───┬───┘ └──────┬──────┘
 │       │            └── descrição curta em kebab-case
 │       └── identificador do ticket (Jira, Linear, issue)
 └── tipo da mudança
```

## 📋 Os tipos

| Prefixo | Para quê | Nasce de | Vira release |
|---|---|---|---|
| `feat/` | Funcionalidade nova | `main` | minor |
| `fix/` | Correção de bug (não urgente) | `main` | patch |
| `hotfix/` | Correção **urgente em produção** | tag de produção | patch imediato |
| `refactor/` | Muda estrutura, não comportamento | `main` | — |
| `chore/` | Dependência, config, build | `main` | — |
| `docs/` | Só documentação | `main` | — |
| `test/` | Só teste | `main` | — |
| `spike/` | Investigação, protótipo descartável | `main` | **nunca** |
| `release/` | Estabilização de versão | `main`/`develop` | major/minor |

> [!note] `spike/` é o prefixo que mais falta nas equipes
> Investigação técnica ("dá para usar essa lib?") gera código sujo por natureza. Marcar como `spike/` deixa explícito que **aquilo não vai ser mergeado** — o resultado do spike é a conclusão, não o código. Sem esse prefixo, protótipo vira produção por acidente, que é uma das origens mais comuns de dívida técnica.

## ✅ Regras de nome

| Regra | Motivo |
|---|---|
| Tudo minúsculo | Alguns sistemas de arquivos não diferenciam maiúscula — `Feat/X` e `feat/X` viram conflito |
| `kebab-case` na descrição | Legível na URL do PR e no terminal |
| Sem acento, sem `ç`, só ASCII | Ferramenta de CI, shell e URL quebram com acento |
| Máximo ~50 caracteres | Nome longo é truncado na interface |
| Sempre com o ticket, quando existe | É o elo entre código e a decisão de negócio |
| Nunca `/` no fim nem duplo `//` | Git rejeita ou cria referência estranha |

### Bom × ruim

| ❌ | ✅ |
|---|---|
| `nova-feature` | `feat/PROJ-88-exportar-csv` |
| `bugfix` | `fix/PROJ-91-timeout-no-upload` |
| `joao/teste` | `spike/avaliar-redis-para-cache` |
| `Correção-Login` | `fix/PROJ-77-login-expira-cedo` |
| `feature/refatorar-tudo-e-arrumar-o-login-e-o-cadastro` | duas branches separadas |

## ⏳ Tempo de vida — a regra mais ignorada

> [!warning] O conflito de merge cresce com o tempo, não com o tamanho
> Uma branch de 2 dias com 500 linhas costuma mergear limpo. Uma de 3 semanas com 200 linhas vira um pesadelo — porque a `main` andou 3 semanas embaixo dela. **O inimigo é a divergência acumulada**, não o volume de código.

| Duração | Situação |
|---|---|
| < 1 dia | Ideal |
| 1–3 dias | Saudável |
| 4–7 dias | Sinal de alerta — quebre em partes ou rebaseie na main todo dia |
| > 1 semana | **A feature foi mal fatiada.** Reveja o escopo, não o Git |

Se a feature é grande demais para caber em dias, as saídas são: fatiar em PRs que entregam valor parcial, ou entrar na main desligada por **feature flag** ([[git-modelos-de-branching|🌳 Trunk-Based]]).

## 🧹 Higiene do repositório

```bash
# atualiza a branch com o que entrou na main (diariamente)
git switch feat/PROJ-142-login-com-google
git fetch origin
git rebase origin/main

# depois do merge: deletar local e remoto
git switch main && git pull
git branch -d feat/PROJ-142-login-com-google
git push origin --delete feat/PROJ-142-login-com-google

# limpar referências de branches já deletadas no servidor
git fetch --prune

# listar branches locais já mergeadas (candidatas a deletar)
git branch --merged main
```

> [!tip] Ligue a exclusão automática de branch no merge
> GitHub, GitLab e Bitbucket têm a opção "delete branch on merge". Um clique elimina para sempre a pergunta "posso apagar essa?" — o histórico fica no PR e no commit, não na branch.

## 🔒 Fazer valer na prática

Convenção que depende de boa vontade dura duas semanas. Formas de garantir:

- **Regra de proteção no servidor** com padrão de nome permitido
- **Template de branch** no cartão do ticket (o Jira/Linear já sugere o nome pronto)
- **Hook local** rejeitando push de branch fora do padrão
- **Bloquear push direto na `main`** — ver [[git-protecao-e-permissoes|🔒 Proteção e permissões]]

## 🧩 Conceitos relacionados

- [[Fluxo Empresarial de Git|🏢 Fluxo Empresarial de Git]]
- [[git-commits-e-mensagens|✍️ Commits e mensagens]]
- [[git-pull-request-e-code-review|🔍 Pull Request e code review]]
- [[Snippets - Git|🧩 Snippets de Git]]
