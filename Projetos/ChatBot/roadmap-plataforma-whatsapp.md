---
title: "Roadmap — Plataforma de Atendimento WhatsApp"
aliases: ["Roadmap — Plataforma de Atendimento WhatsApp"]
projeto: ChatBot
tags: [projeto, saas, whatsapp, roadmap]
status: planejamento
criado: 2026-07-06
---

> [!info] Projeto: [[ChatBot|💬 ChatBot]]

# 🚀 Plataforma de Atendimento e Automação WhatsApp

> [!abstract] Visão geral
> SaaS de atendimento e automação via **WhatsApp Cloud API (oficial)**, inspirado no Umbler Talk mas com marca, UX e diferencial próprios.

## 🧭 Estratégia
- **Motor horizontal** (serve qualquer negócio), mas **entrada de mercado por um nicho só** nos primeiros 5–10 clientes — mais fácil validar, precificar e conseguir depoimento.
- Por cima do motor genérico, criar **pacotes de partida por segmento** (fluxos + campos + tags prontos).
- **Via de envio:** WhatsApp Cloud API oficial (sem risco de ban, escalável).

### Stack
- **Backend:** Django + DRF + Celery + Redis + Postgres
- **Frontend:** React + TypeScript + Tailwind
- **Construtor de fluxo:** React Flow
- **Tempo real:** Django Channels *ou* Supabase Realtime
- **IA:** Claude/GPT + RAG (reaproveitar Qdrant do AMEA)

---

## 🎯 Fase 0 — Fundação
**~1 semana**
- [ ] Registrar conta Meta Business + WhatsApp Cloud API *(começa já — verificação demora)*
- [x] Definir o nicho de entrada ✅ 2026-07-06
- [x] Definir multi-tenancy (isolamento de dados por empresa) ✅ 2026-07-06
- [x] Fechar stack e estrutura de repositório ✅ 2026-07-06

## 🏗️ Fase 1 — MVP: Inbox compartilhado ⭐ *o coração*
**~3–4 semanas**
- [ ] Multi-tenancy + auth + RBAC (dono / admin / atendente)
- [ ] Integração Cloud API: enviar/receber mensagens + webhook de status
- [ ] Inbox em tempo real (WebSocket): lista de conversas, atribuição a atendente, envio de texto/mídia
- [ ] Janela de 24h (texto livre) vs template (fora da janela)

> [!tip] Ponto de lançamento
> **Lançável aqui.** Inbox multi-atendente já é um produto vendável. Validar com clientes reais **antes** de construir o chatbot visual.

## 🤖 Fase 2 — Construtor de chatbot 🔥 *diferencial mais difícil*
**~4–6 semanas**
- [ ] Motor de fluxo no backend (nós: mensagem, pergunta, condição, ação, transferir p/ humano)
- [ ] Editor visual no front com **React Flow**
- [ ] Execução por contato (state machine) + captura de variáveis e etiquetagem por origem
- [ ] Fallback pra atendente humano

## 📇 Fase 3 — CRM / Board de contatos
**~2–3 semanas**
- [ ] Kanban de contatos, tags, campos customizados, histórico por contato
- [ ] Segmentação pra campanhas

## 📢 Fase 4 — Disparo de campanhas + agendamento
**~2–3 semanas**
- [ ] Envio em massa via template com rate limit por tier
- [ ] Agendamento de mensagens
- [ ] Opt-out automático + log de consentimento (LGPD)
- [ ] Relatório de entrega/leitura

## 🧠 Fase 5 — Agente de IA
**~2–3 semanas**
- [ ] Integração Claude/GPT + RAG com base de conhecimento da empresa
- [ ] Bot de IA no fluxo com handoff pra humano

## 🔌 Fase 6 — Integrações, API e relatórios
**contínuo**
- [ ] Webhook de saída + API pública (Swagger via drf-spectacular)
- [ ] 1–2 integrações-chave do nicho
- [ ] Dashboard de relatórios (conversas, tempo de resposta, performance por atendente)

## 💳 Fase 7 — Billing e go-to-market
- [ ] Assinatura por atendente (Asaas / Pagar.me / Stripe)
- [ ] Trial de 7 dias
- [ ] Landing page

---

> [!warning] Regra de ouro
> Lançar após a **Fase 1** e validar com clientes reais antes de construir o chatbot visual. Fases 1 + 2 sozinhas já são **2–3 meses** de trabalho sério.

## 🎁 Ideias de diferencial (autoral)
- **IA nativa** desde o começo, não como add-on caro
- Foco num **vertical** com templates prontos (ex.: "o WhatsApp da sua clínica")
- **Preço/UX mais simples** pra micro e pequenas empresas
