---
type: snippet
tags:
  - snippet
linguagem: Python
created: 2026-06-30
updated: 2026-06-30
status: estavel
---

# 🧩 Snippets de Python

> [!tip] Como usar
> Cada `###` abaixo é um snippet independente. Adicione novos no final usando o mesmo padrão.

## List comprehension com filtro

```python
ativos = [u for u in usuarios if u["ativo"]]
```

**Quando usar:** filtrar ou transformar listas de forma concisa, sem usar `for` tradicional.

#snippet #python

---

## Context manager para arquivos

```python
with open("dados.txt", "r", encoding="utf-8") as arquivo:
    conteudo = arquivo.read()
```

**Quando usar:** sempre que abrir arquivos — garante que serão fechados automaticamente.

#snippet #python

---

## Ambiente virtual

```bash
python -m venv venv
source venv/bin/activate   # Linux/Mac
venv\Scripts\activate      # Windows

pip install -r requirements.txt
pip freeze > requirements.txt
```

**Quando usar:** no início de qualquer projeto Python, para isolar dependências.

#snippet #python

## Veja também

- [[Python|Python]]
- [[Snippets|Todos os snippets]]
