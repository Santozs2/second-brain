---
type: concept
area: Conceitos
status: estavel
aliases: ["Modelos de branching", "GitHub Flow", "Trunk-Based Development", "GitLab Flow"]
tags:
  - concept
  - git
  - devops
  - branching
  - arquitetura
created: 2026-08-20
updated: 2026-08-20
---
> [!info] Conjunto: [[Fluxo Empresarial de Git|🏢 Fluxo Empresarial de Git]] · Relacionado: [[git-convencao-de-branches|🏷️ Convenção de branches]] · [[CI-CD|CI/CD]]

# 🌳 Modelos de branching

> [!abstract] A pergunta que este documento responde
> Quantas branches de longa duração o time precisa? A resposta define tudo o mais: quantos merges, quanto conflito, quanto tempo entre "pronto" e "no ar".

## 1️⃣ Git Flow (Driessen, 2010)

```
main     ──●────────────────●──────────►  (produção, só release e hotfix)
            \              /
release      \        ●──●
              \      /
develop  ──●───●────●───────●──────────►  (integração)
            \ /          \ /
feature      ●            ●
```

Cinco tipos de branch: `main`, `develop`, `feature/*`, `release/*`, `hotfix/*`.

| ✅ Bom quando | ❌ Ruim quando |
|---|---|
| Software com **versões instaladas** no cliente | Deploy contínuo (várias vezes por dia) |
| Múltiplas versões em suporte simultâneo | Time pequeno |
| Release com data marcada e homologação | O custo de manter `develop` sincronizada supera o ganho |

> [!warning] O modelo mais citado e o mais mal aplicado
> Git Flow virou sinônimo de "fluxo de Git" e é copiado por reflexo — inclusive pelo próprio autor, que anos depois publicou uma nota recomendando **não** usá-lo em aplicações web com entrega contínua. `develop` só faz sentido se existir um momento de "fechar a versão". Se você faz deploy da main direto, `develop` é uma branch a mais para manter sincronizada, sem nenhum ganho.

## 2️⃣ GitHub Flow

```
main  ──●──●──●──●──●──●──►  (sempre deployável)
         \    \    \
feature   ●    ●    ●        (curtas, uma por mudança)
```

Uma branch longa (`main`) e branches de feature curtas. Merge na main → deploy.

| ✅ Bom quando | ❌ Ruim quando |
|---|---|
| Aplicação web / SaaS | Existe versão a dar suporte em paralelo |
| Deploy automatizado e confiável | Não há CI — sem testes, a main quebra rápido |
| **É o padrão da maioria das equipes hoje** | Precisa de janela de homologação longa |

## 3️⃣ GitLab Flow

GitHub Flow **mais branches de ambiente**: `main` → `staging` → `production`. O código sobe promovendo de uma para a outra.

Serve quando existem ambientes de homologação com aprovação formal, mas não se quer a complexidade do `develop` + `release/*`.

## 4️⃣ Trunk-Based Development

```
main  ──●─●─●─●─●─●─●─●─●──►  commits pequenos e frequentes
          \ /   \ /
           ●     ●            branches de horas, não dias
```

Todo mundo integra na trunk pelo menos uma vez por dia. Código incompleto entra **desligado por feature flag** em vez de ficar esperando numa branch.

| ✅ Bom quando | ❌ Ruim quando |
|---|---|
| Cobertura de testes alta e CI rápida | Testes fracos — vira caos |
| Time maduro, deploy diário ou mais | Ninguém domina feature flags |
| Quer eliminar conflito de merge | Regulação exige aprovação antes de integrar |

> [!tip] Feature flag é a alternativa moderna à branch longa
> A branch de duas semanas existe porque a feature não está pronta para o usuário. A flag resolve o mesmo problema sem o custo: o código entra na main, **desligado**, e é integrado continuamente. Você troca "conflito de merge gigante no fim" por "um `if` temporário no código" — e paga o preço de precisar limpar as flags mortas depois.

## 📊 Comparação direta

| Critério | Git Flow | GitHub Flow | GitLab Flow | Trunk-Based |
|---|:---:|:---:|:---:|:---:|
| Branches de longa duração | 2+ | 1 | 2–3 | 1 |
| Complexidade | Alta | Baixa | Média | Baixa |
| Frequência de deploy | Baixa | Alta | Média | Muito alta |
| Risco de conflito | Alto | Baixo | Médio | Mínimo |
| Exige CI forte | Médio | Sim | Sim | **Crítico** |
| Suporta várias versões | ✅ | ❌ | ⚠️ | ❌ |

## 🧭 Como escolher (fluxograma honesto)

```
Você entrega software que o cliente instala e mantém versões antigas?
├── SIM  → Git Flow (ou release branches)
└── NÃO  → É web/SaaS?
           ├── Tem CI com testes confiáveis?
           │    ├── SIM → GitHub Flow (padrão) → Trunk-Based quando o time amadurecer
           │    └── NÃO → GitHub Flow + investir em teste ANTES de qualquer outra coisa
           └── Precisa de homologação formal com aprovação? → GitLab Flow
```

> [!success] Na dúvida, GitHub Flow
> É o mais simples que funciona, é o que a maioria dos desenvolvedores já conhece (onboarding de graça) e evolui naturalmente para Trunk-Based. Adicionar `develop` depois é fácil; remover uma branch que o time inteiro já incorporou ao hábito é difícil.

## 🧩 Conceitos relacionados

- [[Fluxo Empresarial de Git|🏢 Fluxo Empresarial de Git]]
- [[git-convencao-de-branches|🏷️ Convenção de branches]]
- [[git-release-e-versionamento|🚀 Release e versionamento]]
- [[Git Flow|Git Flow]] · [[CI-CD|CI/CD]]
