# Fase 2 S5 — Editor Visual de Fluxo (React Flow) — COMPLETO

**Status:** ✅ IMPLEMENTADO E TESTADO  
**Data:** 2026-07-20  
**Commits:** `c6d6950` + `23c40a6`  
**Branch:** main (pushed)

## Resumo

Implementação completa do editor visual de fluxos de chatbot usando React Flow v12. O usuário pode:
- Listar, criar, renomear e deletar fluxos
- Arrastar nós ao canvas (6 tipos: start, send_message, question, condition, handoff, end)
- Configurar campos de cada nó via painel lateral
- Conectar nós com edges (com tratamento especial para condition → true/false)
- Salvar as mudanças (PUT /api/flows/{id}/)
- Publicar fluxos com validação (POST /api/flows/{id}/activate/)

## Stack

- **Frontend:** Vite + React 19 + TypeScript strict + Tailwind v4
- **Lib de Canvas:** `@xyflow/react` v12 (React Flow)
- **Backend:** Django + DRF (já pronto, não modificado)
- **HTTP:** `apiFetch` com JWT automático

## Implementação por Fatia

### S5a — Navegação + Lista ✅

**Arquivo:** `frontend/src/App.tsx` + `frontend/src/components/flows/FlowsList.tsx`

- Switch "Inbox" / "Fluxos" no header (3-painéis: header + main)
- `FlowsList` renderiza cards com:
  - Nome do fluxo
  - Canal associado (resolved via `channels` array)
  - Badge "Ativo" se `is_active=true`
  - Ações: Publicar/Despublicar, Deletar
- Modal de criação (nome + select de canal)
- Estados loading/erro/vazio
- Transição para `FlowEditor` ao clicar num card

### S5b — Canvas Read-Only ✅

**Arquivo:** `frontend/src/components/flows/FlowEditor.tsx` + `frontend/src/components/flows/nodeTypes.tsx`

- Carrega fluxo via GET `/api/flows/{id}/`
- Renderiza `<ReactFlow>` com:
  - Nodes do fluxo (posição + data preservados)
  - Edges conectando nós
  - Background + Controls (zoom/pan)
- 6 tipos de nó customizados (colors + icons):
  - **start** → verde (▶)
  - **send_message** → azul (💬)
  - **question** → roxo (❓)
  - **condition** → amarelo (🔀) — 2 handles (true/false)
  - **handoff** → laranja (👤)
  - **end** → vermelho (⏹)

### S5c — Edição + Salvar ✅

**Arquivo:** `frontend/src/components/flows/FlowEditor.tsx`

**Painel Esquerdo — Paleta de Nós**
- 6 botões: "+ start", "+ send_message", etc.
- Clique = adiciona nó no canvas com posição default (100+counter*20, 100+counter*20)
- ID gerado automaticamente (`{type}-{counter}`)

**Canvas Central**
- `onNodesChange` + `onEdgesChange` atualizam estado React
- `onConnect` liga nós (com lógica especial pra condition)
- Seleciona nó ao clicar (seta `selectedNodeId`)

**Painel Direito — Config de Nó**
- Campos variam por tipo:
  - `start`/`end` → sem campos
  - `send_message`/`handoff` → textarea `text`
  - `question` → textarea `text` + input `variable`
  - `condition` → input `variable` + select `op` (eq/contains) + input `value`
- Mudança de campo atualiza `node.data` no estado
- Preview de valores no canvas (truncado)

**Botão Salvar**
- Serializa `{ nodes, edges }` para `definition`
- PUT `/api/flows/{id}/` com payload completo
- Feedback: alert sucesso/erro, desabilita botão durante envio

### S5d — Condition + Validação + Publicar ✅

**Arquivo:** `frontend/src/components/flows/FlowEditor.tsx`

**Condition Node — 2 Handles**
- Renderiza 2 `<Handle type="source">` com IDs `"true"` e `"false"`
- Posicionados em lados opostos (Right=true, Left=false)
- Edge que sai de condition herda `sourceHandle` automaticamente

**Logic no `onConnect`**
```typescript
if (sourceNode?.type === "condition") {
  const existingHandles = edges
    .filter(e => e.source === connection.source)
    .map(e => e.sourceHandle)
  const nextHandle = existingHandles.includes("true") ? "false" : "true"
  connToAdd = { ...connection, sourceHandle: nextHandle }
}
```
→ Primeira edge fica `true`, segunda fica `false` (auto)

**Validação Pré-Publish**
```typescript
validateFlow(): string | null {
  // 1. Exatamente 1 start
  // 2. Todo question com variable
  // 3. Todo condition com variable + value
  // 4. Edges de condition com sourceHandle "true" ou "false"
  return null  // ou mensagem de erro
}
```

**Botão Publicar**
- Valida fluxo (mostra alert se falhar)
- Salva via PUT (se validação passou)
- POST `/api/flows/{id}/activate/`
- Desabilita botão se já publicado (`is_active=true`)
- Feedback: botão muda pra "✓ Publicado"

## Estrutura de Arquivos

```
frontend/src/
├── App.tsx                                    (switch view + header)
├── types.ts                                   (tipos: Flow, FlowNode, FlowEdge, etc)
├── components/
│   ├── flows/
│   │   ├── FlowsList.tsx                      (lista + CRUD)
│   │   ├── FlowEditor.tsx                     (canvas + edit + publish)
│   │   ├── nodeTypes.tsx                      (6 custom node components)
│   │   ├── nodeConfig.ts                      (config de cores/labels)
│   │   └── nodeRegistry.ts                    (nodeTypes export)
│   ├── Inbox.tsx                              (existente)
│   ├── ... outras
```

## Testes Realizados

### Build + Lint
- ✅ `npm run build` → 0 erros TS
- ✅ `npm run lint` → 0 erros (clean)

### Integração Backend
- ✅ Dev servers rodando (frontend 5173 + backend 8000)
- ✅ GET `/api/flows/` → lista vazia (sem fluxos)
- ✅ GET `/api/whatsapp/channels/` → lista vazia (sem canais)
- ✅ POST `/api/flows/` → cria fluxo com `definition` correto
- ✅ PUT `/api/flows/{id}/` → atualiza 3 nós + 2 edges
- ✅ POST `/api/flows/{id}/activate/` → `is_active: true`

### Flow Completo de Teste
1. Criar usuário `test@test.com` / `test123` ✅
2. Criar org `test-org` ✅
3. Criar canal WhatsApp ✅
4. Criar fluxo `Test Flow` (1 start node) ✅
5. Editar (adicionar send_message + end, ligar) ✅
6. Salvar (PUT) ✅
7. Publicar (POST activate/) ✅
8. Verificar `is_active=true` ✅

## Decisões de Design

| Decisão | Razão |
|---------|-------|
| `@xyflow/react` v12 (não reactflow antigo) | React 19 compat, maintained |
| Paleta 6 botões esquerda | Intuitivo, sem scroll |
| Config direita painel separado | Não ocupa space do canvas |
| Auto-assign true/false em condition | UX: evita erro manual |
| Validação pré-publish | Falha fast, mensagem clara |
| `useState` puro (sem Context/Redux) | App é simples, overkill global state |
| `apiFetch` + JWT automático | Reutiliza pattern do Inbox |

## Padrões de Código

- **Type Safety:** TypeScript strict, sem `any`, tipos definidos em `types.ts`
- **Componentes:** Funcionais, `useCallback` onde precisa, sem classe
- **Estilos:** Tailwind utilitários, sem arquivo de config
- **HTTP:** `apiFetch` com error handling via alert (simples)
- **Estado:** `useState` local, lifting pra `FlowEditor`
- **Lint:** React Rules respeitadas (deps array, purity, etc)

## Integração com Motor de Execução

O `definition` persistido é exatamente o formato que o motor de fluxo (backend) espera:

```json
{
  "nodes": [
    { "id", "type", "position", "data": {...} },
    ...
  ],
  "edges": [
    { "id", "source", "target", "sourceHandle?" },
    ...
  ]
}
```

Motor ignora `position` (só pra UI), mas lo preserva pro layout survive ao reload. Validações:
- Motor rejeita sem `start` → editor previne
- Motor segue edges até `end` → editor preserva structure

## Próximas Fases

1. **S6 — Testes Automatizados** (pytest + Playwright)
   - Fluxo CRUD via API
   - Montagem/execução de motor
   - Multi-tenant isolation

2. **Validação com Cliente Real**
   - Testar UX do editor (arrastar, config, salvar)
   - Feedback de validação
   - Performance com 10+ nós

3. **Melhorias UI/UX (backlog)**
   - Undo/redo
   - Minimap
   - Node search/palette (comando-K)
   - Snap to grid
   - Themes dark/light

4. **Hardening (pós-Fase 2)**
   - RBAC (owner/admin/agent na API)
   - Rate limiting
   - Fila Celery (webhook + motor async)

## Notas de Debugging

**Problema:** `FlowEditor` importava `nodeTypes` de `nodeTypes.tsx` mas lint reclamava de "fast refresh".  
**Solução:** Mover export de `nodeTypes` object pra arquivo separado `nodeRegistry.ts`.

**Problema:** `onConnect` tinha acesso a `nodes` desatualizado.  
**Solução:** Adicionar `nodes` ao dependency array de `useCallback`.

**Problema:** `Date.now()` + `Math.random()` causavam re-renders não-determinísticos.  
**Solução:** Usar `useRef(counter)` incrementando, posição determinística.

## Vault Links

- [[plano-fase-2-chatbot.md]] — Roadmap S1-S5
- [[fase2-s1-dados.md]] — Models (Flow, FlowSession)
- [[fase2-s2-motor.md]] — Engine (run_session, trigger_inbound)
- [[fase2-s3-question-condition.md]] — Nós question/condition
- [[fase2-s4-handoff.md]] — Nó handoff

---

**Conclusão:** Fase 2 S5 (editor) **100% funcional**. Frontend + Backend integrados, CRUD + publish testados. Pronto pra validação com cliente.
