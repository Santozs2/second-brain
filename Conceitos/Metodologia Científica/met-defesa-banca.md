---
type: concept
area: Conceitos
status: estavel
difficulty: beginner
id: met-defesa-banca
category: Metodologia Científica
tags:
  - metodologia
  - concept
  - tcc
created: 2026-08-24
updated: 2026-08-24
---
# 🎤 Defesa de Banca

> A banca não está tentando reprovar você. Está verificando se você entende o que construiu — o que é uma pergunta diferente de se o que você construiu é bom.

---

## ⏱️ A estrutura de uma apresentação de 20 minutos

| Tempo | Bloco | Conteúdo |
|---|---|---|
| 0–2 min | **Problema** | O contexto e a dor, sem jargão |
| 2–4 min | **Objetivo** | O que o trabalho se propôs a fazer |
| 4–7 min | **Fundamentação** | Só o necessário para entender a escolha |
| 7–11 min | **Método** | Como foi feito, e por que assim |
| 11–15 min | **Demonstração** | O sistema funcionando |
| 15–18 min | **Resultados** | Os números e o que eles dizem |
| 18–20 min | **Limitações e futuro** | O que não foi feito e por quê |

> [!important] Demonstração gravada, não ao vivo
> Rede cai, dependência quebra, o notebook não conecta no projetor. **Grave um vídeo da demonstração e leve o ao vivo como bônus.** Um trabalho excelente que não roda no dia parece um trabalho que não roda.

---

## 🎯 O slide que decide a apresentação

Em algum ponto entre o método e os resultados, um slide precisa responder: **"o que existe no mundo agora que não existia antes deste trabalho?"**

Não é a lista de tecnologias. É a contribuição.

```
❌ "Desenvolvemos um sistema em Django com React"
✅ "Um método de recomendação que entrega ranking com justificativa
   rastreável ao cálculo, validado com N usuários"
```

---

## ❓ As perguntas que sempre aparecem

### "Por que vocês escolheram [técnica X] e não [técnica Y]?"

A pergunta mais frequente. A resposta ruim é "porque foi o que aprendemos". A boa tem três partes:

```
1. O critério que decidiu     → "o regime de dados"
2. A alternativa e por que não → "Y exige histórico, que não existe no dia zero"
3. A fonte                     → "conforme Schein et al. (2002)"
```

### "Como vocês chegaram nesses valores/pesos/parâmetros?"

Mata trabalhos. "Achamos que fazia sentido" é fatal. Precisa existir um **método declarado**, mesmo que simples:

> "Cada item foi avaliado a partir de sua ementa, considerando a carga horária dedicada a cada eixo, convertida em nota de 0 a 5, com conferência por um profissional da área."

Um método simples e declarado vence um método sofisticado e não documentado.

### "Isto não é só um CRUD?"

Responda apontando o **núcleo não-trivial**: o algoritmo, a decisão de arquitetura, o experimento. O CRUD existe para servir o núcleo.

### "Como vocês sabem que funciona?"

Aponte a avaliação. Se a avaliação é fraca, **declare-a fraca antes que perguntem** — e diga o que a fortaleceria.

### "E se a API/serviço externo cair?"

Se existe fallback, esta é uma pergunta que você quer que façam. Se não existe, é a pergunta que você não quer.

### "O que vocês fariam diferente?"

Não responda "nada". Não responda listando erros pequenos. Responda com **uma** decisão de arquitetura que você tomaria diferente e por quê. Demonstra reflexão.

---

## 🧠 Como responder ao que você não sabe

```
"Não avaliamos esse aspecto neste trabalho. Isso decorre de
[razão], e seria abordado por [como]."
```

> [!success] "Não sei" bem estruturado é melhor que uma resposta inventada
> A banca é composta por pessoas que conhecem a área. Uma resposta inventada é identificada em segundos e contamina a percepção de tudo que você disse antes. Reconhecer o limite, apontar a causa e propor o caminho demonstra maturidade — e encerra a linha de questionamento.

---

## 🚩 Erros que custam nota

| Erro | Consequência |
|---|---|
| Ler os slides | Sinaliza que você não domina o conteúdo |
| Slides com parágrafos | Ninguém lê; competem com sua fala |
| Estourar o tempo | Corta os resultados, que é o que importa |
| Mostrar código na tela | Ninguém consegue ler; use diagrama |
| Discutir com a banca | Mesmo quando você está certo |
| Atribuir falha ao grupo | "Fulano não fez a parte dele" derruba todo mundo |
| Prometer o que não entregou | A banca leu a monografia |

---

## ✅ Checklist da véspera

- [ ] Vídeo da demonstração gravado e testado
- [ ] Apresentação exportada em PDF (fonte não quebra)
- [ ] Arquivos em pendrive **e** na nuvem
- [ ] Tempo cronometrado em ensaio completo
- [ ] Uma resposta preparada para cada pergunta desta nota
- [ ] A seção de limitações relida — é de lá que vêm as perguntas
- [ ] Quem fala qual parte, definido e ensaiado
- [ ] Uma frase pronta para "o que vocês fariam diferente"

---

## 🤝 Defesa em grupo

- Cada pessoa apresenta **a própria frente**, e precisa saber responder por ela
- Todos precisam conseguir explicar o **núcleo** do trabalho, não só a própria parte
- Combine antes quem responde o quê, para não haver silêncio nem atropelo
- Nunca exponha conflito interno diante da banca

> [!warning] A banca costuma perguntar a alguém sobre a parte de outro
> É um teste deliberado de se o grupo trabalhou junto ou dividiu em quatro projetos. Todo mundo precisa saber explicar, em uma frase, o que cada frente fez e como elas se conectam.

---

## 🔗 Conceitos relacionados

- [[met-estrutura-monografia|📄 Estrutura da monografia]] · [[met-validade-e-limitacoes|🎯 Validade e limitações]]
- [[met-tipos-de-pesquisa|🔬 Tipos de pesquisa]] · [[met-metodo-cientifico|🔬 Método científico]]

## Veja também

- [[Conceitos|🧠 Conceitos]]

---

**Status:** ✅ Completo
