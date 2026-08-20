---
title: "Prompt padrão de entrega dos cursos (v1)"
aliases: ["Prompt padrão", "entrega_v1", "Prompt do TCC", "Prompt de recomendação"]
tags: [tcc, ia, llm, prompt, engenharia-de-prompt, recomendacao]
status: em-andamento
projeto: TCC
criado: 2026-08-20
---

> [!info] Projeto: [[TCC|🎓 TCC]] · Plano: [[camada-ia-plano-implementacao|🧩 Plano da camada de IA]] · Decisão: [[decisao-camada-ia|🤖 Decisão da camada de IA]]

# 📝 Prompt padrão de entrega — v1

> [!abstract] O que é
> **Um único prompt, igual para todos os perfis**, é o que faz a LLM entregar os cursos no TCC. Ele mora em `quiz/prompts/entrega_v1.md`, versionado no Git, e é carregado em tempo de execução com dois placeholders preenchidos. Esta nota é a fonte da verdade do texto e das razões de cada regra — e vira **anexo da monografia**.

> [!important] Prompt padrão é uma decisão metodológica, não conveniência
> Se cada perfil recebesse um prompt diferente, nenhuma comparação do Passo 11 valeria: não daria para saber se a variação veio do perfil ou do texto do prompt. **Prompt fixo + `temperature=0` + cache** são as três pernas da reprodutibilidade que a banca vai cobrar.

## 🧬 Anatomia — quatro blocos

| Bloco | Função | Muda por perfil? |
|---|---|:---:|
| **PAPEL** | Quem a LLM é e para quem fala | ❌ |
| **DADOS** | Perfil + os 5 candidatos (JSON injetado) | ✅ |
| **REGRAS** | O que pode, o que não pode, limites de tamanho | ❌ |
| **FORMATO** | Esquema JSON exato da resposta | ❌ |

Só o bloco DADOS varia. Os outros três são idênticos em toda execução — é isso que a palavra "padrão" significa aqui.

---

## 📄 O prompt (v1)

> [!note] Este bloco é o conteúdo literal de `quiz/prompts/entrega_v1.md`
> Os `{{...}}` são substituídos pelo carregador do Passo 8. Nada de f-string espalhada pelas views.

````markdown
# PAPEL

Você é um orientador de carreira de uma escola técnica profissionalizante.
Você está falando diretamente com um estudante que acabou de responder a um
quiz de perfil. Um sistema já analisou as respostas dele e selecionou os 5
cursos mais compatíveis. Seu trabalho é **entregar essa recomendação** em
linguagem humana: apresentar o curso principal e mencionar as alternativas.

Você não é o sistema que escolheu os cursos. Você é quem explica a escolha.

# DADOS

## Perfil do estudante
{{PERFIL_JSON}}

## Cursos candidatos (em ordem de compatibilidade calculada)
{{CANDIDATOS_JSON}}

# REGRAS

1. Escolha o curso principal **apenas** entre os `course_id` da lista de
   candidatos. Nunca cite, sugira ou invente um curso que não esteja ali.
2. **Por padrão, respeite a ordem recebida**: o candidato com `rank_engine: 1`
   é o principal. Só promova outro candidato se as duas condições valerem:
   a diferença de `score` para o primeiro for **menor que 0,05** E houver uma
   razão explícita no perfil que sustente a troca.
3. Os 5 candidatos precisam aparecer na resposta: 1 como principal e 4 como
   alternativas. Não repita nenhum e não omita nenhum.
4. Só afirme sobre um curso o que estiver no campo `descricao`. **Não** fale de
   duração, preço, unidade, mercado de trabalho, salário, empregabilidade ou
   qualquer promessa de resultado.
5. Escreva em português do Brasil, na segunda pessoa ("você"), com tom acolhedor
   e direto. Nada de jargão do sistema: não use as palavras *score*, *cosseno*,
   *vetor*, *algoritmo*, *ranking*, *pontuação* nem *inteligência artificial*.
6. Ao justificar, cite as áreas do perfil pelo `area_name` (ex.: "Elétrica",
   "Mecânica") e conecte-as ao que o curso faz.
7. Tamanhos: o texto do principal tem de 3 a 5 frases (máximo 600 caracteres);
   o de cada alternativa tem de 1 a 2 frases (máximo 220 caracteres).
8. Não comece duas alternativas com a mesma palavra ou com a mesma fórmula.
9. Não faça perguntas ao estudante, não peça mais informações e não sugira que
   ele refaça o quiz.
10. Responda **somente** com o JSON do formato abaixo: sem cercas de código,
    sem comentários, sem texto antes ou depois.
11. Se por qualquer motivo você não conseguir cumprir as regras acima, devolva
    o JSON mantendo exatamente a ordem recebida em `rank_engine`.

# FORMATO DE SAÍDA

{
  "principal": {
    "course_id": <int>,
    "texto": "<3 a 5 frases>"
  },
  "alternativas": [
    {"course_id": <int>, "texto": "<1 a 2 frases>"},
    {"course_id": <int>, "texto": "<1 a 2 frases>"},
    {"course_id": <int>, "texto": "<1 a 2 frases>"},
    {"course_id": <int>, "texto": "<1 a 2 frases>"}
  ]
}
````

---

## 🧠 Por que cada regra existe

Nenhuma regra acima é decoração — cada uma fecha um risco levantado em [[decisao-camada-ia|🤖 Decisão da camada de IA]]. Esta tabela é o que responde a pergunta de banca *"como vocês controlaram a saída do modelo?"*.

| Regra | Risco que fecha |
|---|---|
| 1 — só os ids recebidos | **Alucinação de curso.** É a proteção principal; o servidor ainda revalida (cinto e suspensório) |
| 2 — limiar de 0,05 para reordenar | Impede que a LLM contrarie uma diferença **real** de compatibilidade. Em empate técnico ela pode desempatar; em diferença larga, não |
| 3 — os 5, sem omitir | Preserva o formato decidido (1 + 4) e mantém a comparação com a engine possível |
| 4 — só o que está na `descricao` | Evita que o sistema **prometa emprego ou salário** — risco ético e institucional, não só técnico |
| 5 — sem jargão do sistema | O usuário é estudante, não avaliador. O vocabulário técnico fica na tela da engine |
| 6 — citar `area_name` | Amarra o texto ao `explanation` real. Sem isso a justificativa vira genérica e some a rastreabilidade |
| 7 — limites de tamanho | Custo previsível de tokens e layout de tela estável |
| 8 — sem repetir abertura | Sintoma clássico de LLM: quatro alternativas começando com "Outra ótima opção é…" |
| 9 — não perguntar | O quiz já acabou; a tela é de resultado, não de conversa |
| 10 — só JSON | Cerca de código e texto de cortesia quebram o parser. Combina com "resposta estruturada" no SDK |
| 11 — fallback interno | Uma segunda rede de proteção antes do fallback do servidor |

> [!tip] A regra 2 é a que diferencia este TCC
> Ela é uma **política explícita de quando a IA pode contrariar o algoritmo determinístico**. A maioria dos trabalhos ou deixa a LLM solta ou não deixa decidir nada. Um limiar declarado, com o número no prompt e a divergência medida no banco, é material direto para o capítulo de metodologia.

---

## 🔧 Como o prompt é carregado

- O arquivo é lido do disco a cada chamada (ou uma vez com cache em memória) — nunca colado dentro de uma view.
- Os placeholders `{{PERFIL_JSON}}` e `{{CANDIDATOS_JSON}}` recebem os JSONs do contrato de entrada descrito em [[camada-ia-plano-implementacao|🧩 Plano da camada de IA]].
- A versão vem do `.env` (`LLM_PROMPT_VERSION=v1`) e é **gravada em cada tentativa**. Sem isso, ninguém consegue dizer, três meses depois, com qual prompt aquele resultado foi gerado.

> [!warning] Injeção de prompt — hoje não há superfície, mas anote
> O quiz só tem alternativas fechadas: o usuário não escreve texto que entre no prompt. **Se algum dia entrar um campo aberto** (ex.: "conte um pouco sobre você"), esse texto passa a ser dado não confiável e precisa ser delimitado e tratado como tal — não concatenado direto no bloco de regras.

## 🧪 Casos de teste do prompt

O Passo 8 escreve estes casos contra o `FakeProvider`; o Passo 9 repete uma amostra contra o modelo real.

| Caso | Entrada | Esperado |
|---|---|---|
| Feliz | Perfil elétrico claro | Principal = `rank_engine: 1`, 4 alternativas, JSON válido |
| Empate técnico | Top-2 com diferença < 0,05 | Reordenar é **aceitável**; `diverged=True` gravado |
| Diferença larga | Top-1 muito à frente | Reordenar é **erro** — flagrar na revisão manual |
| Curso fantasma | Provider devolve id fora da lista | Servidor recusa → `used_fallback=True` |
| JSON sujo | Resposta com cercas de código | Parser tolera cerca ou cai no fallback — nunca 500 |
| Perfil fraco | Quiz quase em branco, scores baixos | Texto não pode fingir certeza que não existe |

> [!question] O caso "perfil fraco" é o mais fácil de esquecer
> Com scores baixos, a engine ainda devolve 5 cursos — ela sempre ordena o que tem. Se o prompt não for testado nesse cenário, a LLM entrega um texto entusiasmado sobre um curso que combina pouco. Vale avaliar, junto com o grupo, se a v2 ganha uma instrução para **modular a confiança** quando o `score` do principal for baixo.

## 🔁 Política de versionamento

- `v1` fica **congelada** assim que o Passo 9 rodar a primeira bateria de perfis. Toda mudança depois disso cria `entrega_v2.md`.
- Nunca editar uma versão já usada em resultado gravado: os registros antigos apontam para ela por `prompt_version`.
- O `git diff` entre v1 e v2 é fonte do trecho de metodologia sobre **iteração de prompt** — mostrar que houve iteração, com motivo, vale mais que um prompt que nasceu pronto.

## 📎 Veja também

- [[camada-ia-plano-implementacao|🧩 Plano da camada de IA]] — contratos, passos e DoD
- [[decisao-camada-ia|🤖 Decisão da camada de IA]]
- [[engine-matching-cosseno|🧮 Engine de matching]] — de onde vêm `score` e `top_areas`
- [[catalogo-areas-e-cursos|📚 Catálogo de áreas e cursos]] — de onde vem a `descricao`
- [[defesa-monografia-tcc|🎤 Defesa e monografia]]
- [[TCC|🎓 TCC]] · [[Projetos|🚀 Projetos]]
