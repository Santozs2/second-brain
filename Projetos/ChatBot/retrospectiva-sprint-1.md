---
title: "Retrospectiva — Sprint 1: Fundação e Multi-Tenancy"
aliases: ["Retrospectiva — Sprint 1: Fundação e Multi-Tenancy"]
tags: [django, sprint-1, retrospectiva, multi-tenancy, supabase, chatbot]
status: concluido
projeto: ChatBot
criado: 2026-07-06
---

> [!info] Projeto: [[ChatBot|💬 ChatBot]] · Tecnologias: [[Django|Django]] · [[Migrations|Migrations]] · [[Models|Models]]

# 📋 Retrospectiva — Sprint 1: Fundação

> [!success] Status: CONCLUÍDO
> Fundação de pé: multi-tenancy isolando dados por organização (testado e aprovado), auth, models no ar, migrations aplicadas no Supabase e admin funcionando. **Base sólida pra sustentar todo o resto do produto.**

## 🎯 Objetivo da Sprint
Projeto Django estruturado, usuário logando, e multi-tenancy isolando dados por organização — a fundação sobre a qual o inbox e o resto do sistema serão construídos.

---

## ✅ O que foi entregue

- Projeto Django estruturado em apps (`accounts`, `organizations`, `whatsapp`, `contacts`, `inbox`, `common`)
- `TenantModel` base (UUID + timestamps + FK `organization`)
- User customizado com login por email
- Models `Organization` e `Membership` (RBAC: owner/admin/agent)
- Multi-tenancy shared database / shared schema (isolamento por linha)
- Banco no Supabase (Postgres gerenciado) com todas as tabelas criadas
- Admin do Django funcionando
- **Isolamento multi-tenant testado e validado** (org A não vê dados da org B)

---

## 🧭 Decisões técnicas tomadas

| Decisão | Escolha | Motivo |
|---------|---------|--------|
| Banco | Supabase (Postgres) | Gerenciado, painel visual, já conhecido de projetos anteriores |
| Conexão com Supabase | Postgres nativo (não URL+Key) | Django ORM precisa de conexão SQL; URL+Key é a API REST, incompatível com o ORM |
| Multi-tenancy | Shared DB, shared schema | Mais simples e adequado pro MVP; isolamento por filtro no código |
| Auth | Custom User (email) + JWT | Padrão pra SaaS; email como identificador |
| RLS do Supabase | Deixado ligado (irrelevante) | Django conecta como owner e bypassa RLS; segurança real está no código |

---

## 🔧 Dificuldades encontradas

> [!note] Por que documentar isso
> A maioria desses erros é de setup — chatos justamente por virem antes da parte divertida. Registrados aqui pra resolver em minutos se aparecerem de novo em outro projeto.

### 1. `env is not defined` no settings.py
- **Sintoma:** `NameError` / "env is not defined" no bloco `DATABASES`, que usava `env("POSTGRES_DB")`.
- **Causa:** a função `env` era usada mas nunca tinha sido definida nem importada.
- **Solução:** instalar `django-environ` e inicializar no topo do `settings.py`:
  ```python
  import environ, os
  from pathlib import Path
  BASE_DIR = Path(__file__).resolve().parent.parent
  env = environ.Env()
  environ.Env.read_env(os.path.join(BASE_DIR, ".env"))
  ```

### 2. Typo no import: `eviron`
- **Sintoma:** "Import 'eviron' could not be resolved".
- **Causa:** faltou o **n** — o certo é `environ` (e-n-v-i-r-o-n).
- **Detalhe que confunde:** o pacote instala como `django-environ`, mas o import é só `environ` (sem o prefixo `django-`).

### 3. Ordem das linhas trocada
- **Sintoma:** `BASE_DIR` sendo usado antes de existir.
- **Causa:** a linha `read_env(... BASE_DIR ...)` estava **acima** da definição de `BASE_DIR`. Python lê de cima pra baixo.
- **Solução:** definir `BASE_DIR` primeiro, depois as linhas do `environ`.

### 4. Falha de conexão com o Supabase (autenticação + IPv6)
- **Sintoma:** `password authentication failed for user "postgres.postgres"` e conexão caindo num endereço IPv6 (`2600:1f1e:...`).
- **Causas (duas ao mesmo tempo):**
  1. Usuário errado no `.env` (`postgres.postgres` em vez de `postgres.<id-do-projeto>`) e senha incorreta.
  2. Supabase resolvendo o host por IPv6, que a rede residencial não roteava.
- **Solução:**
  - Corrigir o usuário completo e **resetar a senha do banco** (que é diferente da senha da conta Supabase).
  - Usar o **Session pooler** do Supabase (host `.pooler.supabase.com`, IPv4) em vez da conexão direta.
  - Adicionar `"OPTIONS": {"sslmode": "require"}` (Supabase exige SSL).

### 5. Migrations não geradas para `contacts`, `whatsapp` e `inbox`
- **Sintoma:** `showmigrations` mostrava esses apps como `(no migrations)`; as tabelas não existiam no Supabase. `makemigrations` dizia "No changes detected".
- **Causa raiz:** os arquivos `models.py` desses três apps **estavam vazios** — os models nunca tinham sido escritos de fato (só existiam no papel/documentos).
- **Solução:** escrever os models (`Contact`, `Tag`, `WhatsAppChannel`, `Conversation`, `Message`), rodar `makemigrations` + `migrate`.
- **Observação:** o app `common` aparecer como `(no migrations)` é **correto** — ele só tem models abstratos (`TenantModel`), que não viram tabela.

### 6. Selo "UNRESTRICTED" nas tabelas do Supabase
- **Sintoma:** todas as tabelas apareciam marcadas como UNRESTRICTED no painel.
- **Causa:** tabelas criadas pelo Django não recebem as policies de RLS do Supabase automaticamente.
- **Resolução:** **não é erro** — é cosmético. No projeto, a segurança é feita pelo Django (filtro por `request.tenant`), não pelo RLS. Pode ignorar.

### 7. `UniqueViolation` no slug da Organization
- **Sintoma:** `duplicate key value violates unique constraint "organizations_organization_slug_key"` com `Key (slug)=()` — slug **vazio**.
- **Causa:** o model `Organization` estava **sem o método `save()`** que gera o slug a partir do nome. Duas orgs ficaram com slug `""`, violando a constraint `unique`.
- **Solução:** adicionar o `save()` com `slugify(self.name)` + laço de unicidade. Não precisa de migration (é lógica Python, não muda schema). Também foi preciso apagar a org já criada com slug vazio antes de retestar.

---

## 🔁 Padrão recorrente identificado

> [!warning] Lição principal da Sprint
> Vários erros vieram do **mesmo padrão**: os arquivos `models.py` saíram mais "enxutos" do que o projetado nos documentos — faltaram models inteiros (dificuldade 5) e depois o método `save()` (dificuldade 7).
>
> **Aprendizado:** ao montar os arquivos a partir de um design, conferir peça por peça contra o documento de referência antes de rodar migrations. Achar buraco no setup custa minutos; achar no meio de uma feature custa horas.

---

## 🧪 Teste de isolamento (o que fechou a Sprint)

Validação feita no shell: duas organizações, um contato em cada, confirmando que o filtro por organização retorna só os dados corretos.

```python
from organizations.models import Organization
from contacts.models import Contact

org_a = Organization.objects.create(name="Imobiliária A")
org_b = Organization.objects.create(name="Imobiliária B")
Contact.objects.create(organization=org_a, wa_id="5511111111111", name="Cliente da A")
Contact.objects.create(organization=org_b, wa_id="5522222222222", name="Cliente da B")

print(list(Contact.objects.filter(organization=org_a).values_list("name", flat=True)))
# → ['Cliente da A']  ✅ (não vaza o contato da B)
```

**Resultado:** cada org enxerga apenas seus próprios dados. Isolamento aprovado.

---

## 📌 Estado final

- [x] Projeto Django estruturado
- [x] Custom User (email) + auth
- [x] Organization + Membership (RBAC)
- [x] Multi-tenancy (shared DB, shared schema)
- [x] Todas as tabelas criadas no Supabase
- [x] Admin funcionando
- [x] Isolamento multi-tenant testado

---

## 🚀 Próximos passos — Sprint 2 (Cloud API)

1. Client da Cloud API (enviar mensagem de texto via POST na Meta)
2. Webhook de verificação (GET → devolve `hub.challenge`)
3. Webhook de eventos (POST → recebe mensagens e status)
4. Processar inbound (JSON da Meta → Contact + Conversation + Message)
5. `ngrok` pra expor o localhost em HTTPS pra Meta

**Pré-requisito:** número conectado na Meta (número de teste do painel já serve pra começar).

> [!note] Pendência antes do Sprint 2
> Conferir `common/models.py` (TenantModel) e `organizations/models.py` (Membership) contra o design, já que o webhook e o RBAC dependem deles.

---

## 📚 Referências

- [[Guia de Implementação — Fase 1: Inbox Multi-Atendente]]
- [[Multi-Tenancy no Django — Implementação Shared DB, Shared Schema]]
- [[Stack Final e Estrutura de Repositório — ChatBot WhatsApp]]
- [[Roadmap — Plataforma de Atendimento WhatsApp]]
