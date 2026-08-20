---
title: "Engine de matching — similaridade de cosseno com explicação"
aliases: ["Engine do TCC", "Matching de cursos", "Similaridade de cosseno TCC"]
tags: [tcc, engine, algoritmo, recomendacao, python, cosseno]
status: concluido
projeto: TCC
criado: 2026-08-17
---

> [!info] Projeto: [[TCC|🎓 TCC]] · Conceitos: [[cs-linear-algebra|Álgebra linear]] · [[Python|Python]] · [[ORM|ORM]]

# 🧮 Engine de matching por similaridade de cosseno

> [!abstract] O que é
> `quiz/engine.py` — o núcleo do TCC. Converte respostas em vetores, compara com o vetor de cada curso pelo **cosseno do ângulo** entre eles e devolve um ranking com justificativa. Cadastrar o 13º curso no admin já o inclui no cálculo: **nenhum `if` por curso**.

## 🧭 O modelo

As 7 áreas viram os eixos de um espaço vetorial. Perfil e curso viram o mesmo tipo de objeto:

```python
perfil = {"eletrica": 9, "eletromecanica": 8, "mecanica": 3, "ti": 1}
curso  = {"eletrica": 5, "eletromecanica": 4, "ti": 1, "mecanica": 1}
```

O perfil é a **soma** dos pesos das alternativas marcadas. Marcou duas opções que pesam elétrica 4 e 5 → perfil com elétrica 9.

$$\text{similaridade}(A,B) = \frac{A \cdot B}{\|A\| \times \|B\|}$$

Resultado entre 0 e 1: 1 = mesma direção (afinidade total), 0 = nenhuma área em comum.

## 🧱 Anatomia do módulo

| Função | Tipo | Responsabilidade |
|---|---|---|
| `course_vector(course)` | pura | Curso → `{slug: peso}` |
| `profile_vector(choices)` | pura | Alternativas marcadas → `{slug: soma}` |
| `dot(a, b)` | pura | Produto escalar |
| `norm(vetor)` | pura | Tamanho do vetor (Pitágoras em N dimensões) |
| `cosine_similarity(a, b)` | pura | O score de 0 a 1 |
| `explain(profile, vetor, area_names)` | pura | Top 3 áreas que puxaram o score |
| `rank_courses(profile, courses, area_names)` | pura | Ordena com desempate |
| `recommend(attempt, limit=5)` | **impura** | Única que toca o banco |

> [!important] Separação pura / impura
> **Toda a matemática é testável sem banco de dados.** É o que permite os testes de cálculo rodarem em `SimpleTestCase` (milissegundos, sem criar tabela) — ver [[testes-e-validacao-tcc|✅ Testes]]. Também é o argumento de arquitetura mais fácil de defender na banca: o algoritmo não depende do Django.

## 🎯 Por que cosseno, e não soma de pontos

> [!question] A alternativa ingênua
> Somar `peso_perfil × peso_curso` e ordenar pelo total.

O problema: **o curso "gordo" ganharia sempre.** "Automação Industrial" tem 5 áreas preenchidas; "Costura Industrial" tem 2. Numa soma bruta, Automação venceria até para um perfil 100% costura, só por ter mais termos somando.

Dividir pelas normas **neutraliza o tamanho dos vetores** e sobra só a direção — ou seja, a afinidade real. Um curso especialista bate um curso genérico quando o perfil é especialista.

> [!success] O teste que prova isso
> `test_escala_nao_muda_o_score`: dobrar todos os pesos do perfil não altera o resultado. Se alguém trocar o cosseno por uma soma, esse teste quebra primeiro.

## 💡 O `explanation` — o diferencial do trabalho

Para cada curso recomendado, a engine guarda as áreas que mais contribuíram (`perfil[área] × curso[área]`) com o percentual de participação:

```json
{"top_areas": [
  {"area": "eletrica", "area_name": "Elétrica", "contribuicao": 45.0, "percentual": 52.3},
  {"area": "eletromecanica", "area_name": "Eletromecânica", "contribuicao": 32.0, "percentual": 37.2},
  {"area": "mecanica", "area_name": "Mecânica", "contribuicao": 9.0, "percentual": 10.5}
]}
```

Isso permite a tela dizer *"recomendado porque seu perfil pontuou alto em Elétrica e Eletromecânica"* em vez de *"score 0.87"*.

> [!tip] Slug **e** nome, juntos
> O JSON grava os dois: `area` (slug, chave estável para escolher ícone/cor no front) e `area_name` (legível). O nome fica **congelado no histórico** — renomear a área depois não reescreve recomendações antigas, que passam a refletir o que a pessoa realmente viu na tela.
> `recommend` monta `dict(Area.objects.values_list("slug", "name"))` em **uma query** e repassa para o `explain`.

## 🔒 Três detalhes que não são decoração

### 1. Desempate reprodutível
```python
resultado.sort(key=lambda item: (-item[1], item[0].name))
```
Sem o segundo critério, cursos com score idêntico trocam de posição a cada execução. Num TCC isso é falha grave: o avaliador roda duas vezes e vê saídas diferentes.

### 2. Guarda do divisor zero
Quiz em branco → norma 0 → `ZeroDivisionError` na cara do usuário. `cosine_similarity` devolve `0.0` e segue.

### 3. Custo de acesso ao banco
`prefetch_related("area_weights__area")` + `bulk_create` derrubam ~40 consultas para **3**. Rende um parágrafo de otimização na monografia.

## 🔁 Ciclo de `recommend()`

```
attempt (respostas salvas)
   │  select_related + prefetch_related  ← 1 query
   ▼
profile_vector  →  {área: soma}
   │
   ├── Course.objects.filter(is_active=True)  ← 1 query
   ├── dict(Area.objects.values_list(...))    ← 1 query
   ▼
rank_courses  →  [(curso, score, explicacao), ...] ordenado
   │  [:limit]
   ▼
apaga recomendações antigas → bulk_create das 5 novas
```

Tudo dentro de `@transaction.atomic`: ou grava o ranking inteiro, ou não grava nada.

> [!note] Recalcular é seguro
> `attempt.recommendations.all().delete()` antes do `bulk_create` torna a função idempotente. Chamar duas vezes na mesma tentativa continua dando 5 recomendações, não 10 (coberto por `test_recalcular_nao_duplica`).

## 📊 Validação — os 4 critérios de aceite

| Perfil simulado | Esperado | Resultado |
|---|---|---|
| Puro elétrico | Eletricista e Comandos Elétricos no topo | 0.995 / 0.985 ✅ |
| Automotivo + elétrico | **Injeção Eletrônica acima de Motores a Combustão** | 0.968 > 0.891 ✅ |
| Costura | Costura e Modelagem no topo, mecânica no fim | 0.999 / 0.969, mecânica 0.13 ✅ |
| TI e dados | Python e IA no topo | 1.000 / 0.979 ✅ |

> [!success] O critério do meio é o coração da defesa
> Ele só passa se a engine entender **combinação** de áreas. "Injeção Eletrônica" tem menos peso automotivo puro que "Motores a Combustão", mas ganha porque o perfil também pontua elétrica. Nenhuma regra `if` foi escrita para isso — emergiu do cálculo.

E um detalhe que **parece** bug e não é: no perfil de TI, "Automação Industrial" aparece no meio do ranking (0.471). É correto — ela pesa TI 3 e IA 2.

## 🧪 Como demonstrar ao vivo

```bash
PYTHONIOENCODING=utf-8 ./.venv/Scripts/python.exe manage.py test_engine
```

Roda os 4 perfis simulados, imprime o ranking com scores e áreas, e faz `transaction.set_rollback(True)` no fim — **não suja o banco**. É o comando para rodar na apresentação; os testes automatizados são para garantir que nada quebrou.

> [!warning] Os scores desta nota são de perfis com 4 respostas
> A tabela acima vem do `test_engine.py`, que marca **4 alternativas**. Com as **6** que o formulário web exige, os números mudam e a ordem entre os dois primeiros do perfil elétrico **inverte** (Comandos 0,9877 × Eletricista 0,9875). Os dois cálculos estão corretos — são entradas diferentes. Para a monografia, use o perfil de 6 respostas: ver [[artigo-secao-calculo-cosseno|✍️ Artigo: seção do cálculo de cosseno]].

## Veja também

- [[TCC|🎓 TCC]]
- [[artigo-secao-calculo-cosseno|✍️ Artigo: seção do cálculo de cosseno]] — o mesmo cálculo escrito como texto científico
- [[modelagem-dados-quiz|🗃️ Modelagem de dados]]
- [[testes-e-validacao-tcc|✅ Testes e validação]]
- [[defesa-monografia-tcc|🎤 Defesa e monografia]]
