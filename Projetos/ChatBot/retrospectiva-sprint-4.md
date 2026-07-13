---
title: "Retrospectiva — Sprint 4: Frontend do Inbox (React + TS + Tailwind)"
aliases: ["Retrospectiva — Sprint 4: Frontend do Inbox (React + TS + Tailwind)"]
tags: [react, typescript, tailwind, vite, sprint-4, retrospectiva, frontend, websocket, chatbot]
status: concluido
projeto: ChatBot
criado: 2026-07-13
---

> [!info] Projeto: [[ChatBot|💬 ChatBot]] · Tecnologias: [[React|React]] · [[TypeScript|TypeScript]] · [[Tailwind CSS|Tailwind]] · [[Vite|Vite]] · [[cs-websocket|WebSocket]] · [[JWT|JWT]]

# 📋 Retrospectiva — Sprint 4: Frontend do Inbox

> [!success] Status: CONCLUÍDO (validado ao vivo no navegador)
> O atendente **loga**, vê a **lista de conversas** numa sidebar (com avatares), abre uma conversa e lê as **mensagens** (com header do contato), e mensagens novas **aparecem sozinhas** na tela (WebSocket), sem refresh. Toda a base de tempo real da Sprint 3 (Redis) finalmente virou tela. Falta só o envio pelo atendente (depende da Meta).

## 🎯 Objetivo da Sprint
Dar cara à Fase 1: consumir no frontend os endpoints REST e o WebSocket que já existiam, fechando o ciclo login → conversas → mensagens → tempo real. Construído no modo **hands-on** (o dev escreve o código; specs + revisão a cada passo).

---

## ✅ O que foi entregue

- **Scaffold** Vite + React + TS + Tailwind v4 (`@tailwindcss/vite` + `@import "tailwindcss"` no `index.css`), ESLint. Pasta `frontend/` irmã de `backend/`.
- **Login** (`components/Login.tsx`) — form → `POST /api/auth/login/` → guarda `{access, refresh}` no storage (`lib/auth.ts`).
- **Portão de auth** (`App.tsx`) — `useState(!!getToken())` decide entre `Login` e `Inbox`; logout limpa token e volta pro login.
- **Cliente HTTP autenticado** (`lib/api.ts` → `apiFetch`) — injeta `Authorization: Bearer`, `Content-Type` json, e em **401** limpa token + recarrega (cai no login).
- **Lista de conversas** (`components/ConversationList.tsx`) — `GET /api/inbox/conversations/` com `useEffect`, destaque do selecionado.
- **Tela de conversa** (`components/MessagePanel.tsx`) — `GET .../{id}/messages/` em balões por `direction`; re-busca quando troca de conversa (`useEffect` com `[conversationId]`).
- **Layout** (`components/Inbox.tsx`) — sidebar + main, dono do estado `selectedId` (lifting state up).
- **Tempo real** — segundo `useEffect` no `MessagePanel` abre `ws://localhost:8000/ws/inbox/?token=`, filtra por `conversation_id` e adiciona a mensagem ao vivo; **cleanup** fecha o socket ao trocar de conversa.
- **Tipos** (`types.ts`) — `Conversation` e `Message` espelhando os serializers.
- **Polimento de UX** (`MessagePanel.tsx`) — **scroll automático** pro fim (`useRef` + `scrollIntoView({ behavior: "smooth" })` num `useEffect([messages])` com âncora `<div ref={bottomRef} />`) e **timestamps** (HH:MM via `toLocaleTimeString`) embaixo de cada balão.
- **Polimento visual do inbox** (avatares, header, estados) — inspirado num mock de app de chat:
  - **Avatares** na `ConversationList` — círculo colorido com a inicial do contato (`contact_name ?? contact_wa_id`); cor **determinística** por nome (hash do `charCodeAt` → paleta fixa), então o mesmo contato mantém sempre a mesma cor.
  - **Header do contato** no `MessagePanel` — avatar + nome + status da janela de atendimento (`is_within_service_window`). Pra isso o `Inbox` passou a guardar a **`Conversation` inteira** (não só o `id`) e o `MessagePanel` recebe `conversation` como prop.
  - **Estados de loading / erro / vazio** nos dois componentes — `useState(false)` de `error` setado no `.catch(...)` do fetch (resetado no início junto do `setLoading(true)`), com mensagens distintas pra carregando, erro e lista vazia ("Nenhuma conversa/mensagem ainda").
  - **Bolhas refinadas** — `rounded-2xl`, `max-w-[75%]`, inbound branco com borda sobre fundo `gray-50`, horário embaixo. Timestamp da última mensagem também na lista.

---

## 🧭 Decisões técnicas tomadas

| Decisão | Escolha | Motivo |
|---------|---------|--------|
| Stack do front | Vite + React + TS + Tailwind v4 | Padrão de mercado, HMR rápido, tipagem forte no consumo da API |
| Linter | **ESLint** (não oxlint) | Já vem no template do Vite; regras type-aware maduras; oxlint só compensa em codebase grande |
| Cliente HTTP | Um `apiFetch` único | Centraliza `Bearer` + `Content-Type` + tratamento de 401 — telas não repetem `fetch` na mão |
| Estratégia de 401 | Limpar token + `reload()` | Simples e eficaz; refresh automático fica pra depois |
| Onde mora o `selectedId` | No `Inbox` (pai) | Lista (destaca) e painel (busca) precisam do mesmo estado → *lifting state up* |
| Onde abre o WebSocket | No `MessagePanel` (por conversa) | Mantém o estado das mensagens local e ensina o **cleanup** do `useEffect`; filtra por `conversation_id` |
| Atualizar lista de mensagens | `setMessages((prev) => [...prev, msg])` | Forma funcional evita *stale closure* dentro do `onmessage` |
| Token no WebSocket | Query string `?token=` | Browser não deixa setar header no handshake (mesma razão da Sprint 3) |

---

## 🔧 Dificuldades encontradas

### 1. `App.tsx` perdeu o portão de login (loop de reload)
- **Sintoma:** o `App` renderizava sempre o `Inbox`, nunca o `Login`; abrir sem token levava a lista a tomar `401` → `apiFetch` recarregava → `Inbox` de novo → **loop infinito de reload**.
- **Causa:** tentativa de guardar o "logado" num módulo solto (`lib/logged`) em vez de estado do React; `setLogged` importado não re-renderiza.
- **Solução:** estado de auth é `useState` **dentro do `App`**, renderizando `Login` ou `Inbox` condicionalmente.

### 2. `MessagePanel` sem `return`
- **Sintoma:** a componente não renderizava nada (retornava `undefined`).
- **Causa:** o `{messages.map(...)}` ficou solto no corpo da função, fora de qualquer `return`.
- **Solução:** envolver o `.map` num `return (<div>...</div>)`. Padrão que se repetiu: em React, **JSX só aparece se estiver dentro de um `return`**.

### 3. Props não passadas pro filho
- **Sintoma:** clique na conversa não fazia nada; o `<main>` centralizava o painel torto.
- **Causa:** `<ConversationList />` sem `selectedId`/`onSelect`; `<main>` com classes de centralização sobrando.
- **Solução:** passar as props explicitamente e deixar o `<main>` só com `flex-1`.

### 4. WebSocket sem cleanup (conexões zumbis)
- **Sintoma:** o primeiro `useEffect` do WS abria a conexão mas nunca fechava.
- **Causa:** faltava o `return () => ws.close()`.
- **Solução:** retornar a função de limpeza — o React a chama ao trocar de conversa (mudança de `[conversationId]`) e ao desmontar. Sem ela, cada troca deixa um socket aberto acumulando.

### 5. Nomes de campo divergentes do serializer
- **Sintoma:** `types.ts` com `last_inbound_message`/`assigned_agent`.
- **Causa:** chute em vez de conferir o serializer.
- **Solução:** bater com o backend real: `last_inbound_at` e `assigned_to`. (TS não quebra em runtime, mas o campo viria `undefined`.)

---

## 🔁 Padrão recorrente identificado

> [!warning] Lição principal da Sprint
> O `useEffect` é o eixo de tudo no consumo de dados:
> - **`[]`** → roda uma vez na montagem (buscar a lista de conversas).
> - **`[conversationId]`** → re-roda quando a dependência muda (re-buscar mensagens ao trocar de conversa).
> - **`[messages]`** → re-roda quando a lista muda (rolar pro fim via `scrollIntoView` na âncora do `useRef`).
> - **`return () => ...`** → cleanup, para desfazer efeitos (fechar o WebSocket).
>
> E ao atualizar estado que depende do valor anterior (adicionar mensagem à lista), **usar a forma funcional** `set(prev => ...)` — dentro de um callback assíncrono (`onmessage`), o valor "direto" está congelado num closure velho.

---

## 🧪 Teste que fechou a Sprint (ao vivo, no navegador)

1. Sem token → tela de **Login**. Logando com `teste@zz.com` → sidebar de **Conversas**.
2. Clicar numa conversa → **mensagens** aparecem (entrada cinza à esquerda, saída azul à direita); trocar de conversa → recarrega.
3. Com a conversa aberta, disparar o webhook simulado pelo `fetch` no console (mesmo contato, `id` único via `Date.now()`) → o balão **aparece sozinho**, sem refresh. ✅

**Fluxo validado:** webhook → cria `Message` → `group_send("org_{id}")` → consumer → WebSocket → `onmessage` → `setMessages` → balão na tela.

---

## 📌 Estado final

- [x] Scaffold Vite + React + TS + Tailwind v4
- [x] Login com JWT + portão de auth no `App`
- [x] `apiFetch` (Bearer + 401)
- [x] Lista de conversas (REST)
- [x] Tela de conversa / mensagens (REST)
- [x] Tempo real via WebSocket (com cleanup e filtro por conversa)
- [x] Polimento inicial de UX: scroll automático pro fim + timestamps nos balões
- [x] Polimento visual: avatares (inicial + cor determinística), header do contato, estados de loading/erro/vazio
- [x] Commitado e pushado (frontend)
- [ ] Enviar mensagem pelo atendente (input + `POST`) — esbarra na conta Meta
- [ ] Reordenar/atualizar a sidebar ao vivo (hoje só a conversa aberta atualiza)
- [ ] Refresh automático do token (hoje: expirou → relogar)

---

## 🚀 Próximos passos

**Fecho da Fase 1 (lançável):**
1. Envio pelo atendente — depende da conta **Meta** (número + token). Cria `Message` out + envia pela Graph API (`send_text_message` já existe).
2. Reordenar/atualizar a sidebar ao vivo (hoje só a conversa aberta atualiza em tempo real).
3. Deploy: servir com Daphne/ASGI + Redis gerenciado.

**Fase 2 — Construtor de chatbot** (só depois de validar a Fase 1 com cliente real):
- Motor de fluxo no backend (state machine por contato) + editor visual com React Flow. Spec de arquitetura já rascunhada.

---

## 📚 Referências

- [[Retrospectiva — Sprint 3: Inbox em Tempo Real (WebSocket)]]
- [[Retrospectiva — Sprint 2: WhatsApp Cloud API (Webhook + Client)]]
- [[Retrospectiva — Sprint 1: Fundação e Multi-Tenancy]]
- [[Guia de Implementação — Fase 1: Inbox Multi-Atendente]]
- [[Roadmap — Plataforma de Atendimento WhatsApp]]
