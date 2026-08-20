---
type: concept
area: Conceitos
status: estavel
aliases: ["Conventional Commits", "Mensagem de commit", "Commit atômico", "Padrão de commit"]
tags:
  - concept
  - git
  - devops
  - convencao
  - commits
created: 2026-08-20
updated: 2026-08-20
---
> [!info] Conjunto: [[Fluxo Empresarial de Git|🏢 Fluxo Empresarial de Git]] · Relacionado: [[git-release-e-versionamento|🚀 Release]] · [[git-merge-rebase-e-historico|🔀 Histórico]]

# ✍️ Commits e mensagens

> [!abstract] Para quem se escreve um commit
> Não é para você hoje — é para quem vai rodar `git blame` naquela linha daqui a dois anos tentando entender por que ela existe. Essa pessoa tem só a sua mensagem. Se ela diz "ajustes", a informação se perdeu para sempre.

## 📐 Conventional Commits — o padrão de mercado

```
<tipo>(<escopo>)!: <descrição no imperativo>
<linha em branco>
<corpo: o PORQUÊ da mudança>
<linha em branco>
<rodapé: refs e breaking changes>
```

Exemplo completo:

```
fix(auth): impede sessão de expirar antes do tempo configurado

O TTL era lido de SESSION_TTL, mas o middleware sobrescrevia com o
default de 30 min do framework. Usuários eram deslogados durante o
preenchimento de formulários longos.

A leitura passou a acontecer depois da inicialização do middleware.

Refs: PROJ-91
```

### Os tipos e o impacto na versão

| Tipo | Significa | [[git-release-e-versionamento\|SemVer]] |
|---|---|:---:|
| `feat` | Funcionalidade nova | **minor** (1.2.0 → 1.3.0) |
| `fix` | Correção de bug | **patch** (1.2.0 → 1.2.1) |
| `refactor` | Muda estrutura, não comportamento | — |
| `perf` | Melhora desempenho | patch |
| `test` | Adiciona ou corrige teste | — |
| `docs` | Só documentação | — |
| `build` | Build, dependências | — |
| `ci` | Pipeline de CI | — |
| `chore` | Tarefa sem impacto no código de produção | — |
| `revert` | Desfaz um commit anterior | depende |

O `!` antes dos dois-pontos (ou o rodapé `BREAKING CHANGE:`) sinaliza quebra de compatibilidade → **major** (1.2.0 → 2.0.0).

> [!success] O ganho concreto do padrão
> Com Conventional Commits, o **changelog e o número da versão são gerados sozinhos**. Sem ele, alguém escreve as notas de release à mão toda vez — e esquece metade. É a rara convenção que se paga em automação, não só em disciplina.

## ✂️ Commit atômico

Um commit = uma mudança lógica completa e coerente. O teste: **dá para descrever sem usar "e"?**

| ❌ | ✅ |
|---|---|
| `feat: adiciona login e corrige o header e atualiza libs` | três commits |
| `fix: bug` | `fix(upload): trata arquivo vazio sem estourar exceção` |
| `wip` | não commite wip em branch que vai virar PR |

> [!tip] `git add -p` é o comando que torna isso possível
> Ele deixa você escolher **pedaços** de um arquivo para o commit, em vez de tudo. É o que permite separar "arrumei o bug" de "aproveitei e renomeei três variáveis" quando as duas coisas foram feitas juntas.

Por que importa: `git revert` desfaz **um commit inteiro**. Se o commit misturou o bug fix com uma refatoração, desfazer o bug fix leva a refatoração junto. `git bisect` também depende disso — ele acha o commit que quebrou, e um commit gigante não diz nada.

## 📝 Regras de escrita

| Regra | Exemplo |
|---|---|
| Imperativo, como uma ordem | `adiciona`, não `adicionado` nem `adicionando` |
| Minúscula na primeira letra | `fix(api): corrige...` |
| Sem ponto final no assunto | `...no upload` |
| Assunto até ~50 caracteres | O Git trunca visualmente depois disso |
| Corpo quebrado em ~72 colunas | Terminal não quebra linha sozinho |
| O corpo explica **por quê**, nunca **o quê** | O diff já mostra o quê |

> [!warning] "O quê" no corpo é desperdício
> `Alterei a função X para receber o parâmetro Y` — isso está no diff, em cores, com precisão. O que o diff **não** consegue mostrar é: qual bug isso resolve, qual alternativa foi descartada, qual restrição obrigou essa escolha. É isso que o corpo tem de guardar.

## 🏷️ Rodapés úteis

```
Refs: PROJ-142                          → liga ao ticket
Closes: #87                             → fecha a issue no merge
Co-authored-by: Nome <email@x.com>      → crédito em pair programming
BREAKING CHANGE: /v1/users foi removido → força bump de major
Reviewed-by: Nome <email@x.com>         → rastreabilidade de auditoria
```

## 🤖 Automatizar a conferência

```bash
# commitlint + husky: rejeita commit fora do padrão antes de criar
npm i -D @commitlint/cli @commitlint/config-conventional husky
```

- **commitlint** valida a mensagem no `commit-msg` hook
- **husky** instala os hooks no clone de todo mundo
- **commitizen** oferece um assistente interativo para quem está aprendendo
- No CI, validar também as mensagens do PR — hook local pode ser burlado com `--no-verify`

> [!note] Squash muda quem escreve a mensagem final
> Se o time usa **squash merge** ([[git-merge-rebase-e-historico|🔀]]), os commits da branch somem e o que fica na `main` é o **título do Pull Request**. Nesse caso a convenção precisa valer para o título do PR — validar só os commits locais dá uma falsa sensação de padrão.

## 🧩 Conceitos relacionados

- [[Fluxo Empresarial de Git|🏢 Fluxo Empresarial de Git]]
- [[git-convencao-de-branches|🏷️ Convenção de branches]]
- [[git-release-e-versionamento|🚀 Release e versionamento]]
- [[git-merge-rebase-e-historico|🔀 Merge, rebase e histórico]]
