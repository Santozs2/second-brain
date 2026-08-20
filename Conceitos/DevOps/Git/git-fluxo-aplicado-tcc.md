---
type: concept
area: Conceitos
status: em-andamento
aliases: ["Git no TCC", "Fluxo de Git do TCC", "Git EducMatch"]
tags:
  - concept
  - git
  - devops
  - tcc
  - processo
created: 2026-08-20
updated: 2026-08-20
---
> [!info] Conjunto: [[Fluxo Empresarial de Git|🏢 Fluxo Empresarial de Git]] · Projeto: [[TCC|🎓 TCC]] · Equipe: [[divisao-de-trabalho-tcc|👥 Divisão de trabalho]]

# 🎓 Fluxo de Git aplicado ao TCC

> [!abstract] O mesmo processo, no tamanho certo
> Quatro pessoas, um repositório Django, ~3 meses. As notas deste conjunto descrevem práticas de empresas com dezenas de desenvolvedores; aqui está o recorte que faz sentido para o grupo — **o que adotar, o que simplificar e o que ignorar**.

## ✅ Modelo escolhido: GitHub Flow

```
main  ──●──●──●──●──●──►   sempre funcionando, sempre demonstrável
         \    \    \
          ●    ●    ●      uma branch por tarefa, curta
```

**Uma branch longa (`main`) e branches curtas por tarefa.** Nada de `develop`, `release/*` ou `hotfix/*`.

> [!warning] Git Flow completo seria um erro aqui
> É a tentação natural — o modelo mais citado, com cara de "profissional". Mas `develop` só existe para separar "integrado" de "publicado", e o TCC **não publica versões**: ele demonstra a `main` na banca. Com 4 pessoas, cada branch longa a mais é mais merge, mais conflito e mais tempo em Git em vez de no TCC. Ver [[git-modelos-de-branching|🌳 Modelos de branching]].

## 🏷️ Branches por frente

Cada frente da [[divisao-de-trabalho-tcc|divisão de trabalho]] usa o próprio escopo no nome — dá para ver de quem é a branch pelo `git branch -a`:

| Frente | Padrão de branch | Exemplo |
|---|---|---|
| 1 · 🧮 Motor | `feat/motor-*` · `fix/motor-*` | `feat/motor-indicador-de-confianca` |
| 2 · 🤖 Camada de IA | `feat/ia-*` | `feat/ia-fake-provider` |
| 3 · 📚 Catálogo e trilha | `feat/catalogo-*` | `feat/catalogo-seed-18-cursos` |
| 4 · 🎨 Jornada e indicadores | `feat/front-*` · `feat/dash-*` | `feat/front-tela-de-entrega` |

Sem ticket (o grupo não usa Jira) — o escopo no nome cumpre o papel de rastreabilidade. Regras de nome em [[git-convencao-de-branches|🏷️ Convenção de branches]]: minúsculas, kebab-case, sem acento.

## 💬 Commits

[[git-commits-e-mensagens|Conventional Commits]], com o escopo batendo com a frente:

```
feat(motor): adiciona indicador de confiança para perfis indecisos
fix(ia): trata JSON com cerca de código na resposta do Gemini
test(motor): cobre empate técnico entre top-1 e top-2
docs(tcc): escreve a seção de metodologia do cálculo
chore: atualiza requirements com google-genai
```

> [!tip] O histórico do TCC vira material da monografia
> `git log --oneline` filtrado por escopo é a linha do tempo real do desenvolvimento — quem fez o quê, quando, em que ordem. Serve para o cronograma do trabalho, para provar autoria individual e para responder na banca "como vocês dividiram?". Vale mais do que qualquer print de reunião.

## 🔍 Quem revisa quem

Com 4 pessoas, revisar em anel evita que todo mundo espere pela mesma pessoa:

```
F1 (motor) ──► revisado por F2
F2 (IA)    ──► revisado por F1
F3 (base)  ──► revisado por F4
F4 (front) ──► revisado por F3
```

Exceção: mudança que toca **contrato entre frentes** ([[divisao-de-trabalho-tcc|as quatro fronteiras]]) precisa de aprovação das duas pontas.

- **1 aprovação basta.** Duas travam demais um grupo desse tamanho.
- **SLA de 24h** para o primeiro retorno.
- Ninguém aprova o próprio PR — nem quando é "só um ajuste".

## 📝 Template de PR (versão curta)

```markdown
## O que muda
## Por quê
## Como testar
## Toca contrato de outra frente? ( ) não ( ) sim → qual
```

A última linha é a adaptação mais importante ao contexto do grupo: é o campo que impede alguém mudar `build_payload` e a outra frente descobrir no merge.

## 🛡️ Proteção mínima da `main`

Três configurações, cinco minutos, e a `main` para de quebrar:

- [ ] Exigir Pull Request (sem push direto)
- [ ] Exigir 1 aprovação
- [ ] Exigir o check de testes verde
- [ ] Deletar branch automaticamente no merge

O resto de [[git-protecao-e-permissoes|🔒 Proteção e permissões]] (commits assinados, CODEOWNERS, ambientes) é exagero para um TCC. **Mas a regra do segredo vale integralmente**: a chave da API do Gemini nunca entra no repositório — `.env` no `.gitignore`, `.env.example` versionado sem valores.

## 🤖 CI mínima

Um workflow que roda `manage.py test` a cada PR já entrega o essencial:

```yaml
# .github/workflows/test.yml
name: testes
on: [pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.12"
      - run: pip install -r requirements.txt
      - run: python manage.py test
        env:
          PYTHONIOENCODING: utf-8
          LLM_ENABLED: "false"
```

> [!important] `LLM_ENABLED=false` no CI não é detalhe
> Sem isso, cada PR chamaria o Gemini de verdade: gasta free tier, deixa o teste lento e o torna dependente de rede — um PR reprovaria por instabilidade da API, não por bug. É exatamente para isso que o `FakeProvider` existe ([[camada-ia-plano-implementacao|🧩 Plano da camada de IA]]).

## 🏷️ Tags por entrega

Marcar cada passo concluído dá pontos de retorno seguros e uma linha do tempo defensável:

```bash
git tag -a passo-7-fake-provider -m "Camada de IA offline concluída"
git push origin passo-7-fake-provider
```

E uma tag para a apresentação:

```bash
git tag -a defesa-v1 -m "Versão apresentada na banca"
```

> [!success] A tag da defesa é seguro de vida
> Se alguém quebrar alguma coisa na véspera, `git checkout defesa-v1` devolve o estado exato que funcionava. Crie essa tag **no dia em que o sistema estiver rodando redondo**, não na véspera.

## 📋 Checklist de setup (fazer na primeira reunião)

- [ ] Todos com acesso de escrita ao repositório
- [ ] `main` protegida com as 4 regras acima
- [ ] `.gitignore` cobrindo `.env`, `db.sqlite3`, `__pycache__`, `.venv`
- [ ] `.env.example` versionado
- [ ] Template de PR em `.github/pull_request_template.md`
- [ ] Workflow de testes rodando
- [ ] Padrão de branch e de commit combinado com os 4 (esta nota)

## 🧩 Conceitos relacionados

- [[Fluxo Empresarial de Git|🏢 Fluxo Empresarial de Git]]
- [[divisao-de-trabalho-tcc|👥 Divisão de trabalho do TCC]] · [[TCC|🎓 TCC]]
- [[git-modelos-de-branching|🌳 Modelos de branching]] · [[git-protecao-e-permissoes|🔒 Proteção]]
- [[CI-CD|CI/CD]] · [[ci-github-actions|CI com GitHub Actions]]
