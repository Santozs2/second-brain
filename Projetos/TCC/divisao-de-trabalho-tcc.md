---
title: "Divisão de trabalho — 4 frentes do TCC (EducMatch)"
aliases: ["Divisão do TCC", "Frentes do TCC", "Quem faz o quê", "Divisão de trabalho"]
tags: [tcc, gestao, equipe, planejamento, escopo]
status: em-andamento
projeto: TCC
criado: 2026-08-20
---

> [!info] Projeto: [[TCC|🎓 TCC]] · Escopo: [[escopo-fluxo-educmatch|🗺️ Fluxo EducMatch]] · Plano da IA: [[camada-ia-plano-implementacao|🧩 Camada de IA]]

# 👥 Divisão de trabalho — 4 frentes

> [!abstract] Como esta divisão foi feita
> Não por arquivo nem por "quem gosta de front", mas por **fronteira de contrato**. Cada frente publica um formato de dados fixo e consome o da anterior. Enquanto o contrato não muda, as quatro pessoas trabalham em paralelo sem se esbarrar — e ninguém fica esperando ninguém para começar.

| Frente | Dono | Uma frase |
|---|---|---|
| 1️⃣ 🧮 **Motor e calibração** | **você** (scrum master) | Do perfil ao ranking numérico |
| 2️⃣ 🤖 **Camada de IA e experimento** | **você** (scrum master) | Do ranking ao texto entregue |
| 3️⃣ 📚 **Catálogo, trilha e integração** | *a definir* | A matéria-prima que o motor consome |
| 4️⃣ 🎨 **Jornada, avaliação e indicadores** | *a definir* | O que o usuário vê e o que a instituição mede |

> [!todo] Preencher os dois nomes que faltam
> A divisão abaixo funciona com qualquer arranjo, mas **cada frente precisa de um dono único**. Frente com dois donos vira frente sem dono.

> [!success] Atualização de 2026-08-23 — F1 e F2 passam a ter o mesmo dono
> O scrum master assume **as duas primeiras frentes** e acompanha as outras duas. É a única fusão que quase não custa coordenação: F1 e F2 vivem do mesmo contrato (`build_payload`), então a fronteira entre elas vira código em vez de reunião. O detalhamento — contratos congelados, backlog tarefa a tarefa, migração de dados e cronograma de 5 semanas — está em [[spec-motor-e-ia-frentes-1-2|🧭 Spec das Frentes 1 e 2]].
> **O que isso não resolve:** a F3 continua sem dono e continua sendo o caminho crítico do TCC inteiro.

---

## 🔗 As quatro fronteiras (é aqui que a divisão se sustenta)

```
[F3] catálogo  ──►  [F1] motor  ──►  [F2] camada de IA  ──►  [F4] jornada e indicadores
      cursos,          top-5 +          rank_final +             telas, avaliações,
      pesos,           explanation      texto entregue           dashboards
      trilha
```

| Fronteira | Quem publica | Quem consome | O que fica congelado |
|---|---|---|---|
| Catálogo → Motor | F3 | F1 | Tabelas `Area`, `Course`, `CourseAreaWeight` — o motor lê, nunca escreve |
| Motor → IA | F1 | F2 | `build_payload(attempt)` — o JSON de entrada do prompt |
| IA → Jornada | F2 | F4 | `rank_final`, `llm_text`, `is_primary`, `used_fallback` |
| Jornada → Indicadores | F4 | F4 | `QuizAttempt` e seus metadados |

> [!important] Regra de ouro: contrato antes de código
> Os dois contratos centrais já estão escritos em [[camada-ia-plano-implementacao|🧩 Plano da camada de IA]]. **Fechem os outros dois na primeira reunião**, antes de qualquer um abrir o editor. Uma tarde definindo formato de dados economiza duas semanas de retrabalho — e é a única coisa que impede quatro pessoas de virarem quatro projetos.

---

## 1️⃣ 🧮 Motor e calibração — *você*

**Fronteira:** recebe respostas, devolve ranking numérico com justificativa. Não sabe de onde vêm os cursos nem para onde vai o resultado.

### Escopo
- `quiz/engine.py` inteiro: `course_vector`, `profile_vector`, `dot`, `norm`, `cosine_similarity`, `explain`, `rank_courses`, `recommend`
- **Recalibração com os 18 cursos reais** — os pesos atuais foram feitos para 12 cursos fictícios
- **Análise de sensibilidade**: o que acontece com o ranking quando um peso muda de 4 para 5
- **Indicador de confiança**: sinalizar perfil indeciso (top-1 e top-2 quase empatados) — já estava no roadmap e agora tem dono
- `build_payload(attempt)` — o contrato que a Frente 2 consome
- Todos os testes matemáticos e de recomendação (os 12 atuais + os novos)
- Seção de metodologia do cálculo ([[artigo-secao-calculo-cosseno|✍️ artigo]])

### Não é seu
Definir o peso de cada curso (é F3, com validação de especialista), escrever o prompt, mexer em template.

> [!warning] O motor já está pronto — e isso é um problema de divisão, não uma vantagem
> `engine.py` está implementado e com 12 testes verdes. Se "ficar com o motor" significar só manter o que existe, sua frente acaba na primeira semana e as outras três afogam. Por isso o escopo acima **não é o código pronto**: é calibração com dados reais, sensibilidade, indicador de confiança e a defesa matemática escrita. É aí que está o trabalho — e é a parte que a banca mais vai apertar.

> [!success] Você é o guardião do contrato
> Como o motor está no meio do fluxo, toda mudança de formato passa por você. Isso é papel, não burocracia: quem estiver quebrando o contrato dos outros vai descobrir pelo seu teste falhando, não em cima da entrega.

---

## 2️⃣ 🤖 Camada de IA e experimento

**Fronteira:** recebe o payload do motor, devolve texto validado e a ordem final.

### Escopo
- Pacote `quiz/llm/` — protocolo, `FakeProvider`, `GeminiProvider`
- `quiz/prompts/entrega_v1.md` — o [[prompt-padrao-recomendacao|📝 prompt padrão]]
- `quiz/delivery.py` — orquestração, validação da saída, cache, timeout, fallback
- Persistência dos metadados (`llm_model`, `latency_ms`, tokens, `used_fallback`, `diverged`)
- **Passo 11 — o experimento comparativo**: N perfis pelos dois caminhos, taxa de divergência, avaliação humana
- Capítulo de engenharia de prompt e resultados comparativos

### Não é seu
Mudar o ranking do motor por código (só via prompt, e dentro do limiar), escrever tela.

> [!tip] Comece pelos Passos 7 e 8, que são 100% offline
> Essa frente é a única que depende de credencial externa — e é justamente por isso que ela foi desenhada para render **três passos sem internet nenhuma**. Se a chave da API atrasar, essa pessoa não fica parada.

---

## 3️⃣ 📚 Catálogo, trilha e integração

**Fronteira:** publica os cursos, os pesos e as trilhas. É a matéria-prima de todo o resto.

### Escopo
- Levantamento dos cursos reais e **recorte dos 18** que entram (critério documentado)
- **Atribuição dos pesos por área** — com validação de alguém que conheça os cursos, não por chute do grupo
- Seeds atualizados (`seed_areas`, `seed_courses`, `seed_questions`)
- **Modelo de trilha de cursos** — sequência/pré-requisitos, novo no banco
- **Registro de interesse** — o que acontece quando a pessoa clica "quero este curso"
- **Proposta de integração com SGSET/matrícula** — contrato de dados e ponto de entrada, documentado; sem código de integração (ver [[escopo-fluxo-educmatch|🗺️ recorte de escopo]])
- Capítulo sobre construção da base e método de atribuição de pesos

> [!warning] Esta é a frente do caminho crítico — ela precisa começar hoje
> A Frente 1 não calibra sem os cursos reais. A Frente 4 não tem dado de verdade para mostrar em dashboard. A Frente 2 gera texto sobre cursos fictícios. **Um atraso aqui atrasa as outras três**, e é a única frente que depende de coisas fora do computador (achar o catálogo, falar com quem conhece os cursos). Se o grupo tiver alguém com acesso fácil a essas informações, é essa pessoa que assume aqui.

> [!question] A pergunta que a banca vai fazer nesta frente
> *"Como vocês chegaram nesses pesos?"* — "achamos que fazia sentido" é a resposta que derruba. Um método declarado (ementa do curso → carga horária por eixo → nota de 0 a 5 → conferência com um instrutor) transforma a limitação mais óbvia do trabalho em metodologia.

---

## 4️⃣ 🎨 Jornada, avaliação e indicadores

**Fronteira:** consome a API e o resultado da entrega. Não toca em `engine.py` nem em `llm/`.

### Escopo
- Front: wizard do quiz e a **tela de entrega (1 curso principal + 4 alternativas)** com o selo de fallback
- Tela de trilha e de registro de interesse (visual; a regra é da F3)
- **Instrumento de satisfação**: perguntas, escala de 5 estrelas, anônima ou identificada
- **Avaliação final** e o piloto simulado que valida os dois instrumentos
- **Relatórios, dashboards e indicadores** — a coluna direita do quadro
- Coleta com respondentes reais (colegas), que alimenta o experimento da F2
- Capítulo de avaliação com usuários e indicadores

> [!note] Os indicadores são o que transforma o TCC em produto
> Enquanto o motor prova que o cálculo funciona, esta frente prova que **a instituição ganharia alguma coisa com isso**: quantas pessoas responderam, quais áreas puxam mais interesse, em quantos casos a IA discordou do cálculo, qual a satisfação. É o que responde "e daí?" depois da demonstração.

---

## 🚦 Ordem de partida e dependências

| Semana | F1 Motor | F2 IA | F3 Catálogo | F4 Jornada |
|---|---|---|---|---|
| **1** | Congelar `build_payload` | Passo 7 (offline) | **Levantar cursos reais** | Fechar contrato das telas |
| **2** | Testes de sensibilidade | Passo 8 (prompt + validação) | Recorte dos 18 + método de pesos | Tela de entrega 1+4 |
| **3** | **Recalibrar com dados reais** | Passo 9 (Gemini real) | Seeds reais + trilha | Instrumento de satisfação |
| **4** | Indicador de confiança | Passo 11 (experimento) | Registro de interesse | Dashboards + coleta real |
| **5** | Seção do cálculo escrita | Capítulo comparativo | Capítulo da base | Capítulo de avaliação |

> [!important] Ninguém espera o catálogo real para começar
> As seeds fictícias atuais continuam válidas como dado de desenvolvimento. F1, F2 e F4 trabalham em cima delas na semana 1 e 2, e a troca para os dados reais na semana 3 é só rodar os seeds de novo — **desde que ninguém tenha escrito o nome de um curso dentro do código**. Essa é a regra que sustenta o paralelismo.

## 🤝 Regras de convivência

1. **Uma branch por frente**, merge por pull request. Ninguém commita direto na main.
2. **Ninguém edita arquivo de outra frente.** Precisou de mudança lá? Pede — não conserta por conta.
3. **Mudança de contrato é decisão de grupo**, anunciada antes, nunca descoberta no merge.
4. **`manage.py test` verde é condição de merge.** Quebrou teste do outro, o merge não entra.
5. **Reunião semanal de 30 min**: cada frente responde "o que fechei / o que trava / o que preciso de quem".
6. **Cada um escreve o próprio capítulo**, na semana da entrega dele — não tudo na última semana.

## 📝 Monografia — quem escreve o quê

| Parte | Dono |
|---|---|
| Introdução e objetivos | consolidador (definir) |
| Fundamentação teórica (VSM, cosseno, cold start) | F1 |
| Metodologia — cálculo e calibração | F1 |
| Metodologia — construção da base e pesos | F3 |
| Metodologia — engenharia de prompt | F2 |
| Resultados — comparativo engine × híbrido | F2 |
| Resultados — avaliação com usuários e indicadores | F4 |
| Riscos, restrições, premissas (bloco de gestão do quadro) | consolidador, com insumo de todos |
| Considerações finais e trabalhos futuros | consolidador |

> [!tip] O papel de "consolidador" é um chapéu, não uma quinta frente
> Uma pessoa (que já tem uma das quatro frentes) fica responsável por unificar voz, formatação e referências. Sem isso, a monografia sai com quatro estilos diferentes — a banca percebe na primeira página.

## ❓ A decidir na próxima reunião

1. **Nomes das frentes 3 e 4** e quem é o consolidador — e **quem revisa os PRs do dono de F1+F2**
2. **Top-5 ou top-3 para a LLM** — o quadro e a decisão registrada divergem ([[escopo-fluxo-educmatch|🗺️ ponto 2]])
3. **Quantos cursos entram de verdade** — o quadro diz 18; confirmar e congelar
4. **Quem guarda a credencial** da API (pendência aberta em [[decisao-camada-ia|🤖 Decisão]])
5. **N do piloto** com respondentes reais e quando ele roda

## 📎 Veja também

- [[spec-motor-e-ia-frentes-1-2|🧭 Spec de atualização das Frentes 1 e 2]] — backlog, contratos e cronograma
- [[escopo-fluxo-educmatch|🗺️ Fluxo EducMatch e recorte de escopo]]
- [[camada-ia-plano-implementacao|🧩 Plano da camada de IA]] · [[prompt-padrao-recomendacao|📝 Prompt padrão]]
- [[engine-matching-cosseno|🧮 Engine]] · [[catalogo-areas-e-cursos|📚 Catálogo]] · [[front-templates-django|🎨 Front]]
- [[defesa-monografia-tcc|🎤 Defesa e monografia]] · [[testes-e-validacao-tcc|✅ Testes]]
- [[TCC|🎓 TCC]] · [[Projetos|🚀 Projetos]]
