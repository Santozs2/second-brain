---
type: concept
area: Conceitos
status: estavel
aliases: ["Pull Request", "Code review", "PR", "Merge Request", "CODEOWNERS"]
tags:
  - concept
  - git
  - devops
  - processo
  - review
created: 2026-08-20
updated: 2026-08-20
---
> [!info] Conjunto: [[Fluxo Empresarial de Git|🏢 Fluxo Empresarial de Git]] · Relacionado: [[git-protecao-e-permissoes|🔒 Proteção]] · [[CI-CD|CI/CD]]

# 🔍 Pull Request e code review

> [!abstract] O que um PR realmente é
> Não é um pedido de permissão. É **uma unidade de mudança com contexto anexado**: o que muda, por quê, como testar e quem já olhou. O merge é a menor parte — o valor é o registro que fica.

## 📏 Tamanho: a variável que decide tudo

> [!warning] Acima de ~400 linhas, o review deixa de funcionar
> É o achado mais consistente da literatura de engenharia: a capacidade de encontrar defeitos cai fortemente conforme o PR cresce. Não porque a pessoa fique preguiçosa — é limite de atenção. Um PR de 1.500 linhas recebe "LGTM 👍" em três minutos, e todo mundo sabe disso.

| Linhas | O que costuma acontecer |
|---|---|
| < 100 | Review cuidadoso, comentários específicos |
| 100–400 | Zona saudável |
| 400–1000 | Review superficial, foco só no óbvio |
| > 1000 | Aprovação por cansaço |

**Quando o PR é inevitavelmente grande** (migração, geração automática, renomeação em massa): avise no título, separe os commits por etapa, e diga na descrição o que precisa de atenção e o que é mecânico.

## 🧾 Template de PR

```markdown
## O que muda
<uma frase>

## Por quê
<o problema ou o ticket; link para a decisão>

## Como testar
1. …
2. …

## Riscos e impacto
- [ ] Migração de banco
- [ ] Muda contrato de API
- [ ] Precisa de variável de ambiente nova
- [ ] Afeta desempenho

## Evidência
<print, log, saída de teste>
```

Salvo em `.github/pull_request_template.md`, ele aparece preenchido em todo PR novo.

> [!tip] O campo "Riscos" é o que mais economiza incidente
> Ele obriga o autor a parar e pensar antes de pedir review — e é onde o revisor descobre a migração de banco que passaria despercebida no meio de 30 arquivos.

## 👥 CODEOWNERS

```
# .github/CODEOWNERS
*                       @time-plataforma
/api/                   @time-backend
/web/                   @time-frontend
/infra/                 @time-devops @seguranca
*.sql                   @dba
/quiz/engine.py         @dono-do-motor
```

Quem mexeu no arquivo, o dono é chamado automaticamente para revisar. Combinado com proteção de branch, vira **aprovação obrigatória** de quem entende daquele pedaço.

## 🔎 O que revisar (em ordem de importância)

1. **Corretude** — resolve o problema? Trata os casos de borda?
2. **Contrato** — quebra alguém que consome isso? Precisa de migração?
3. **Segurança** — entrada validada? Segredo no código? Injeção? Permissão checada?
4. **Testes** — cobrem o caso que o bug expôs, ou só o caminho feliz?
5. **Legibilidade** — quem chegar depois entende sem perguntar?
6. **Desempenho** — consulta em laço? N+1?

### O que **não** revisar

Formatação, aspas, ponto e vírgula, ordem de import. Isso é trabalho de **linter e formatter automáticos**. Discussão de estilo em PR gasta o capital de atenção que deveria ir para os itens 1 a 3 — e azeda o clima do time por nada.

## 💬 Como comentar

Prefixar o comentário com a intenção elimina metade dos mal-entendidos:

| Prefixo | Significa |
|---|---|
| `nit:` | Detalhe, pode ignorar |
| `question:` | Não entendi, me explica |
| `suggestion:` | Ideia, você decide |
| `blocking:` | Precisa mudar antes do merge |
| `praise:` | Isso ficou bom |

> [!important] Revise o código, nunca a pessoa
> "Esse método faz três coisas, dá para separar?" e "você não sabe separar responsabilidade?" apontam para o mesmo problema, mas só um deles mantém a pessoa disposta a abrir o próximo PR. Code review é a atividade de maior risco social de um time técnico — o tom não é firula, é o que faz o processo sobreviver.

## ⏱️ Ritmo

- **SLA de 24h** para o primeiro retorno. PR parado é trabalho parado e branch envelhecendo ([[git-convencao-de-branches|🏷️]]).
- **Draft PR** desde o começo: mostra o que está sendo feito e evita duas pessoas na mesma coisa.
- **Self-review antes de pedir**: leia o próprio diff na interface. Metade dos comentários que você receberia aparecem sozinhos aí — `console.log` esquecido, arquivo que não devia estar ali.
- **Ninguém aprova o próprio PR.** Nem o líder, nem quem escreveu sozinho.

## 🤖 O robô revisa primeiro

Antes de gastar tempo humano, o CI deve garantir: lint, formatação, testes, build, cobertura e varredura de segredos. Ver [[CI-CD|CI/CD]] e [[git-protecao-e-permissoes|🔒 Proteção e permissões]].

> [!success] A ordem certa é robô → humano
> Toda checagem mecânica que um humano faz é desperdício de uma habilidade que só ele tem: julgar se a solução **é a certa para o problema**. Automatize tudo que for objetivo e reserve a revisão humana para o que exige contexto.

## 🧩 Conceitos relacionados

- [[Fluxo Empresarial de Git|🏢 Fluxo Empresarial de Git]]
- [[git-protecao-e-permissoes|🔒 Proteção e permissões]]
- [[git-merge-rebase-e-historico|🔀 Merge, rebase e histórico]]
- [[CI-CD|CI/CD]]
