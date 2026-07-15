---
tipo: seguranca
projeto: ChatBot
data: 2026-07-15
status: isolamento-aprovado
tags:
  - chatbot
  - seguranca
  - multi-tenant
  - pentest
---

# Segurança — Teste de Isolamento Multi-Tenant

> [!success] Resultado
> Isolamento entre organizações **aprovado**: 6 vetores de ataque bloqueados, nenhum vazamento de dados entre tenants. Um bug de robustez (HTTP 500 em tenant nulo) foi encontrado e corrigido.

Relacionado: [[guia-fase-1-inbox]] · [[plano-fase-2-chatbot]] · [[memoria-tecnica-claude]]

## Contexto

O ChatBot é um SaaS **multi-tenant**: várias organizações (clientes) compartilham o mesmo banco. A garantia mais crítica é que um usuário da **org A nunca acesse dados da org B** — nem lendo, nem escrevendo, nem trocando cabeçalhos, nem adivinhando UUIDs.

O isolamento é garantido por duas camadas:
- **`TenantMiddleware._resolve_tenant`** (`common/middleware.py`): resolve `request.tenant` considerando **apenas memberships do usuário autenticado**. Trocar o header `X-Organization` para uma org que o usuário não é membro → `tenant = None` (não dá acesso).
- **`TenantModel.objects.for_tenant(tenant)`** (`common/models.py`): todo queryset de dados de tenant é filtrado por `organization=tenant`.

## Metodologia do pentest

Script Python one-off com o `APIClient` do DRF, executado **dentro de uma transação com rollback no final** — cria org A (vítima) + org B (atacante) temporárias, roda os ataques e desfaz tudo. **Nada é gravado no banco de produção (Supabase).**

Pontos técnicos:
- `APIClient(raise_request_exception=False)` → captura respostas 500 sem re-lançar a exceção (senão o teste aborta).
- Passar `SERVER_NAME='localhost'` em toda request (para satisfazer `ALLOWED_HOSTS`).
- Envolver tudo em `transaction.atomic()` e chamar `transaction.set_rollback(True)` no fim.
- Autenticar com `AccessToken.for_user(user)` (SimpleJWT) + header `X-Organization=<slug>`.

## Vetores de ataque testados

| # | Ataque | Esperado | Resultado |
|---|--------|----------|-----------|
| 1 | Listar conversas **sem token** | 401 | ✅ 401 |
| 2 | Atacante B lista as próprias conversas (isolamento) | não vê as de A | ✅ isolado |
| 3 | B **falsifica** `X-Organization` com o slug de A | não vaza | ✅ lista vazia |
| 4 | **IDOR leitura**: B abre conversa de A pelo UUID | 404 | ✅ 404 |
| 5 | **IDOR escrita**: B muda o status da conversa de A | 404 | ✅ 404 |
| 6 | **Cross-tenant**: B tenta se atribuir a conversa de A | 404 | ✅ 404 |

## Bug encontrado e corrigido

> [!bug] HTTP 500 em tenant nulo (info disclosure)
> No ataque #3, quando o header falsificado resultava em `tenant = None`, o `for_tenant(None)` executava `.filter(organization=<None>)` e **estourava HTTP 500** ao tentar converter `None` em UUID. Não vazava dados, mas derrubava o endpoint para qualquer usuário sem tenant resolvível e, com `DEBUG=True`, exibia o stack trace inteiro.

**Correção** (`common/models.py`, `TenantManager.for_tenant`):

```python
def for_tenant(self, tenant):
    if not tenant:
        return self.get_queryset().none()
    return self.get_queryset().filter(organization=tenant)
```

Detalhe importante: usar **`if not tenant`** e **não** `is None`. O `request.tenant` é um `SimpleLazyObject`, então `is None` falha (é um wrapper, não o `None` puro); `not tenant` força a avaliação e cobre tanto `None` quanto o lazy-None, enquanto uma org real é *truthy*. A correção protege **todos os viewsets de tenant de uma vez** (Conversation e os futuros de `flows`).

Commit: `fix: for_tenant retorna queryset vazio quando tenant e None`.

## Próximos testes de segurança (pendentes, por prioridade)

1. **Assinatura do webhook da Meta (`X-Hub-Signature-256`)** — o webhook é `AllowAny` sem validar assinatura; hoje dá para forjar mensagens de cliente. *Gap mais sério.*
2. **RBAC** — papéis `owner/admin/agent` não são checados em nenhum endpoint (qualquer membro faz tudo).
3. **Brute-force no login** — sem throttling em `/api/auth/login/`.
4. **Robustez do JWT** — token expirado/adulterado/de usuário deletado.
5. **Re-rodar a suíte de invasão contra `/api/flows/`** quando o CRUD estiver pronto.
