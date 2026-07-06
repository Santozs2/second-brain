---
title: "Multi-Tenancy no Django — Implementação Shared DB, Shared Schema"
aliases: ["Multi-Tenancy no Django — Implementação Shared DB, Shared Schema", "Multi-Tenancy no Django — Implementação"]
tags: [django, multi-tenancy, arquitetura, saas, chatbot]
status: em-desenvolvimento
projeto: ChatBot
criado: 2026-07-06
---

> [!info] Projeto: [[ChatBot|💬 ChatBot]] · Tecnologias: [[Django|Django]] · [[Django REST Framework|DRF]] · [[ORM|ORM]] · [[Models|Models]]

# Multi-Tenancy no Django

> [!abstract] O que é multi-tenancy
> Uma única aplicação servindo múltiplos "clientes" (tenants), cada um com seus próprios dados isolados, sem um ver o do outro. Pense num prédio com múltiplos apartamentos — mesma estrutura, dados separados.

## 🎯 Abordagem escolhida: Shared Database, Shared Schema

- **1 banco de dados** para todos os clientes
- **Todas as tabelas** carregam um campo `organization_id`
- **Isolamento** acontece no código (filter nas queries)
- **Custo:** mínimo
- **Complexidade:** baixa
- **Escalabilidade:** ótima pro MVP

---

## 1️⃣ O modelo base — TenantModel

Cada modelo de negócio (Contact, Conversation, Message) herda de `TenantModel`:

```python
# common/models.py
from django.db import models
import uuid

class TenantModel(models.Model):
    """
    Modelo abstrato que todo modelo de negócio estende.
    O 'organization' é o tenant — a empresa cliente.
    """
    id = models.UUIDField(primary_key=True, default=uuid.uuid4, editable=False)
    organization = models.ForeignKey(
        'organizations.Organization',
        on_delete=models.CASCADE,
        related_name="%(class)ss"
    )
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    class Meta:
        abstract = True
```

**Uso:**
```python
class Contact(TenantModel):
    wa_id = models.CharField(max_length=20)
    name = models.CharField(max_length=255)
    # organization já está herdado automaticamente
```

---

## 2️⃣ Middleware — Captura o tenant da requisição

O middleware identifica qual organização o usuário logado pertence:

```python
# common/middleware.py
from organizations.models import Membership

class TenantMiddleware:
    """
    Injeta request.tenant (a Organization do usuário) em toda requisição.
    Usa a primeira Membership do usuário.
    """
    def __init__(self, get_response):
        self.get_response = get_response

    def __call__(self, request):
        request.tenant = None
        
        if request.user.is_authenticated:
            try:
                membership = request.user.memberships.first()
                request.tenant = membership.organization
            except Membership.DoesNotExist:
                pass
        
        response = self.get_response(request)
        return response
```

**Registra em `settings.py`:**
```python
MIDDLEWARE = [
    # ... middlewares padrão ...
    'common.middleware.TenantMiddleware',
]
```

---

## 3️⃣ Manager customizado — Filtragem automática

```python
# common/managers.py
from django.db import models

class TenantQuerySet(models.QuerySet):
    """QuerySet que sabe filtrar por tenant."""
    
    def for_tenant(self, organization):
        return self.filter(organization=organization)


class TenantManager(models.Manager):
    def get_queryset(self):
        return TenantQuerySet(self.model, using=self._db)

    def for_tenant(self, organization):
        """Shortcut: Model.objects.for_tenant(org)"""
        return self.get_queryset().for_tenant(organization)
```

**Adiciona aos modelos:**
```python
class Contact(TenantModel):
    wa_id = models.CharField(max_length=20)
    name = models.CharField(max_length=255)
    
    objects = TenantManager()  # ← adiciona aqui
```

---

## 4️⃣ Decorator — Protege as views

```python
# common/decorators.py
from functools import wraps
from django.http import Http404

def require_tenant(view_func):
    """
    Garantir que:
    1. User está logado
    2. User pertence a uma organização
    3. request.tenant foi injetado
    """
    @wraps(view_func)
    def wrapper(request, *args, **kwargs):
        if not request.user.is_authenticated:
            from django.contrib.auth.decorators import login_required
            return login_required(view_func)(request, *args, **kwargs)
        
        if not request.tenant:
            raise Http404("Usuário não pertence a nenhuma organização.")
        
        return view_func(request, *args, **kwargs)
    return wrapper
```

---

## 5️⃣ View com DRF — Filtragem obrigatória

```python
# inbox/views.py
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework.permissions import IsAuthenticated
from .models import Conversation
from .serializers import ConversationSerializer

class ConversationListView(APIView):
    permission_classes = [IsAuthenticated]

    def get(self, request):
        # request.tenant foi injetado pelo middleware
        organization = request.tenant
        
        # ⭐ SEMPRE filtrar por tenant
        conversations = Conversation.objects.for_tenant(organization)
        
        serializer = ConversationSerializer(conversations, many=True)
        return Response(serializer.data)
    
    def post(self, request):
        data = request.data
        # ⭐ FORÇA o organization, nunca deixa o client passar
        data['organization'] = organization.id
        
        serializer = ConversationSerializer(data=data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=201)
        return Response(serializer.errors, status=400)
```

---

## 6️⃣ Serializer com DRF — Proteção na criação

```python
# inbox/serializers.py
from rest_framework import serializers
from .models import Conversation

class ConversationSerializer(serializers.ModelSerializer):
    class Meta:
        model = Conversation
        fields = ['id', 'contact', 'status', 'assigned_to', 'created_at']
        read_only_fields = ['id', 'organization', 'created_at']

    def create(self, validated_data):
        # ⭐ Força o organization do request
        validated_data['organization'] = self.context['request'].tenant
        return super().create(validated_data)
```

---

## 7️⃣ Admin multi-tenant (bonus)

```python
# inbox/admin.py
from django.contrib import admin
from .models import Conversation

@admin.register(Conversation)
class ConversationAdmin(admin.ModelAdmin):
    list_display = ('contact', 'status', 'assigned_to', 'organization')
    list_filter = ('status', 'organization')
    
    def get_queryset(self, request):
        qs = super().get_queryset(request)
        # Admin superuser vê tudo; usuário normal vê só sua org
        if request.user.is_superuser:
            return qs
        
        membership = request.user.memberships.first()
        if membership:
            return qs.filter(organization=membership.organization)
        return qs.none()
```

---

## 🔄 O fluxo de uma requisição (mental model)

```
1. User faz: GET /api/conversations/
   ↓
2. Middleware: request.tenant = Organization(id='org_123')
   ↓
3. View: Conversation.objects.for_tenant(request.tenant)
   ↓
4. Django executa:
   SELECT * FROM inbox_conversation 
   WHERE organization_id = 'org_123'
   ↓
5. User vê APENAS suas conversas ✅
```

---

## ⚠️ Regras de ouro

> [!warning] Segurança
> - **NUNCA confie no frontend** — sempre filtre server-side
> - **NUNCA deixe o client passar `organization_id`** como parâmetro
> - **SEMPRE use `request.tenant`** nas queries
> - **SEMPRE force o organization na criação** (serializer.create)

### Teste brutal

Cria 2 users em 2 orgs diferentes:
- User A (Org 1)
- User B (Org 2)

Depois:
1. User A tenta acessar `/api/conversations/` → vê **só conversas de Org 1** ✅
2. User B tenta acessar `/api/conversations/` → vê **só conversas de Org 2** ✅
3. User A tenta passar `?organization=org_2` na URL → **é ignorado**, vê Org 1 mesmo ✅

---

## 🎁 Índices de performance

Adiciona índices às foreign keys de tenant pra garantir queries rápidas:

```python
class Contact(TenantModel):
    wa_id = models.CharField(max_length=20)
    name = models.CharField(max_length=255)
    
    objects = TenantManager()
    
    class Meta:
        indexes = [
            models.Index(fields=['organization', 'wa_id']),
        ]
        constraints = [
            models.UniqueConstraint(
                fields=['organization', 'wa_id'],
                name='unique_contact_per_org'
            )
        ]
```

---

## 📚 Referências

- [[Roadmap — Plataforma de Atendimento WhatsApp]]
- [[Estrutura Django — Fase 1]]
