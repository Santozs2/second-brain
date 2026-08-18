# Material de Validação — Produto ChatBot (Atendimento WhatsApp)

> Objetivo desta fase: descobrir se alguém **paga** por isso, ANTES de gastar tempo com deploy/infra.
> Meta concreta: falar com 5-10 negócios e conseguir 1 "quero testar".

---

## 0. Alvo ideal (com quem falar)

Negócios que **já usam WhatsApp pra vender/atender** e sofrem com volume:
- Clínicas (odonto, estética, fisio), petshops, salões/barbearias
- Lojas (roupa, calçado, peças) que vendem por WhatsApp
- Prestadores: assistência técnica, imobiliárias pequenas, escritórios
- Delivery / hortifruti / distribuidoras

Sinais de que é um bom alvo:
- Responde cliente no celular pessoal do dono/atendente
- Perde mensagem, demora pra responder, ou tem 1 pessoa só no zap
- Repete as mesmas respostas o dia todo (horário, preço, endereço)

Comece por quem você **já conhece** (dono conhecido, indicação da família). Fricção zero.

---

## 1. One-page do Produto

**Nome provisório:** _(defina — ex.: ZapDesk, Atende Zap, InboxZap)_

**Uma frase:** Central de atendimento no WhatsApp com robô de respostas — sua equipe atende junto, sem perder cliente, e o bot responde o básico sozinho.

**O problema (dor real):**
- Mensagem de cliente se perde no celular pessoal
- Só uma pessoa consegue responder por vez
- O dono responde a mesma coisa 50x por dia (horário, preço, endereço)
- Quando o atendente sai, o histórico vai junto

**A solução:**
- **Caixa de entrada compartilhada** — vários atendentes no mesmo número, cada conversa organizada
- **Robô (chatbot) que você monta arrastando blocos** — responde perguntas comuns, coleta dados e chama um humano quando precisa
- **Tudo em tempo real** — mensagem nova aparece na hora pra equipe

**Benefícios (o que o cliente ganha):**
- Não perde mais mensagem = não perde venda
- Responde mais rápido = cliente mais satisfeito
- Bot tira o trabalho repetitivo da equipe
- Histórico fica na empresa, não no celular de ninguém

**Prova (o que já está pronto):**
- Inbox multi-atendente funcionando
- Construtor de fluxo visual (arrasta os blocos)
- Integração real com a API oficial do WhatsApp (Meta)

**Preço sugerido pra piloto:** _(ex.: R$ X/mês por atendente, ou R$ Y/mês fixo até Z atendentes)_
→ No piloto, cobre BARATO ou dê desconto em troca de feedback + depoimento.

---

## 2. Roteiro do vídeo demo (2 minutos)

Grave a tela (OBS/Loom/celular). Fale de forma simples, como se explicasse pra um dono de loja.

**Cena 1 (0:00-0:20) — O problema**
> "Sabe quando chega um monte de mensagem no WhatsApp e alguma se perde? Ou quando só uma pessoa consegue responder? Isso resolve isso."

**Cena 2 (0:20-0:50) — A caixa de entrada**
- Mostra a lista de conversas + uma conversa aberta
- Manda uma mensagem de um celular e ela **aparece na hora** na tela
> "Todas as conversas num lugar só, a equipe inteira vê. Chegou, aparece na hora."

**Cena 3 (0:50-1:30) — O robô**
- Abre o construtor de fluxo, mostra arrastando 2-3 blocos (mensagem → pergunta → chamar humano)
- Publica e mostra o bot respondendo
> "E esse robô aqui você monta arrastando. Ele responde o básico sozinho e, quando precisa, chama uma pessoa."

**Cena 4 (1:30-2:00) — Fechamento**
> "Isso já tá funcionando. Tô procurando alguns negócios pra testar de graça/desconto e me dar feedback. Se fizer sentido pro seu, bora conversar."

Dica: grave 2-3 vezes, use a melhor. Não precisa ser perfeito, precisa ser CLARO.

---

## 3. Script de abordagem

### 3a. Mensagem fria (WhatsApp/DM) — curta
> Oi [nome], tudo bem? Eu desenvolvi uma ferramenta que organiza o atendimento de WhatsApp da empresa — vários atendentes no mesmo número + um robô que responde as perguntas repetidas. Tô procurando alguns negócios pra testar antes de lançar. Posso te mandar um vídeo de 2 min mostrando? Sem compromisso.

Se responder "pode": manda o vídeo. Se curtir: marca uma conversa de 15 min.

### 3b. Roteiro da conversa (call/presencial, 15 min)
1. **Escuta primeiro (não venda ainda):**
   - "Como vocês atendem no WhatsApp hoje?"
   - "Quantas pessoas respondem? É o celular de quem?"
   - "Já perderam cliente por demora ou mensagem perdida?"
   - "Quais perguntas vocês mais repetem?"
2. **Conecte a dor com a solução** (só as partes que a dor DELE pediu)
3. **Proposta de piloto:**
   > "Topa testar por [30 dias] com [preço piloto]? Eu configuro pra você, e em troca só quero seu feedback honesto."
4. **Fecha o próximo passo** (data pra configurar, o que você precisa dele)

---

## 4. Perguntas que você PRECISA responder nesta fase

Anote as respostas de cada conversa. O objetivo é aprender, não vender a qualquer custo:

- [ ] Essa dor é real e forte pra ele? (ou ele só foi educado?)
- [ ] Ele **pagaria**? Quanto? (se travar no preço, a dor não é forte o suficiente)
- [ ] Qual feature ele mais quer? (talvez não seja o bot — talvez seja só o inbox)
- [ ] O que falta pra ele dizer SIM hoje?
- [ ] Ele tem WhatsApp Business API ou usa o app comum? (impacta a conexão)

**Regra:** se 5 pessoas ouvirem e ninguém topar nem um piloto barato, o problema não é o produto pronto — é a proposta/alvo. Ajuste antes de codar mais.

---

## 5. Checklist da fase de validação

- [ ] Definir o nome do produto
- [ ] Definir preço de piloto
- [ ] Gravar o vídeo demo de 2 min
- [ ] Listar 10 negócios-alvo (começar pelos conhecidos)
- [ ] Mandar a mensagem fria pros 10
- [ ] Fazer pelo menos 5 conversas de 15 min
- [ ] Conseguir 1 "quero testar"
- [ ] (Em paralelo) resolver o responsável legal maior de idade

→ Só depois de conseguir o 1º interessado: partir pro deploy + Celery + token permanente.
