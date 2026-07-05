---

## tags: [projeto, assistente-virtual, roadmap, ferias-2026] status: planejamento inicio: 2026-07-01 tipo: second-brain/projeto
---
# 🎙️ Roadmap — Assistente Virtual Pessoal ("AMEA AI")



Assistente de voz local, tipo Jarvis/Alexa, capaz de tocar música no Spotify, abrir/pesquisar no YouTube, marcar reuniões no Google Calendar e crescer com novas "skills" ao longo do tempo. Este documento é a fonte única de verdade do projeto — atualize o status das fases conforme avança.

---

## 1. Visão geral da arquitetura

O diagrama acima resume o pipeline. Em texto:

1. **Wake word** — detecta a palavra de ativação ("Ei, Amea") rodando sempre em background, offline.
2. **STT (Speech-to-Text)** — converte o áudio capturado após a wake word em texto.
3. **Orquestrador (LLM)** — recebe o texto, decide a intenção ("tocar música", "marcar reunião"...) e extrai os parâmetros (artista, horário, etc.) via _function calling / tool use_.
4. **Skill dispatcher** — roteia a intenção pra função Python certa.
5. **Skills** — módulos independentes que chamam APIs externas (Spotify, YouTube, Google Calendar, e outras que você quiser adicionar).
6. **TTS (Text-to-Speech)** — converte a resposta do Atlas de volta em voz.

Isso é exatamente o padrão de **skills plugáveis** (parecido com Alexa Skills) — cada integração nova é um arquivo novo, sem tocar no core.

---

## 2. Stack recomendada (aproveitando o que você já sabe)

|Camada|Ferramenta|Por quê|
|---|---|---|
|Linguagem principal|Python 3.11+|Você já domina, e é o que tem mais libs de voz/IA prontas|
|Orquestrador/backend|**Django** (via management command, não `runserver`)|Você já domina o ORM, admin e estrutura de apps — ver seção 3.4|
|API/dashboard opcional|Django REST Framework|Mesmo padrão que você já usa no CoffeeChain e no Painel de Governança|
|Wake word|[openWakeWord](https://github.com/dscripka/openWakeWord)|Open-source, gratuito, treina wake word customizado|
|STT|[faster-whisper](https://github.com/SYSTRAN/faster-whisper) (local) ou Groq Whisper API|Local = grátis e privado; Groq = grátis e mais rápido se tiver internet|
|TTS|[edge-tts](https://github.com/rany2/edge-tts)|Gratuito, sem chave de API, vozes PT-BR boas (ex: `pt-BR-AntonioNeural`, `pt-BR-FranciscaNeural`)|
|LLM (intent + function calling)|Ver seção 5 (estratégia de tokens)|—|
|Banco local (histórico, config, tokens)|SQLite via ORM do Django|Zero SQL cru, admin pronto pra inspecionar dados|
|Interface opcional|React + Vite consumindo a API DRF|Reaproveita seu stack de frontend|
|Automação/voz contínua|Loop Python dentro de um management command (`sounddevice` + `numpy`)|Roda fora do ciclo request/response do Django|

Assunções que estou fazendo (me avisa se for diferente):

- Você vai rodar isso no seu PC Windows, ligado, não num Raspberry Pi dedicado.
- Você não precisa de multi-usuário — é assistente pessoal, single-user.
- Prioridade é ter algo funcionando rápido nas férias, não uma arquitetura de produção.
- Você vai usar Django puro (não FastAPI) por já ter fluência nele — isso muda a forma de rodar o core (seção 3.4), mas não muda a lógica do pipeline.

---

## 3. Componentes detalhados

### 3.1 Wake word

- Lib: `openwakeword`
- Roda em loop separado (thread ou processo), sempre ouvindo o microfone com baixo custo de CPU.
- Ao detectar, dispara a gravação de um trecho de áudio (ex: 5 segundos ou até detectar silêncio) → passa pro STT.
- Alternativa mais simples pro MVP: **pular o wake word no início** e usar push-to-talk (apertar uma tecla de atalho global com `keyboard` ou `pynput`). Você adiciona wake word depois, como fase 5.

### 3.2 STT (voz → texto)

```python
from faster_whisper import WhisperModel

model = WhisperModel("small", device="cpu", compute_type="int8")
segments, info = model.transcribe("audio.wav", language="pt")
texto = " ".join([s.text for s in segments])
```

- Modelo `small` já é bom pra PT-BR e roda em CPU comum.
- Se quiser mais velocidade e tiver internet: Groq tem endpoint de Whisper gratuito (`whisper-large-v3` via `groq.com`), latência muito baixa.

### 3.3 Orquestrador (LLM com function calling)

Esse é o cérebro. Você define "ferramentas" (tools) que o LLM pode chamar, no mesmo padrão de tool use que você já viu funcionando aqui na Anthropic API:

```python
tools = [
    {
        "name": "tocar_musica_spotify",
        "description": "Toca uma música ou playlist no Spotify",
        "input_schema": {
            "type": "object",
            "properties": {
                "query": {"type": "string", "description": "nome da música/artista/playlist"}
            },
            "required": ["query"]
        }
    },
    {
        "name": "abrir_youtube",
        "description": "Abre o YouTube ou pesquisa um vídeo",
        "input_schema": {
            "type": "object",
            "properties": {"query": {"type": "string", "description": "termo de busca, opcional"}}
        }
    },
    {
        "name": "criar_evento_calendario",
        "description": "Cria um evento no Google Calendar",
        "input_schema": {
            "type": "object",
            "properties": {
                "titulo": {"type": "string"},
                "data_hora_inicio": {"type": "string", "description": "ISO 8601"},
                "duracao_minutos": {"type": "integer", "default": 60}
            },
            "required": ["titulo", "data_hora_inicio"]
        }
    }
]
```

O LLM recebe o texto transcrito + essas tools, e devolve qual função chamar com quais parâmetros já extraídos (ex: "reunião amanhã às 17h" → `data_hora_inicio: 2026-07-02T17:00:00`). Isso resolve o parsing de linguagem natural sem você escrever regex nenhuma.

### 3.4 Skills (plugins)

Estrutura de projeto Django sugerida — cada skill é um **app Django** de verdade, e o loop de voz roda como **management command** (não como view HTTP):

```
atlas/
├── manage.py
├── atlas/                          # settings do projeto
│   ├── settings.py
│   └── urls.py
├── core/                           # app: orquestrador + dispatcher + histórico
│   ├── models.py                   # Comando, Sessao, IntentCache
│   ├── orchestrator.py             # chama o LLM, decide a tool
│   ├── dispatcher.py               # mapeia tool_name -> skill.executar()
│   └── management/commands/
│       └── run_assistant.py        # <- loop principal roda aqui (não runserver)
├── voice/                          # app: STT, TTS, wake word
│   ├── stt.py
│   ├── tts.py
│   └── wake_word.py
├── skills/
│   ├── spotify_skill/               # app
│   │   ├── models.py                # SpotifyToken (refresh_token salvo com segurança)
│   │   ├── skill.py                 # def executar(params: dict) -> str
│   │   └── views.py                 # callback do OAuth (precisa de 1 view HTTP)
│   ├── youtube_skill/                # app
│   │   └── skill.py
│   └── calendar_skill/               # app
│       ├── models.py                # GoogleToken
│       ├── skill.py
│       └── views.py                  # callback do OAuth
└── dashboard/                       # app opcional: API DRF + admin
    ├── serializers.py
    └── views.py
```

Cada skill exporta uma função com a mesma assinatura (`def executar(params: dict) -> str`), registrada no `dispatcher.py` — isso facilita testar e adicionar novas isoladamente, exatamente como no padrão de apps do Django que você já usa.

**Por que isso funciona bem com Django:**

- O **ORM substitui o SQLite manual** — em vez de escrever SQL cru pro cache de intents, você cria um model `IntentCache` e usa queryset normal.
- **Django admin de graça** — em minutos você tem uma tela pra ver histórico de comandos, status das integrações e tokens salvos, sem escrever frontend nenhum.
- **Tokens do Spotify/Google como models** (`SpotifyToken`, `GoogleToken`) em vez de arquivo solto — mais organizado, e você guarda o `refresh_token` no banco em vez de texto puro no `.env`.
- O **fluxo OAuth do Spotify e do Google exige uma URL de callback** — por isso essas duas skills têm uma `views.py` com uma view HTTP simples, mesmo que o resto do projeto não use `runserver` no dia a dia. Você sobe o servidor só na hora de autenticar (`python manage.py runserver`), pega o token, salva no banco, e depois o `run_assistant` usa o token salvo sem precisar do servidor rodando.

**O loop principal** (`core/management/commands/run_assistant.py`) roda assim:

```python
from django.core.management.base import BaseCommand
from core.orchestrator import processar_comando
from voice.wake_word import aguardar_wake_word
from voice.stt import transcrever
from voice.tts import falar

class Command(BaseCommand):
    help = "Roda o loop principal do assistente Atlas"

    def handle(self, *args, **options):
        self.stdout.write("Atlas rodando. Aguardando wake word...")
        while True:
            aguardar_wake_word()
            texto = transcrever()
            resposta = processar_comando(texto)
            falar(resposta)
```

Você inicia o assistente com `python manage.py run_assistant` — um processo separado, contínuo, sem WSGI/ASGI no meio.

### 3.5 TTS (texto → voz)

```python
import edge_tts
import asyncio

async def falar(texto: str):
    communicate = edge_tts.Communicate(texto, voice="pt-BR-AntonioNeural")
    await communicate.save("resposta.mp3")
    # depois: tocar com playsound ou sounddevice
```

### 3.6 Fluxo do loop principal

```
loop (dentro do management command run_assistant):
  aguarda wake word (ou push-to-talk)
  grava áudio
  audio -> STT -> texto
  texto -> LLM (com tools) -> decisão
  decisão -> dispatcher -> skill.executar() -> resultado (salvo em Comando via ORM)
  resultado -> LLM (resposta em linguagem natural, opcional) -> TTS -> fala
```

---

## 4. Integrações — endpoints específicos

### 🎵 Spotify Web API

- Crie um app em https://developer.spotify.com/dashboard (gratuito).
- **Importante:** controlar playback via API exige **Spotify Premium** — sem premium você só consegue buscar dados, não dar play remoto. Se você não tem Premium, a alternativa é abrir o Spotify desktop/web e simular o play via automação local (ex: abrir `spotify:track:ID` como URI, que o app do Windows reconhece nativamente sem precisar de token nenhum).
- Fluxo OAuth (Authorization Code):
    1. `GET https://accounts.spotify.com/authorize` (com `client_id`, `scope=user-modify-playback-state`, `redirect_uri`)
    2. `POST https://accounts.spotify.com/api/token` → troca o `code` por `access_token` + `refresh_token`
- Endpoints principais:
    - Buscar música: `GET https://api.spotify.com/v1/search?q={query}&type=track`
    - Tocar: `PUT https://api.spotify.com/v1/me/player/play` — body `{"uris": ["spotify:track:{id}"]}`
    - Pausar: `PUT https://api.spotify.com/v1/me/player/pause`
    - Próxima faixa: `POST https://api.spotify.com/v1/me/player/next`
- Lib Python que já embrulha tudo isso: `spotipy` (recomendo, economiza muito boilerplate).

### 📺 YouTube

Duas abordagens, escolha pelo esforço vs. resultado:

- **Simples (sem API, sem token):** `webbrowser.open(f"https://www.youtube.com/results?search_query={query}")` — abre o navegador já com a busca pronta. Resolve 90% do seu caso de uso ("abrir o YouTube", "pesquisar tal vídeo").
- **Avançado (com API):** YouTube Data API v3, endpoint `GET https://www.googleapis.com/youtube/v3/search?part=snippet&q={query}&key={API_KEY}` — cota gratuita de 10.000 unidades/dia (cada busca custa ~100 unidades, então ~100 buscas/dia de graça). Só vale a pena se quiser abrir direto o primeiro resultado sem passar pela tela de busca.

### 📅 Google Calendar

- Crie um projeto no [Google Cloud Console](https://console.cloud.google.com/) (gratuito) e ative a Calendar API.
- OAuth2 com escopo `https://www.googleapis.com/auth/calendar.events`.
- Criar evento: `POST https://www.googleapis.com/calendar/v3/calendars/primary/events`

```json
{
  "summary": "Reunião",
  "start": {"dateTime": "2026-07-02T17:00:00-03:00"},
  "end": {"dateTime": "2026-07-02T18:00:00-03:00"}
}
```

- Lib Python: `google-api-python-client` + `google-auth-oauthlib`.

---

## 5. Gerenciando tokens/custo — a parte que você pediu

Você não precisa gastar nada pra ter um MVP funcional. A estratégia é **roteamento por complexidade**: usar modelo gratuito/local para o trabalho simples e só "gastar" em modelo pago quando realmente precisar.

|Opção|Custo|Quando usar|
|---|---|---|
|**Ollama** (local, ex: `llama3.2:3b`, `phi3`)|100% grátis, roda no seu PC|Classificação de intenção simples ("tocar música" vs "marcar reunião") — não precisa de internet nem de chave|
|**Groq** (`groq.com`)|Tier gratuito bem generoso, latência muito baixa|Ótimo pro dia a dia — hospeda Llama 3.1/3.3, Mixtral, e o STT Whisper também tem free tier lá|
|**OpenRouter** (`openrouter.ai`)|Tem modelos com tag `:free` (ex: variações de Llama, Gemini Flash)|Bom pra prototipar rápido testando vários modelos sem se comprometer com uma conta só|
|**Google AI Studio** (Gemini)|Tier gratuito com limite diário de requisições|Alternativa robusta, bom para parsing de linguagem natural em PT-BR|
|**Anthropic API (Claude)**|Pago por token, mas com function calling muito preciso|Reserve pra quando quiser qualidade máxima na extração de parâmetros (datas relativas complexas, ambiguidade)|

**Arquitetura de custo zero recomendada pro MVP:**

```
Intent simples (comandos diretos) → Ollama local (grátis, sem internet)
Intent ambíguo/complexo → Groq free tier (rápido, ainda grátis)
Fallback de qualidade (opcional, se quiser) → Claude/OpenRouter pago, só nos casos difíceis
```

Isso significa que, no dia a dia, você pode rodar o Atlas **sem gastar nenhum token pago**. Comece só com Ollama + Groq, e só pense em pagar algo se sentir que a precisão não é suficiente pra você.

Truque extra de economia: cacheie respostas de intents repetidos (ex: "toca minha playlist de trabalho" sempre mapeia pro mesmo `spotify:playlist:ID`) num SQLite local, e só chama o LLM quando o texto não bater com nada em cache.

---

## 6. Estrutura do vault Obsidian (second-brain do agente)

Aproveitando a estrutura de developer second-brain que você já está montando:

```
Projetos/Atlas/
├── 00-Roadmap.md              # este arquivo
├── 01-Decisoes-Arquitetura/   # ADRs — por que escolheu X e não Y
│   └── ADR-001-escolha-do-llm.md
├── 02-Skills/                 # 1 nota por skill, documentando endpoint, auth, exemplos
│   ├── spotify.md
│   ├── youtube.md
│   └── calendar.md
├── 03-Testes/
│   └── casos-de-teste.md
├── logs/                      # 1 nota por sessão de trabalho
│   └── 2026-07-01.md
└── templates/
    └── log-sessao.md
```

**Template de log de sessão** (`templates/log-sessao.md`):

```markdown
---
data: {{date}}
fase: 
tempo_investido: 
---

## O que fiz hoje
- 

## Decisões tomadas
- 

## Bloqueios / dúvidas
- 

## Próximos passos
- [ ] 

## Commits
- 
```

Dataview query útil pro seu vault, pra ver o progresso geral:

```dataview
TABLE fase, tempo_investido
FROM "Projetos/Atlas/logs"
SORT data DESC
```

---

## 7. Roadmap por fases

### Fase 0 — Setup do ambiente (1-2 dias)

- [x] Criar repositório Git (`ameaia`) ✅ 2026-07-02
- [x] Ambiente virtual Python (`venv` ou `poetry`) ✅ 2026-07-02
- [x] `django-admin startproject atlas` + criar os apps: `core`, `voice`, `skills/spotify_skill`, `skills/youtube_skill`, `skills/calendar_skill`, `dashboard` (opcional) ✅ 2026-07-02
- [x] Rodar `python manage.py migrate` (SQLite) e criar superuser pro admin ✅ 2026-07-02
- [x] Instalar dependências: `django`, `djangorestframework` (se for usar dashboard), `faster-whisper`, `edge-tts`, `openwakeword`, `sounddevice`, `python-dotenv` ou `django-environ` ✅ 2026-07-02
- [x] Configurar `.env` (nunca commitar) e `.env.example` ✅ 2026-07-04
- [ ] Configurar Ollama local, baixar `llama3.2:3b`
- [ ] Criar conta gratuita na Groq, gerar API key
- [ ] Criar a nota `00-Roadmap.md` no vault e o template de log de sessão

### Fase 1 — Núcleo de voz (STT + TTS), sem LLM ainda (2-3 dias)

- [ ] Script que grava áudio do microfone (push-to-talk com tecla)
- [ ] Integrar `faster-whisper`, transcrever e imprimir no console
- [ ] Integrar `edge-tts`, o Atlas responde falando um texto fixo
- [ ] Teste manual: falar "oi" → ver o texto transcrito certo → ouvir uma resposta falada

### Fase 2 — Orquestrador com LLM e primeira skill (3-4 dias)

- [ ] Definir o schema das 3 tools (Spotify, YouTube, Calendar) como na seção 3.3
- [ ] Integrar Ollama local pra classificar intenção simples
- [ ] Implementar a skill mais fácil primeiro: **YouTube** (sem OAuth, só `webbrowser.open`)
- [ ] Fluxo completo ponta a ponta: voz → texto → intenção → abre o YouTube

### Fase 3 — Spotify e Calendar (autenticação OAuth) (4-5 dias)

- [ ] Registrar app no Spotify Developer Dashboard
- [ ] Implementar fluxo OAuth do Spotify (guardar refresh_token localmente, criptografado ou fora do Git)
- [ ] Skill de tocar música (`spotipy`)
- [ ] Registrar projeto no Google Cloud, ativar Calendar API
- [ ] Implementar fluxo OAuth do Google
- [ ] Skill de criar evento — testar parsing de datas relativas ("amanhã às 17h", "sexta que vem")

### Fase 4 — Groq como fallback + roteamento inteligente (2 dias)

- [ ] Implementar lógica: se confiança do Ollama for baixa, escalar pra Groq
- [ ] Cache de intents repetidos em SQLite
- [ ] Medir tempo de resposta de cada camada (log de latência)

### Fase 5 — Wake word e loop contínuo (3 dias)

- [ ] Treinar/baixar modelo de wake word customizado com openWakeWord
- [ ] Trocar push-to-talk pelo wake word em loop contínuo
- [ ] Tratar falsos positivos (ruído ambiente ativando à toa)

### Fase 6 — Polimento e skills extras (opcional, tempo restante das férias)

- [ ] Dashboard React simples mostrando histórico de comandos e status das integrações
- [ ] Skills extras que fazem sentido pra você: clima, lembretes, WhatsApp (você já mexeu com Baileys), controle de janelas/apps
- [ ] Persona/voz customizada, ajuste de "personalidade" do Atlas nas respostas

---

## 8. Estratégia de testes

|Tipo|Ferramenta|O que testar|
|---|---|---|
|Unitário|`TestCase` do Django (ou `pytest-django`, se preferir)|Cada skill isolada — mockando a API externa (`unittest.mock.patch` ou `responses` pra simular respostas HTTP do Spotify/Google)|
|Modelos|`TestCase` do Django|`SpotifyToken`, `GoogleToken`, `IntentCache` — validações, expiração de token, etc.|
|Integração|`TestCase` + fixtures|Orquestrador → dispatcher → skill, com o LLM real ou mockado|
|Parsing de linguagem natural|Casos de teste manuais documentados no vault|Lista de frases de teste: "toca uma música do X", "marca uma reunião amanhã às 17", "abre o YouTube e procura Y" — valide se o LLM extrai os parâmetros certos|
|Voz (STT)|Script manual com gravações de exemplo|Grave 10-15 frases suas, rode o STT, confira a taxa de acerto|
|Latência|Log com `time.perf_counter()` em cada etapa|Meça wake word → resposta final, veja onde está o gargalo|
|End-to-end|Roteiro manual de cenários|"Diga X, espere Y aconteça" — documente isso como checklist reproduzível no vault|

Rode tudo com `python manage.py test` (padrão que você já usa no CoffeeChain e no Painel de Governança).

Exemplo de teste unitário de skill:

```python
from django.test import TestCase
from unittest.mock import patch
from skills.calendar_skill.skill import executar

class CalendarSkillTest(TestCase):
    @patch("skills.calendar_skill.skill.requests.post")
    def test_criar_evento_calendario(self, mock_post):
        mock_post.return_value.status_code = 200
        resultado = executar({"titulo": "Reunião", "data_hora_inicio": "2026-07-02T17:00:00"})
        self.assertIn("Reunião", resultado)
        mock_post.assert_called_once()
```

Casos de teste de linguagem natural pra documentar no vault (`03-Testes/casos-de-teste.md`):

- [ ] "toca Legião Urbana no Spotify" → chama `tocar_musica_spotify` com query correta
- [ ] "marca uma reunião amanhã às 17" → `data_hora_inicio` = amanhã 17:00 no fuso certo
- [ ] "abre o YouTube" → abre sem query de busca
- [ ] "pesquisa receita de bolo no YouTube" → abre com busca
- [ ] frase ambígua tipo "toca aquela música" → Atlas pede esclarecimento em vez de alucinar

---

## 9. Git, commits e documentação

**Convenção de commits** (Conventional Commits, você provavelmente já usa algo parecido):

```
feat: adiciona skill de tocar música no Spotify
fix: corrige parsing de data relativa "amanhã"
docs: documenta fluxo OAuth do Google Calendar no vault
test: adiciona testes unitários da skill de calendário
chore: configura .env.example e gitignore
refactor: extrai lógica de roteamento de intent pro orchestrator
```

**Estratégia de branches:**

- `main` — sempre estável
- `feature/spotify-skill`, `feature/wake-word`, etc. — uma branch por fase/skill
- Merge via PR mesmo sendo projeto solo, é bom hábito e você já deve ter isso automatizado no seu fluxo com Claude Code

**Checklist de fim de sessão** (todo dia que trabalhar nisso):

- [ ] Commit com mensagem seguindo a convenção
- [ ] Atualizar o log de sessão no vault (`logs/YYYY-MM-DD.md`)
- [ ] Marcar os itens concluídos nesta roadmap
- [ ] Se tomou alguma decisão de arquitetura relevante, criar um ADR em `01-Decisoes-Arquitetura/`

**README do repositório** deve conter: como rodar localmente, variáveis de ambiente necessárias, como adicionar uma skill nova (guia rápido de 5 passos), e um link pra este roadmap no vault.

---

## 10. Checklist de MVP (Definition of Done)

O Atlas está "pronto" (v1) quando:

- [ ] Você consegue falar um comando e ele toca uma música específica no Spotify
- [ ] Você consegue pedir pra abrir/pesquisar algo no YouTube por voz
- [ ] Você consegue marcar uma reunião no Google Calendar dizendo dia e hora por voz
- [ ] O fluxo funciona sem gastar nenhum token pago (Ollama + Groq free tier)
- [ ] Existe pelo menos 1 teste automatizado por skill
- [ ] O vault documenta as 3 integrações e tem pelo menos 5 logs de sessão
- [ ] README permite que você (ou outra pessoa) rode o projeto do zero seguindo o passo a passo

---

## 11. Recursos

- Spotify Web API docs: https://developer.spotify.com/documentation/web-api
- YouTube Data API: https://developers.google.com/youtube/v3
- Google Calendar API: https://developers.google.com/calendar/api/guides/overview
- openWakeWord: https://github.com/dscripka/openWakeWord
- faster-whisper: https://github.com/SYSTRAN/faster-whisper
- edge-tts: https://github.com/rany2/edge-tts
- Groq (free tier): https://groq.com
- OpenRouter (modelos free): https://openrouter.ai/models
- Ollama: https://ollama.com