---
type: concept
area: Conceitos
status: estavel
aliases: ["Branch protection", "Proteção de branch", "Permissões no Git", "Segredo vazado"]
tags:
  - concept
  - git
  - devops
  - seguranca
  - governanca
created: 2026-08-20
updated: 2026-08-20
---
> [!info] Conjunto: [[Fluxo Empresarial de Git|🏢 Fluxo Empresarial de Git]] · Relacionado: [[git-pull-request-e-code-review|🔍 Code review]] · [[CI-CD|CI/CD]]

# 🔒 Proteção e permissões

> [!abstract] O princípio
> Regra que depende de as pessoas lembrarem não é regra — é intenção. **Proteção de branch transforma acordo de equipe em restrição técnica**, que funciona igual às 3 da manhã, no dia da entrega, com o time cansado.

## 🛡️ Regras de proteção da `main`

| Regra | O que impede |
|---|---|
| **Exigir Pull Request** | Commit direto na `main` |
| **Exigir N aprovações** | Merge sem ninguém olhar |
| **Descartar aprovações ao novo push** | Aprovar, depois trocar o código escondido |
| **Exigir checks de CI verdes** | Merge com teste quebrado |
| **Exigir branch atualizada com a main** | Merge que passa isolado e quebra integrado |
| **Exigir resolução das conversas** | Comentário de review ignorado |
| **Exigir aprovação de CODEOWNERS** | Mexer na área de outro time sem avisar |
| **Bloquear force push** | Reescrever histórico público |
| **Bloquear exclusão da branch** | Apagar a `main` por acidente |
| **Exigir histórico linear** | Impõe squash/rebase, se for a política |
| **Exigir commits assinados** | Commit forjado com o e-mail de outra pessoa |

> [!warning] "Incluir administradores" é a regra que separa política de teatro
> Na maioria das ferramentas, admins escapam das proteções por padrão. Se o líder técnico pode dar merge sem review "porque é urgente", a regra vale só para quem tem menos poder — e a exceção vira hábito. Ou a proteção vale para todos, ou ela não vale.

## 👤 Níveis de acesso

| Papel | Pode |
|---|---|
| **Read** | Clonar, abrir issue |
| **Triage** | Gerenciar issues/PRs, sem escrever código |
| **Write** | Criar branch, abrir PR, mergear onde não há proteção |
| **Maintain** | Configurar repositório, sem tocar em segurança |
| **Admin** | Tudo, inclusive mudar as proteções |

**Princípio do menor privilégio:** o padrão é Write; Admin fica com duas pessoas, nunca uma (fator ônibus) e nunca o time inteiro.

## ✍️ Commits assinados

Git não verifica identidade: qualquer um configura `user.email` com o endereço de outra pessoa e commita no nome dela. Assinatura (GPG ou SSH) resolve, e o selo *Verified* aparece na plataforma.

```bash
git config --global commit.gpgsign true
git config --global gpg.format ssh
git config --global user.signingkey ~/.ssh/id_ed25519.pub
```

Em ambiente regulado (financeiro, saúde), costuma ser obrigatório por auditoria.

## 🔑 Segredos — a regra sem exceção

> [!danger] Segredo commitado é segredo vazado — mesmo que você apague depois
> `git rm` só remove do estado atual: o valor continua no histórico, em qualquer clone que alguém já tenha, e possivelmente em cache da plataforma e em bots que varrem repositórios em segundos. **A única resposta correta é rotacionar a credencial.** Limpar o histórico é a segunda tarefa, nunca a primeira.

**Se vazou, nesta ordem:**
1. **Revogar e gerar nova credencial** — imediatamente
2. Verificar logs de uso indevido
3. Limpar o histórico (`git filter-repo`, ou o suporte da plataforma) — coordenado com o time, porque reescreve tudo
4. Ligar varredura automática de segredos para não repetir

**Prevenção:**
- `.env` no `.gitignore`, com um `.env.example` versionado sem valores
- Secret scanning e push protection ligados no repositório
- Segredo de CI em cofre de variáveis, nunca no YAML
- Hook local (`gitleaks`, `detect-secrets`) antes do commit

## 🏗️ Ambientes e aprovação de deploy

Separado da proteção de branch, há a proteção de **ambiente**: quem pode deployar em produção, com aprovação manual, janela de horário e restrição de qual branch pode chegar lá.

```
main (protegida) ──► staging (deploy automático)
                  └► production (exige aprovação humana)
```

> [!tip] Merge e deploy são decisões diferentes
> Confundir as duas é o que faz times terem medo de mergear. Com ambientes separados, entrar na `main` é barato e reversível; ir para produção é a decisão que merece o gate.

## 📋 Configuração mínima para qualquer repositório sério

- [ ] `main` protegida, sem push direto
- [ ] PR com pelo menos 1 aprovação
- [ ] CI obrigatório (teste + lint) para mergear
- [ ] Force push e exclusão bloqueados
- [ ] Aprovações descartadas a cada novo push
- [ ] `.gitignore` cobrindo `.env`, credenciais e build
- [ ] Secret scanning ligado
- [ ] Regras valendo **também para administradores**

## 🧩 Conceitos relacionados

- [[Fluxo Empresarial de Git|🏢 Fluxo Empresarial de Git]]
- [[git-pull-request-e-code-review|🔍 Pull Request e code review]]
- [[git-release-e-versionamento|🚀 Release e versionamento]]
- [[CI-CD|CI/CD]] · [[cs-authentication|Autenticação]]
