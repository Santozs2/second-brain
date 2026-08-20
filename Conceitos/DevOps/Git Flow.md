---
type: concept
area: Conceitos
status: estavel
aliases: ["Gitflow", "Git Flow do Driessen"]
tags:
  - concept
  - git
  - devops
created: 2026-06-30
updated: 2026-08-20
---
# Git Flow

## Definição

Modelo de organização de branches no Git: `main` (produção), `develop` (integração), `feature/*` (novas funcionalidades), `release/*` e `hotfix/*`.

> [!warning] "Git Flow" é **um** modelo, não sinônimo de "fluxo de Git"
> É o modelo específico publicado por Vincent Driessen em 2010 — e o mais copiado por reflexo. Existem outros (GitHub Flow, GitLab Flow, Trunk-Based), e para aplicação web com deploy contínuo eles costumam ser melhores. A comparação está em [[git-modelos-de-branching|🌳 Modelos de branching]].

## Quando usar

Em projetos com **versões que o cliente instala** e mais de uma versão em suporte ao mesmo tempo, ou com ciclos de release bem definidos e homologação formal.

Para aplicação web com deploy contínuo, `develop` vira uma branch a mais para manter sincronizada sem ganho — nesses casos, [[git-modelos-de-branching|GitHub Flow]] entrega o mesmo com menos peça. Em projeto pessoal pequeno, `main` + `feature/*` basta.

## Conceitos relacionados

- [[Fluxo Empresarial de Git|🏢 Fluxo Empresarial de Git]] — o conjunto completo: branches, PR, review, release e hotfix
- [[git-modelos-de-branching|🌳 Modelos de branching]] — Git Flow comparado com as alternativas
- [[CI-CD|CI/CD]]

## Veja também

- [[Git|Git]]
- [[Documentações|Documentações]]
