---
type: tech
area: Estudos
status: aprendendo
tecnologia: Django
tags:
  - tech
  - estudo
  - backend
created: 2026-06-30
updated: 2026-06-30
---
# Django

> [!tip] Status
> 🟡 Aprendendo

## 📝 Resumo

Django é um framework web em Python "baterias inclusas": ORM, admin, autenticação e roteamento prontos para uso.

## 🧠 Conceitos principais

- **Projetos e apps**: `django-admin startproject`, `startapp`
- **Models e ORM**: `models.py`, migrations
- **Views**: baseadas em função e em classe (CBV)
- **URLs**: `urls.py`, roteamento
- **Templates** (quando não usado como API)
- **Admin**: painel administrativo automático
- **Settings**: configuração por ambiente

## 💻 Exemplos

```python
# models.py
from django.db import models

class Tarefa(models.Model):
    titulo = models.CharField(max_length=200)
    concluida = models.BooleanField(default=False)
    criada_em = models.DateTimeField(auto_now_add=True)

    def __str__(self):
        return self.titulo
```

```bash
python manage.py makemigrations
python manage.py migrate
python manage.py runserver
```

## 🔗 Links úteis

- [Documentação oficial Django](https://docs.djangoproject.com/pt-br/)
- [Django Girls Tutorial](https://tutorial.djangogirls.org/pt/)

## ✅ Checklist de aprendizado

- [ ] Models e migrations
- [ ] Views (função e classe)
- [ ] Admin do Django
- [ ] Autenticação de usuários
- [ ] Integração com [[Django REST Framework|DRF]]

## 🗒️ Notas pessoais


## 🧩 Conceitos relacionados

- [[MVC|MVC]]
- [[Models|Models]]
- [[Views|Views]]
- [[ORM|ORM]]
- [[Migrations|Migrations]]

## 🔗 Veja também

- [[Python|Python]]
- [[Django REST Framework|Django REST Framework]]
- [[Banco de Dados|Banco de Dados]]
