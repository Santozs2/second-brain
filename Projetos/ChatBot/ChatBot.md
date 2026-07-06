---
type: project
area: Projetos
status: 🟢 Em Desenvolvimento
priority: 🔥 Alta
tecnologia: Django
tags:
  - project
  - saas
  - whatsapp
  - chatbot
created: 2026-07-06
updated: 2026-07-06
---
# 💬 ChatBot — Plataforma de Atendimento WhatsApp

> [!abstract] Visão geral
> SaaS de atendimento e automação via **WhatsApp Cloud API (oficial)**, com inbox multi-atendente, construtor de chatbot visual, CRM e agente de IA. Codinome: `zapinbox`.

## 🗺️ Páginas do projeto

- [[roadmap-plataforma-whatsapp|🚀 Roadmap da plataforma]] — fases 0 a 7, do MVP ao billing
- [[stack-e-estrutura-repo|🔧 Stack e estrutura do repositório]] — decisões técnicas e organização do código
- [[multi-tenancy-django|🏢 Multi-tenancy no Django]] — isolamento de dados por organização
- [[guia-fase-1-inbox|🏗️ Guia da Fase 1: Inbox multi-atendente]] — sprints 1 a 4
- [[retrospectiva-sprint-1|📋 Retrospectiva do Sprint 1]] — fundação e multi-tenancy concluídos

## 🎯 Estado atual

> [!success] Sprint 1 concluído
> Fundação de pé: multi-tenancy isolando dados por organização (testado), auth JWT, models e migrations no Supabase. Próximo: Sprint 2 (integração Cloud API).

## 🧱 Stack

- **Backend:** [[Django|Django]] + [[Django REST Framework|DRF]] + Celery + [[Cache|Redis]] + PostgreSQL
- **Frontend:** [[React|React]] + [[TypeScript|TypeScript]] + Tailwind
- **Tempo real:** Django Channels ([[cs-websocket|WebSocket]])
- **Infra:** [[Docker|Docker]] · [[Git|Git]] · [[CI-CD|CI/CD]]

## 🔗 Relacionado no Vault

- **Backend:** [[Django|Django]] · [[Django REST Framework|DRF]] · [[Python|Python]] · [[REST API|REST API]]
- **Frontend:** [[React|React]] · [[TypeScript|TypeScript]]
- **Conceitos:** [[cs-websocket|WebSockets]] · [[JWT|JWT]] · [[cs-authentication|Autenticação]] · [[Cache|Cache/Redis]] · [[ORM|ORM]] · [[Models|Models]] · [[Migrations|Migrations]]
- **Infra:** [[Docker|Docker]] · [[Git|Git]] · [[CI-CD|CI/CD]]

## Veja também

- [[Projetos|🚀 Projetos]]
- [[Home|Painel Principal]]
