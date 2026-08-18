---
title: "Catálogo — áreas, cursos e perguntas do quiz"
aliases: ["Catálogo do TCC", "Seeds do TCC", "Áreas e cursos"]
tags: [tcc, catalogo, seeds, dados, quiz]
status: concluido
projeto: TCC
criado: 2026-08-17
---

> [!info] Projeto: [[TCC|🎓 TCC]] · Ver também: [[modelagem-dados-quiz|🗃️ Modelagem]] · [[engine-matching-cosseno|🧮 Engine]]

# 📚 Catálogo de áreas, cursos e perguntas

> [!abstract] O que é
> A base de dados que alimenta a engine, versionada em três management commands idempotentes. Inspirada no catálogo de cursos técnicos do SENAI. Enquanto o scraping não existe, **o seed é a fonte da verdade**.

## 🧭 As 7 áreas (os eixos do espaço vetorial)

| Slug | Nome | Descrição |
|---|---|---|
| `eletrica` | Elétrica | Instalações elétricas, comandos e NR-10 |
| `costura` | Costura | Modelagem, corte e costura industrial |
| `ia` | Inteligência Artificial | Machine learning, dados e automação inteligente |
| `ti` | Tecnologia da Informação | Programação, redes e suporte técnico |
| `mecanica-automotiva` | Mecânica Automotiva | Motores, injeção eletrônica e freios |
| `mecanica` | Mecânica | Usinagem, metrologia e manutenção mecânica |
| `eletromecanica` | Eletromecânica | Integração de sistemas elétricos e mecânicos |

`catalog/management/commands/seed_areas.py`

## 🎓 Os 12 cursos e seus pesos

Pesos de 0 a 5. Célula vazia = 0 (a área nem aparece no banco).

| Curso | h | Área principal | Elé | Cost | IA | TI | Auto | Mec | Eletro |
|---|---|---|---|---|---|---|---|---|---|
| Eletricista Instalador | 160 | Elétrica | **5** | | | | | 1 | 3 |
| Comandos Elétricos e CLP | 120 | Elétrica | **5** | | | 1 | | 1 | 4 |
| Costura Industrial | 200 | Costura | | **5** | | | | 1 | |
| Modelagem e Corte de Vestuário | 160 | Costura | | **5** | | 1 | | | |
| Python para Análise de Dados | 80 | TI | | | 4 | **5** | | | |
| Fundamentos de Inteligência Artificial | 100 | IA | | | **5** | 4 | | | |
| Redes e Infraestrutura de TI | 160 | TI | 2 | | | **5** | | | 1 |
| Mecânica de Motores a Combustão | 200 | Mec. Automotiva | | | | | **5** | 3 | 1 |
| Injeção Eletrônica Automotiva | 120 | Mec. Automotiva | 3 | | | 1 | **5** | 2 | 2 |
| Usinagem CNC | 220 | Mecânica | 1 | | | 2 | 1 | **5** | 2 |
| Manutenção Eletromecânica Industrial | 240 | Eletromecânica | 4 | | | 1 | 1 | 4 | **5** |
| Automação Industrial | 180 | Eletromecânica | 4 | | 2 | 3 | | 2 | **5** |

`catalog/management/commands/seed_courses.py`

### 🔥 Os pares híbridos — o que dá o que provar

> [!important] Catálogo só com cursos "puros" não prova nada
> Se cada curso pesasse uma única área, a engine seria um `if` glorificado. Os pares abaixo foram desenhados de propósito para que **só o cálculo** consiga desempatar:

| Par | O que desempata |
|---|---|
| **Injeção Eletrônica** vs **Motores a Combustão** | Elétrica. Injeção só vence quando o perfil mistura automotiva **com** elétrica |
| **Automação Industrial** vs **Manutenção Eletromecânica** | TI/IA contra mecânica pura — os dois pesam eletromecânica 5 |
| **Python para Dados** vs **Fundamentos de IA** | Quase idênticos, invertidos (5/4 e 4/5) |

É nesses pares que o `explanation` justifica por que um passou o outro por centésimos.

## ❓ As 6 perguntas do quiz

`quiz/management/commands/seed_questions.py` — cada pergunta tem 4 alternativas, cada alternativa tem seus pesos.

| # | Pergunta | Eixo que ela mede |
|---|---|---|
| 1 | Qual tipo de tarefa te dá mais satisfação? | Natureza do trabalho (montar / criar / resolver / desmontar) |
| 2 | Em qual ambiente você se imagina trabalhando? | Contexto físico (oficina / escritório / fábrica / ateliê) |
| 3 | Você prefere trabalhar mais com o quê? | Objeto de trabalho (circuitos / tecidos / dados / motores) |
| 4 | Diante de um equipamento quebrado, o que faz primeiro? | Método de diagnóstico |
| 5 | O que mais te atrai em uma profissão? | Motivação |
| 6 | Com qual dessas ferramentas você se imagina trabalhando? | Afinidade instrumental |

> [!note] Por que 6 perguntas com 4 alternativas
> Cada alternativa pesa 2 ou 3 áreas ao mesmo tempo, nunca uma só de forma isolada — é o que faz o perfil final ser um **vetor rico** em vez de uma contagem de votos. Seis perguntas bastam para separar as 7 áreas e mantêm o quiz curto o suficiente para ninguém abandonar no meio.

## ⚙️ Padrão dos seeds

```python
Course.objects.update_or_create(name=..., defaults={...})   # chave natural
course.area_weights.exclude(area__slug__in=...).delete()    # limpa órfãos
```

- **Idempotente:** roda 10 vezes, não duplica.
- **Fonte da verdade:** tirou um peso do arquivo, ele some do banco.
- **`@transaction.atomic`:** ou popula tudo, ou nada.
- **`CommandError` se a área não existir:** o seed de cursos falha alto se `seed_areas` não rodou antes, em vez de criar dado inconsistente.

> [!tip] Os seeds também são as fixtures dos testes
> `setUpTestData` os invoca via `call_command`. Se um seed quebrar, a suíte inteira acusa — ver [[testes-e-validacao-tcc|✅ Testes]].

## Veja também

- [[TCC|🎓 TCC]]
- [[engine-matching-cosseno|🧮 Engine de matching]]
- [[guia-tcc-quiz-perfil|🏗️ Guia de implementação]]
