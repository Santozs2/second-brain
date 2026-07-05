---
type: note
id: note-00roadmap-001
created: 2026-07-04
updated: 2026-07-04
tags:
  - type/note
status: draft
ai_generated: false
last_reviewed: 2026-07-04
---

# 00-Roadmap

[Conteúdo da nota]

---
tags: [projeto, assistente-virtual, roadmap, ferias-2026]
status: em-andamento
inicio: 2026-07-01
tipo: second-brain/projeto
---
# 🎙️ Roadmap — Assistente Virtual Pessoal ("Atlas")

> Nota original completa (arquitetura, integrações, testes etc.): [[Roadmap AMEA IA]]

Este arquivo é o índice de progresso vivo do projeto Atlas. Marque os itens conforme avança.

---

## Fase 0 — Setup do ambiente (1-2 dias)

- [x] Criar repositório Git ✅ 2026-07-02
- [x] Ambiente virtual Python (`.venv`) ✅ 2026-07-02
- [x] `django-admin startproject atlas` + apps: `core`, `voice`, `skills/spotify_skill`, `skills/youtube_skill`, `skills/calendar_skill`, `dashboard` ✅ 2026-07-02
- [x] Rodar `python manage.py migrate` (SQLite) ✅ 2026-07-02 — superuser fica pendente, criar manualmente com `python manage.py createsuperuser`
- [x] Instalar dependências (`django`, `djangorestframework`, `faster-whisper`, `edge-tts`, `openwakeword`, `sounddevice`, `django-environ`) ✅ 2026-07-02
- [x] Configurar `.env` / `.env.example` (via `django-environ`) ✅ 2026-07-02
- [ ] Configurar Ollama local, baixar `llama3.2:3b`
- [ ] Criar conta gratuita na Groq, gerar API key
- [x] Criar a nota `00-Roadmap.md` no vault e o template de log de sessão ✅ 2026-07-02

## Fase 1 — Núcleo de voz (STT + TTS), sem LLM ainda (2-3 dias)

- [ ] Script que grava áudio do microfone (push-to-talk com tecla)
- [ ] Integrar `faster-whisper`, transcrever e imprimir no console
- [ ] Integrar `edge-tts`, o Atlas responde falando um texto fixo
- [ ] Teste manual: falar "oi" → ver o texto transcrito certo → ouvir uma resposta falada

## Fase 2 — Orquestrador com LLM e primeira skill (3-4 dias)

- [ ] Definir o schema das 3 tools (Spotify, YouTube, Calendar)
- [ ] Integrar Ollama local pra classificar intenção simples
- [ ] Implementar a skill mais fácil primeiro: **YouTube** (sem OAuth)
- [ ] Fluxo completo ponta a ponta: voz → texto → intenção → abre o YouTube

## Fase 3 — Spotify e Calendar (autenticação OAuth) (4-5 dias)

- [ ] Registrar app no Spotify Developer Dashboard
- [ ] Implementar fluxo OAuth do Spotify
- [ ] Skill de tocar música (`spotipy`)
- [ ] Registrar projeto no Google Cloud, ativar Calendar API
- [ ] Implementar fluxo OAuth do Google
- [ ] Skill de criar evento

## Fase 4 — Groq como fallback + roteamento inteligente (2 dias)

- [ ] Escalar pra Groq quando confiança do Ollama for baixa
- [ ] Cache de intents repetidos em SQLite
- [ ] Medir latência de cada camada

## Fase 5 — Wake word e loop contínuo (3 dias)

- [ ] Treinar/baixar modelo de wake word customizado
- [ ] Trocar push-to-talk pelo wake word em loop contínuo
- [ ] Tratar falsos positivos

## Fase 6 — Polimento e skills extras (opcional)

- [ ] Dashboard React simples
- [ ] Skills extras (clima, lembretes, WhatsApp, controle de janelas/apps)
- [ ] Persona/voz customizada

---

## Dataview — progresso das sessões

```dataview
TABLE fase, tempo_investido
FROM "Projetos/Atlas/logs"
SORT data DESC
```

