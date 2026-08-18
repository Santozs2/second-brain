# Infraestrutura de Testes — Fase 2 S5

**Status:** ✅ COMPLETO
**Data:** 2026-07-20 · **Atualizado:** 2026-07-21 (reorg `e2e/` + Selenium removido)
**Commits:** `65ef08f` (infra original) · `9cfd329` (reorg em `e2e/` + remove selenium) · `aa4f2a1` (credenciais reais)

## Visão Geral

3 formas de validar o editor de fluxos **sem abrir o navegador manualmente**:

| Teste | Tempo | Tipo | Navegador | Recomendado |
|-------|-------|------|-----------|-------------|
| API | 30s | Automático | ❌ | ✅ Rápido |
| Pytest | 2min | Unittest | ❌ | ✅ CI/CD |
| Playwright | 2-3min | Visual | ✅ | ⭐ Moderno |

> **Selenium removido** (`test_flows_e2e.py`): era redundante com o Playwright, que já cobre o mesmo fluxo visual de forma mais rápida e moderna. Um só teste E2E visual basta.

## Arquivos (reorganizados em `e2e/`)

Todos os testes saíram da raiz e foram pra pasta `e2e/` — a raiz do repo ficou limpa (só `backend/`, `frontend/`, `e2e/`, `Makefile`, `README.md`, `TESTING.md`).

```
chatbot/
├── e2e/
│   ├── run_tests.sh                ← Runner automático (bash)
│   ├── test_flows_api.sh           ← Teste API puro
│   ├── test_flows_e2e.js           ← Teste Playwright (Node)
│   ├── test_flows_pytest.py        ← Testes Pytest
│   ├── conftest.py                 ← Fixtures pytest
│   ├── package.json / node_modules ← deps do Playwright
├── Makefile                        ← Comandos make (aponta p/ e2e/)
└── TESTING.md                      ← Documentação
```

> **Credenciais reais** nos testes (commit `aa4f2a1`): `teste@zz.com` / `teste1234` / org `zz_teste` (antes eram placeholders `test@test.com`). O header `X-Organization` é inofensivo se errado (o middleware cai na 1ª membership), mas o login precisa bater.

## 1. Teste API (Recomendado)

**Comando:**
```bash
bash e2e/run_tests.sh api
```

**Características:**
- ⚡ Mais rápido (30 segundos)
- 🎯 Determinístico (sem UI randomness)
- ✅ Perfeito pra CI/CD
- 📦 Nenhuma dependência extra (só curl)

**O que testa:**
1. Login
2. Listar canais
3. Criar fluxo (1 start node)
4. Listar fluxos
5. Recuperar fluxo por ID
6. Atualizar fluxo (3 nós + 2 edges)
7. Publicar (activate)
8. Despublicar (deactivate)
9. Deletar (delete)

**Script:** `e2e/test_flows_api.sh` (~150 linhas)

**Implementação:**
- Usa curl pra fazer requests HTTP
- Grep pra parsear JSON
- Testes sequenciais em bash puro
- Feedback visual colorido

**Saída esperada:**
```
==================================================
✅ TESTE COMPLETO COM SUCESSO!
==================================================
```

## 2. Teste Pytest (Profissional)

**Comando:**
```bash
pytest e2e/test_flows_pytest.py -v
```

**Características:**
- 🏛️ Framework padrão Python
- 🔧 Fixtures reutilizáveis
- 📊 Relatórios HTML
- 🏷️ Markers (@pytest.mark.api)

**Dependências:**
```bash
pip install pytest requests
```

**Classes de Teste:**

### `TestFlowsCRUD`
- `test_create_flow()` — POST /api/flows/
- `test_list_flows()` — GET /api/flows/
- `test_flow_crud_cycle()` — Ciclo completo (create→get→update→publish→delete)

### `TestFlowsValidation`
- `test_condition_node_structure()` — Valida condition com 2 handles true/false
- `test_requires_start_node()` — Testa fluxo sem start node

**Fixtures** (em `e2e/conftest.py`):
- `api_client` — Client HTTP com autenticação
- `channel_id` — ID do canal (reutilizável)

**Saída esperada:**
```
test_flows_pytest.py::TestFlowsCRUD::test_create_flow PASSED
test_flows_pytest.py::TestFlowsCRUD::test_list_flows PASSED
test_flows_pytest.py::TestFlowsCRUD::test_flow_crud_cycle PASSED
test_flows_pytest.py::TestFlowsValidation::test_condition_node_structure PASSED
======================== 5 passed in 1.23s ========================
```

## 3. Teste Playwright (Visual Moderno)

**Comando:**
```bash
bash e2e/run_tests.sh playwright
```

**Características:**
- 🖥️ Abre navegador real
- 🖱️ Simula cliques, digitação, navegação
- 📸 Screenshots automáticas
- ⚡ Headless (rápido)

**Dependências:**
```bash
cd e2e && npm install
```

**O que testa:**
1. Login na UI
2. Navegar pra aba Fluxos
3. Criar fluxo (modal)
4. Abrir editor (canvas)
5. Adicionar nós (send_message, end)
6. Editar nó (textarea)
7. Salvar fluxo (PUT)
8. Publicar fluxo (POST activate)

**Script:** `e2e/test_flows_e2e.js` (~200 linhas)

**Implementação:**
- Usa Playwright API (page.click, page.fill, etc)
- Waits explícitos (waitForSelector)
- Dialog handling (alerts automáticas)
- Screenshot em sucesso/erro (`test_*.png`, ignorados pelo git)

**Saída esperada:**
```
1. Fazendo login...
   ✓ Login enviado

2. Navegando para Fluxos...
   ✓ Abriu aba Fluxos

...

✅ TESTE COMPLETO COM SUCESSO!
📸 Screenshot salvo: test_result.png
```

---

## Runner Automático

**Script:** `e2e/run_tests.sh` (faz `cd` pra própria pasta no início)

**Funcionalidades:**
- ✅ Verifica pré-requisitos (backends rodando)
- ✅ Detecta qual teste rodar (api/playwright/all)
- ✅ Mostra cores ANSI (verde/vermelho/amarelo)
- ✅ Relatório consolidado no final
- ✅ Exit code correto (0 = sucesso)

**Uso:**
```bash
bash e2e/run_tests.sh              # Roda todos (padrão)
bash e2e/run_tests.sh api          # Só API
bash e2e/run_tests.sh playwright   # Só Playwright
bash e2e/run_tests.sh -h           # Ajuda
```

---

## Alternative: Make

**Makefile** com comandos (apontam pra `e2e/`):

```bash
make help              # Lista todos os comandos
make backend           # Inicia Django
make frontend          # Inicia Vite
make check-ports       # Verifica disponibilidade
make test              # Roda teste API
make test-api          # Alias
make test-e2e-pw       # Teste Playwright
make test-all          # Roda todos
make setup             # Instala dependências (cd e2e && npm install)
```

---

## CI/CD Integration

O CI real do projeto **não** roda esses E2E (exigem backend + frontend + banco ao vivo, inviável no runner). Em vez disso roda checagens: `manage.py check` + `makemigrations --check` + `migrate` no backend e `lint` + `build` no frontend. Detalhes em [[ci-github-actions]].

Quando existir uma suíte pytest **in-process** (sem depender de servidor rodando), ela entra direto no job `backend` do CI.

---

## Troubleshooting

| Problema | Solução |
|----------|---------|
| "Port 8000 Closed" | `cd backend && python manage.py runserver` |
| "Port 5173 Closed" | `cd frontend && npm run dev` |
| "No such file" | Rode da raiz: `bash e2e/run_tests.sh api` |
| Playwright não found | `cd e2e && npm install` |
| Browser crashed | Feche navegadores abertos, tente rodar API |

---

## Checklist de Validação

Antes de considerar **pronto**:

- [ ] `bash e2e/run_tests.sh api` — ✅ PASS
- [ ] `pytest e2e/test_flows_pytest.py -v` — ✅ PASS
- [ ] `make check-ports` — ✅ Ambos abertos
- [ ] Screenshots salvos
- [ ] Zero warnings no console

---

## Próximas Melhorias

1. **Coverage:** Adicionar pytest-cov pra ver % de cobertura
2. **Suíte pytest in-process:** testes que não dependem de servidor ao vivo → entram no CI
3. **Load Testing:** Script com k6 ou locust pra testar performance
4. **Mobile:** Testes Playwright com viewport mobile
5. **Retry:** Adicionar retry automático em E2E flaky

---

**Conclusão:** Infraestrutura completa e organizada em `e2e/`. Escolha o teste conforme o contexto:
- **Dev local:** Teste API (rápido feedback)
- **CI/CD:** Pytest (determinístico)
- **QA:** Playwright (visual)

Links: [[fase2-s5-editor-react-flow.md]] (implementação) · [[ci-github-actions]] (pipeline)
