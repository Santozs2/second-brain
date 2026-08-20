---
title: "EducMatch — fluxo do quadro e recorte de escopo"
aliases: ["EducMatch", "Fluxo do quadro", "Escopo do TCC", "Quadro branco TCC"]
tags: [tcc, escopo, fluxo, planejamento, gestao, produto]
status: em-andamento
projeto: TCC
criado: 2026-08-20
---

> [!info] Projeto: [[TCC|🎓 TCC]] · Divisão: [[divisao-de-trabalho-tcc|👥 Divisão de trabalho]] · Motor: [[engine-matching-cosseno|🧮 Engine]] · IA: [[camada-ia-plano-implementacao|🧩 Camada de IA]]

# 🗺️ EducMatch — o fluxo desenhado no quadro

> [!abstract] O que é esta nota
> Transcrição e análise do quadro branco desenhado pelo grupo em 2026-08-20. O quadro é **maior que o sistema que existe hoje**: ele descreve o produto completo (do link até os indicadores institucionais), enquanto o vault documenta a parte já construída (quiz → motor → resultado). Esta nota separa as duas coisas e faz o **recorte de escopo** — o que entra no TCC, o que fica como proposta e o que sai.

> [!note] O projeto tem nome: **EducMatch**
> Apareceu no canto do quadro e não estava em lugar nenhum do vault. Vale adotar oficialmente na monografia — um nome de produto ajuda a banca a separar "o sistema" de "o algoritmo".

---

## 📋 O que o quadro diz, bloco a bloco

### 1️⃣ Entrada e recomendação (preto, esquerda)
```
pessoa → SENAI → link → formulário/quiz
                          ↓
              recomendações → análise de perfil → (IA)
                          ↓
                 180 cursos → 18 cursos → trilha de cursos
                          ↓
                 registro de interesse
                          ↓
              ? integração com sistema institucional
```

### 2️⃣ Matrícula e acompanhamento (preto)
```
SGSET → matrícula → durante o curso (etapa nova)
                      ↓
        pesquisa de satisfação (anônima ou identificada)
                      ↓
        informa o curso → nota de 1 a 5 estrelas
```

### 3️⃣ Avaliação final e indicadores (preto)
```
ao final → avaliação final → visão geral
                              ↓
              relatórios · dashboards · indicadores
```

### 4️⃣ O motor (vermelho)
```
perfil × curso → (A · B) / (‖A‖ × ‖B‖) → 0,00 a 1,00 → % → curso recomendado
alternativas com peso: A→5, B→3, C→…, D→…
```
Mais a pergunta circulada: **"como escrever o artigo?"**

### 5️⃣ A camada de IA (azul)
```
80% exato, baseado em matemática
        ↓
LLM → "prompt" → GEMINI
        ↓
curso + perfil + contexto institucional → resposta
faixas 80% / 70% / … encaminhadas para a LLM
```

### 6️⃣ Gestão do projeto (preto, direita)
```
STOP → MAPEAR → RISCOS → RESTRIÇÕES → PREMISSAS → CONCLUSÃO → implantação
sujeito de exemplo: 90% costura · 5% elétrica · 5% TI
```

---

## 🔍 Quadro × vault — o que é novo

| Item do quadro | Estado no vault | Leitura |
|---|---|---|
| Quiz → motor → resultado | ✅ construído e testado | O quadro confirma o que existe |
| Cálculo por cosseno, 0 a 1 | ✅ [[engine-matching-cosseno\|engine]] | Idêntico |
| LLM com prompt padrão via Gemini | ✅ [[decisao-camada-ia\|decidido ontem]] | Idêntico |
| **180 cursos → 18 cursos** | ⚠️ hoje são **12 cursos fictícios** | **Novo e pesado** — ver alerta abaixo |
| **Trilha de cursos** | ❌ não existe no modelo de dados | Novo — feature real e viável |
| **Registro de interesse** | ❌ não existe | Novo — barato de fazer |
| **Matrícula / SGSET** | ❌ não existe | Novo — **dependência externa** |
| **Pesquisa de satisfação durante o curso** | ❌ não existe | Novo — depende de tempo real de curso |
| **Avaliação final** | ❌ não existe | Novo — mesma dependência |
| **Dashboards e indicadores** | ❌ não existe | Novo — viável e rende capítulo |
| Riscos / restrições / premissas | ❌ não escrito | Novo — é capítulo de gestão |

---

## ⚠️ Cinco pontos que o grupo precisa resolver antes de programar

> [!warning] 1. "180 cursos" é o maior risco de cronograma do quadro inteiro
> O motor não depende da **quantidade** de cursos — cadastrou, entra no cálculo. O que não escala é o **peso**: cada curso precisa de 7 notas de área atribuídas por julgamento humano. São até **1.260 decisões** para 180 cursos. Por isso o próprio quadro já escreve **18 cursos**: esse é o recorte executável. Definir explicitamente na monografia: *"a base foi recortada em 18 cursos representativos das 7 áreas; a atribuição de pesos é o gargalo, não o algoritmo"*.

> [!warning] 2. O quadro manda 3 faixas para a LLM; a decisão registrada é top-5
> No bloco azul aparecem três percentuais encaminhados à LLM. A decisão fechada em [[decisao-camada-ia|🤖 Decisão]] é **top-5 → 1 principal + 4 alternativas**. As duas coisas não conversam. Precisa escolher **uma** — e minha recomendação é manter o top-5, porque o formato 1+4 já está desenhado e mandar 5 candidatos custa praticamente o mesmo que mandar 3.

> [!warning] 3. SGSET e matrícula são dependência de terceiros
> Nenhum grupo de TCC consegue garantir acesso de escrita a um sistema acadêmico institucional. Tratar como **proposta de integração documentada** (contrato de dados + ponto de entrada + o que seria necessário), não como código a entregar. Se sair, é bônus; se não sair, o TCC não perde nada — e a proposta bem-feita ainda vale um capítulo.

> [!warning] 4. Satisfação "durante o curso" não cabe no calendário
> Medir satisfação durante e ao final de um curso exige que alguém **se matricule e faça o curso** — meses. O que cabe no TCC é: desenhar o instrumento (as perguntas, a escala de 5 estrelas, anônima ou identificada), implementar a coleta e rodar um **piloto simulado**. A monografia declara isso como limitação, exatamente como já se faz com os perfis sintéticos ([[defesa-monografia-tcc|🎤 Defesa]]).

> [!note] 5. Detalhe de nomenclatura
> O quadro escreve "cálculo de seno"; o que está na fórmula é **cosseno**. Vale corrigir antes que vire slide.

---

## ✂️ Recorte de escopo proposto

| Camada | Entra no TCC? | Forma da entrega |
|---|:---:|---|
| Quiz + motor de cosseno | ✅ | Implementado (feito) |
| Camada de IA com prompt padrão | ✅ | Implementado (Passos 7–11) |
| Catálogo de **18 cursos reais** + pesos | ✅ | Implementado + método de atribuição documentado |
| Trilha de cursos | ✅ | Implementado (modelo + tela) |
| Registro de interesse | ✅ | Implementado |
| Dashboards e indicadores | ✅ | Implementado com dados do piloto |
| Instrumento de satisfação e avaliação final | 🟡 | Implementado, validado em **piloto simulado** |
| Matrícula / integração SGSET | 📄 | **Proposta documentada**, sem código de integração |
| 180 cursos completos | ❌ | Declarado como trabalho futuro |

> [!success] O critério que decide tudo isso
> **O que a banca pode ver funcionando no dia da defesa entra; o que depende de terceiros ou de meses de calendário vira proposta ou limitação declarada.** Um TCC não é avaliado por prometer um produto grande — é avaliado por entregar um recorte inteiro e saber dizer por que recortou.

## 📎 Veja também

- [[divisao-de-trabalho-tcc|👥 Divisão de trabalho entre as 4 frentes]]
- [[decisao-camada-ia|🤖 Decisão da camada de IA]] · [[camada-ia-plano-implementacao|🧩 Plano da camada de IA]]
- [[engine-matching-cosseno|🧮 Engine de matching]] · [[catalogo-areas-e-cursos|📚 Catálogo atual]]
- [[defesa-monografia-tcc|🎤 Defesa e monografia]]
- [[TCC|🎓 TCC]] · [[Projetos|🚀 Projetos]]
