---
type: tech
tags:
  - tech
  - estudo
  - backend
  - api
tecnologia: Django REST Framework
status: aprendendo
aliases:
  - DRF
created: 2026-06-30
updated: 2026-06-30
---

# Django REST Framework (DRF)

> [!tip] Status
> 🟡 Aprendendo

## 📝 Resumo

DRF é uma extensão do Django para construir APIs REST de forma rápida, com serializers, viewsets e autenticação prontos.

## 🧠 Conceitos principais

- **Serializers**: validação e conversão entre Python e JSON
- **Views**: `APIView`, `ViewSet`, `ModelViewSet`
- **Routers**: geração automática de URLs
- **Autenticação e permissões**: Token, JWT, `IsAuthenticated`
- **Paginação e filtros**

## 💻 Exemplos

```python
# serializers.py
from rest_framework import serializers
from .models import Tarefa

class TarefaSerializer(serializers.ModelSerializer):
    class Meta:
        model = Tarefa
        fields = ["id", "titulo", "concluida"]
```

```python
# views.py
from rest_framework import viewsets
from .models import Tarefa
from .serializers import TarefaSerializer

class TarefaViewSet(viewsets.ModelViewSet):
    queryset = Tarefa.objects.all()
    serializer_class = TarefaSerializer
```

## 🔗 Links úteis

- [Documentação oficial DRF](https://www.django-rest-framework.org/)

## ✅ Checklist de aprendizado

- [ ] Serializers
- [ ] ViewSets e Routers
- [ ] Autenticação (Token/JWT)
- [ ] Permissões customizadas
- [ ] Documentação de API (Swagger)

## 🗒️ Notas pessoais


## 🧩 Conceitos relacionados

- [[Conceitos/REST API|REST API]]
- [[Conceitos/Serializers|Serializers]]
- [[Conceitos/CRUD|CRUD]]
- [[Conceitos/JWT|JWT]]

## 🔗 Veja também

- [[Estudos/Django|Django]]
- [[Estudos/Python|Python]]
