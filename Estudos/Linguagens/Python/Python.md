---
type: tech
area: Estudos
status: aprendendo
tecnologia: Python
tags:
  - tech
  - estudo
  - backend
created: 2026-06-30
updated: 2026-06-30
---
# Python

> [!tip] Status
> 🟡 Aprendendo

## 📝 Resumo

Python é uma linguagem de propósito geral, conhecida pela sintaxe limpa. Muito usada em backend (Django, FastAPI), automação e dados.

## 🧠 Conceitos principais

- **Tipos e estruturas**: listas, dicionários, tuplas, sets
- **Funções**: `def`, `*args`, `**kwargs`, lambdas
- **POO**: classes, herança, métodos especiais (`__init__`)
- **Ambientes virtuais**: `venv`, `pip`
- **Comprehensions**: list/dict comprehensions
- **Tratamento de erros**: `try/except`

## 💻 Exemplos

```python
def filtrar_ativos(usuarios: list[dict]) -> list[dict]:
    return [u for u in usuarios if u.get("ativo")]

class Usuario:
    def __init__(self, nome: str):
        self.nome = nome

    def __repr__(self):
        return f"Usuario({self.nome})"
```

## 🔗 Links úteis

- [Documentação oficial Python](https://docs.python.org/pt-br/3/)
- [Real Python](https://realpython.com/)

## ✅ Checklist de aprendizado

- [x] Sintaxe básica e estruturas de dados
- [ ] Programação orientada a objetos
- [ ] Ambientes virtuais e pacotes
- [ ] Tratamento de exceções
- [ ] Testes com `pytest`

## 🗒️ Notas pessoais


## 📖 Aprofundar

- [[lang-py-overview|Guia detalhado de Python]] — visão geral e sintaxe aprofundada

## 🔗 Veja também

- [[Django|Django]]
- [[Django REST Framework|Django REST Framework]]
- [[Banco de Dados|Banco de Dados]]
- [[Snippets - Python|Snippets de Python]]

## 🔤 Outras linguagens

- [[lang-go-overview|Go]] · [[JavaScript|JavaScript]] · [[TypeScript|TypeScript]]
