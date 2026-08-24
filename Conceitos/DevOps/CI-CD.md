---
type: concept
area: Conceitos
status: estavel
aliases: 
tags:
  - concept
  - devops
created: 2026-06-30
updated: 2026-08-24
---
# CI/CD

## Definição

Integração Contínua (CI) e Entrega/Implantação Contínua (CD): automatizam testes, build e deploy a cada mudança no código, em vez de fazer tudo manualmente.

| Sigla | Nome | O que garante |
|---|---|---|
| **CI** | Continuous Integration | Todo commit é integrado e verificado automaticamente |
| **CD** | Continuous **Delivery** | Todo commit aprovado fica **pronto** para deploy (com aprovação manual) |
| **CD** | Continuous **Deployment** | Todo commit aprovado **vai** para produção, sem intervenção |

> [!note] Delivery e Deployment são coisas diferentes com a mesma sigla
> *Delivery* deixa o artefato pronto e espera alguém apertar o botão. *Deployment* aperta sozinho. A maior parte dos times faz **Delivery** — e para projeto acadêmico ou pessoal, é a escolha correta.

## Quando usar

Em qualquer projeto com mais de um colaborador, ou quando o deploy manual já virou repetitivo o suficiente para valer a pena automatizar.

> [!tip] O valor do CI aparece antes do segundo colaborador
> Mesmo em projeto solo, o CI responde a uma pergunta que a máquina local não responde: *"isso funciona fora do meu computador?"*. Ele pega dependência esquecida no `requirements.txt`, variável de ambiente que só existe na sua máquina e teste que passa por acidente.

## O pipeline típico

```
push / pull request
      ↓
┌─────────────────────────────┐
│ 1. Checkout do código       │
│ 2. Setup do ambiente        │
│ 3. Cache de dependências    │
│ 4. Instalar dependências    │
│ 5. Lint / formatação        │
│ 6. Testes + cobertura       │
│ 7. Build                    │
│ 8. Deploy (se main)         │
└─────────────────────────────┘
```

## GitHub Actions — exemplo funcional

```yaml
name: CI

on:
  push: { branches: [main] }
  pull_request:

jobs:
  test:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:16
        env: { POSTGRES_PASSWORD: postgres }
        options: >-
          --health-cmd pg_isready --health-interval 10s --health-retries 5
        ports: ["5432:5432"]

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-python@v5
        with:
          python-version: "3.12"
          cache: pip

      - run: pip install -r requirements.txt

      - name: Lint
        run: ruff check .

      - name: Testes
        env:
          DATABASE_URL: postgres://postgres:postgres@localhost:5432/postgres
          SECRET_KEY: chave-de-teste
        run: |
          coverage run --branch manage.py test
          coverage report --fail-under=70
```

## Boas práticas

- **Rápido** — pipeline acima de 10 minutos deixa de ser usado; paralelize e cacheie
- **Confiável** — teste instável (*flaky*) treina o time a ignorar o vermelho
- **Falha cedo** — lint antes de teste; teste rápido antes de teste lento
- **Um artefato, vários ambientes** — build uma vez, promova o mesmo artefato
- **Segredo em cofre** — `secrets` do GitHub, nunca no YAML
- **Branch protegida** — merge só com CI verde → [[Git Flow|Git Flow]]

> [!warning] Segredo em log de CI é vazamento permanente
> Logs de execução costumam ser públicos em repositório público. Um `echo $API_KEY` de depuração, ou uma exceção que imprime a configuração, expõe o segredo para sempre — e o histórico de execuções não é apagável com facilidade. Ver [[cs-authentication|🔑 Autenticação]].

## Estratégias de deploy

| Estratégia | Como funciona | Risco |
|---|---|---|
| **Recreate** | Derruba o antigo, sobe o novo | Downtime |
| **Rolling** | Substitui instâncias aos poucos | Duas versões convivem |
| **Blue-Green** | Dois ambientes; troca o tráfego | Custo dobrado |
| **Canary** | Fração do tráfego na versão nova | Complexidade |

> [!important] Migração de banco é o ponto onde o rollback deixa de ser trivial
> Reverter o código é fácil; reverter uma migration que apagou coluna, não. A prática segura é o **deploy em duas fases**: primeiro uma migração compatível com as duas versões do código (adiciona coluna, não remove), depois o código, e só em um deploy posterior a limpeza. Ver [[Migrations|Migrations]].

## Métricas DORA

As quatro métricas de *Accelerate* (Forsgren, Humble & Kim, 2018), que correlacionam com desempenho de entrega:

| Métrica | O que mede |
|---|---|
| **Frequência de deploy** | Com que frequência se entrega |
| **Lead time** | Do commit à produção |
| **Taxa de falha de mudança** | % de deploys que causam incidente |
| **Tempo de restauração** | Quanto leva para se recuperar |

## Conceitos relacionados

- [[Git Flow|Git Flow]] · [[Fluxo Empresarial de Git|🏢 Fluxo Empresarial de Git]]
- [[tst-piramide-de-testes|🔺 Pirâmide de testes]] · [[tst-cobertura|📊 Cobertura]]
- [[Docker|Docker]] · [[Git|Git]] · [[Migrations|Migrations]]

## Veja também

- [[Documentações|Documentações]]
