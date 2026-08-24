---
type: concept
area: Conceitos
status: estavel
difficulty: beginner
id: tst-cobertura
category: Testes
tags:
  - testes
  - concept
  - qualidade
  - metricas
created: 2026-08-24
updated: 2026-08-24
---
# 📊 Cobertura de Testes

> Mede qual percentual do código foi executado pela suíte. É um bom detector de buraco e um péssimo indicador de qualidade.

---

## 📐 Os tipos de cobertura

| Tipo | O que mede | Rigor |
|---|---|---|
| **De linha** | Linhas executadas | Baixo |
| **De comando** | Comandos executados | Baixo |
| **De ramo** (*branch*) | Cada `if` testado nos dois caminhos | **Médio-alto** |
| **De condição** | Cada subcondição booleana | Alto |
| **De caminho** | Todas as combinações de caminhos | Impraticável |

> [!important] Cobertura de ramo é a única que vale acompanhar
> Cobertura de linha considera um `if` coberto quando apenas o caminho verdadeiro foi executado. **Cobertura de ramo exige os dois.** É onde os bugs moram — no `else` que ninguém testou.

```python
def cosine_similarity(a, b):
    na, nb = norm(a), norm(b)
    if na == 0 or nb == 0:      # ← cobertura de LINHA: basta passar aqui
        return 0.0              # ← cobertura de RAMO: exige testar este caso
    return dot(a, b) / (na * nb)
```

---

## 💻 Como medir

```bash
pip install coverage
coverage run --branch manage.py test
coverage report -m
coverage html          # relatório navegável em htmlcov/index.html
```

```
Name                 Stmts   Miss Branch BrPart  Cover   Missing
------------------------------------------------------------------
quiz/engine.py          48      2     14      1    95%   62, 71->exit
quiz/views.py           35     12      8      2    62%   40-55
catalog/models.py       28      0      2      0   100%
------------------------------------------------------------------
TOTAL                  111     14     24      3    84%
```

A coluna **Missing** é a única que importa no dia a dia — ela aponta exatamente as linhas e ramos não exercitados.

---

## 🎯 Qual número perseguir

| Faixa | Leitura |
|---|---|
| < 40% | Provavelmente há área crítica sem teste algum |
| 40–60% | Caminho feliz coberto; casos-limite descobertos |
| 60–80% | Saudável para a maior parte dos projetos |
| 80–90% | Bom; a partir daqui o retorno cai rápido |
| > 95% | Só se justifica em código crítico |
| 100% | Quase sempre custo maior que benefício |

> [!warning] Cobertura alta não significa código correto
> Este teste dá 100% de cobertura e não verifica nada:
> ```python
> def test_engine(self):
>     recommend(self.perfil, self.cursos)   # executou, não afirmou nada
> ```
> Cobertura mede **execução**, não **verificação**. Um teste sem `assert` cobre tanto quanto um teste rigoroso.

---

## 🎯 Cobertura desigual é o objetivo correto

Perseguir um número único para o projeto inteiro é desperdício. A distribuição saudável é **deliberadamente desigual**:

| Módulo | Alvo | Motivo |
|---|---|---|
| Lógica de negócio central | **95%+** | É o que justifica o projeto existir |
| Regras e validações | 85% | Onde nascem os bugs de borda |
| Views / controllers | 60% | Coberto de lado pelos testes de integração |
| Serializers, models simples | 40% | O framework já garante boa parte |
| Migrations, configs, admin | 0% | Testar não agrega |

> [!success] Em trabalho acadêmico, reporte a cobertura do núcleo, não a global
> "84% de cobertura global" é um número sem significado. **"A engine de matching tem 96% de cobertura de ramo, incluindo todos os casos-limite matemáticos"** é evidência de rigor — e é verificável. Ver [[met-defesa-banca|🎤 Defesa de banca]].

---

## 🧨 Como a métrica é distorcida

Quando cobertura vira meta imposta, ela deixa de ser informação — uma instância da **Lei de Goodhart**: *"quando uma medida vira meta, ela deixa de ser uma boa medida."*

| Distorção | Como aparece |
|---|---|
| Testes sem asserção | Executam código para inflar o número |
| Testar getters e setters | Sobe o percentual, não pega bug |
| Excluir arquivos do relatório | Melhora o número sem melhorar nada |
| Testar código gerado | Testa o framework, não o seu código |

---

## ⚙️ Configuração útil

```ini
# .coveragerc
[run]
branch = True
source = quiz,catalog
omit =
    */migrations/*
    */tests/*
    */venv/*
    manage.py

[report]
exclude_lines =
    pragma: no cover
    raise NotImplementedError
    if __name__ == .__main__.:
show_missing = True
```

Em CI, vale falhar o build quando a cobertura **cai** em relação ao commit anterior — isso protege o núcleo sem transformar o número em meta arbitrária. Ver [[CI-CD|CI/CD]].

---

## 📚 Referências

- **Marick, B.** — *How to Misuse Code Coverage*
- **Fowler, M.** — *TestCoverage*, martinfowler.com
- Documentação do **coverage.py**

---

## 🔗 Conceitos relacionados

- [[tst-piramide-de-testes|🔺 Pirâmide de testes]] · [[tst-tdd|🔴 TDD]]
- [[tst-testes-django|🐍 Testes em Django]] · [[CI-CD|CI/CD]]

## Veja também

- [[Conceitos|🧠 Conceitos]]

---

**Status:** ✅ Completo
