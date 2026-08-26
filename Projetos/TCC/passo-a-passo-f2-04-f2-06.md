---
title: "Passo a passo — F2-04 a F2-06 (prompt em arquivo, carregador e validação)"
aliases: ["Passo a passo Bloco 3", "entrega_v1", "DeliverySerializer", "Validacao da saida da LLM"]
tags: [tcc, ia, llm, prompt, validacao, django, execucao, passo-a-passo]
status: em-andamento
projeto: TCC
criado: 2026-08-26
---

> [!info] Plano: [[plano-execucao-f1-f2|🗂️ Plano de execução F1+F2]] · Prompt: [[prompt-padrao-recomendacao|📝 Prompt v1]] · Contratos: [[spec-motor-e-ia-frentes-1-2|🧭 Spec F1+F2]] · Bloco anterior: [[passo-a-passo-f2-01-f2-03|🔌 F2-01 a F2-03]]

# 📝 Passo a passo — F2-04 a F2-06

> [!abstract] O que esta nota é
> O **Bloco 3**: o prompt sai da vault e vira arquivo versionado, ganha um carregador que injeta os dados, e nasce o validador que **não deixa nada que a LLM devolva entrar no sistema sem passar por revista**.
> Ainda sem rede: tudo roda contra o `FakeProvider`. Escrito contra a branch `feat/ia-provider-offline` mesclada.

> [!warning] Nada disto foi executado
> A seção 5 é a verificação.

> [!success] O que este bloco entrega, em uma frase
> Um provedor que devolve lixo proposital **não derruba a aplicação e não grava nada inválido**. É o Definition of Done do Passo 8 do [[camada-ia-plano-implementacao|plano da camada de IA]], e é o que permite ligar o Gemini no Bloco 4 sem medo.

---

## 🚦 Antes de abrir o editor

```bash
git checkout main && git pull
git checkout -b feat/ia-prompt-e-validacao
```

---

## 1️⃣ F2-04 — `quiz/prompts/entrega_v1.md`

> [!danger] A regra 2 **muda** em relação à nota do vault
> [[prompt-padrao-recomendacao|📝 Prompt v1]] ainda fala em *"diferença de score menor que 0,05"*. Isso saiu quando o grupo ratificou o eixo D: quem calcula o empate é a engine, e o prompt recebe a lista pronta. **Não copie a nota literalmente** — use o texto abaixo, que já está corrigido. E depois atualize a nota, senão as duas versões divergem e ninguém sabe qual vale.

Crie `quiz/prompts/` e o arquivo:

````markdown
# PAPEL

Você é um orientador de carreira de uma escola técnica profissionalizante.
Você está falando diretamente com um estudante que acabou de responder a um
quiz de perfil. Um sistema já analisou as respostas dele e selecionou os
cursos mais compatíveis. Seu trabalho é **entregar essa recomendação** em
linguagem humana: apresentar o curso principal e mencionar as alternativas.

Você não é o sistema que escolheu os cursos. Você é quem explica a escolha.

# DADOS

## Perfil calculado do estudante
{{PERFIL_JSON}}

## O que ele respondeu, na ordem
{{RESPOSTAS_JSON}}

## Confiança do cálculo
{{CONFIANCA_JSON}}

## Cursos candidatos, em ordem de compatibilidade calculada
{{CANDIDATOS_JSON}}

# REGRAS

1. Escolha o curso principal **apenas** entre os `course_id` da lista de
   candidatos. Nunca cite, sugira ou invente um curso que não esteja ali.

2. O curso principal tem que ser um dos ids listados em `conjunto_empate`.
   - Se a lista tiver **um id só**, ele é o principal. A diferença de
     compatibilidade é real e não deve ser contrariada.
   - Se tiver **mais de um**, esses candidatos estão tecnicamente empatados no
     cálculo: escolha entre eles usando o que o estudante respondeu.

3. Os candidatos precisam aparecer todos na resposta: um como principal e os
   demais como alternativas. Não repita nenhum e não omita nenhum.

4. O campo `banda` define o grau de certeza do seu texto:
   - `alta` — pode afirmar que o curso combina muito com o perfil dele.
   - `media` — diga que é o que mais se aproxima do que ele descreveu.
   - `baixa` — **não finja certeza**. Apresente como caminho mais próximo e
     sugira conversar com um orientador da escola.

5. Só afirme sobre um curso o que estiver no campo `descricao`. **Não** fale de
   duração, preço, unidade, mercado de trabalho, salário, empregabilidade ou
   qualquer promessa de resultado.

6. Escreva em português do Brasil, na segunda pessoa ("você"), com tom acolhedor
   e direto. Nada de jargão do sistema: não use as palavras *score*, *cosseno*,
   *vetor*, *algoritmo*, *ranking*, *pontuação*, *empate* nem *inteligência
   artificial*.

7. Ao justificar, conecte o curso ao que ele respondeu e às áreas do perfil,
   citando-as pelo `area_name` (ex.: "Elétrica", "Mecânica").

8. Tamanhos: o texto do principal tem de 3 a 5 frases (máximo 600 caracteres);
   o de cada alternativa tem de 1 a 2 frases (máximo 220 caracteres).

9. Não comece duas alternativas com a mesma palavra ou com a mesma fórmula.

10. Não faça perguntas ao estudante, não peça mais informações e não sugira que
    ele refaça o quiz.

11. Responda **somente** com o JSON do formato abaixo: sem cercas de código,
    sem comentários, sem texto antes ou depois.

12. Se por qualquer motivo você não conseguir cumprir as regras acima, devolva
    o JSON mantendo exatamente a ordem recebida em `rank_engine`.

# FORMATO DE SAÍDA

{
  "principal": {
    "course_id": <int>,
    "texto": "<3 a 5 frases>"
  },
  "alternativas": [
    {"course_id": <int>, "texto": "<1 a 2 frases>"}
  ]
}
````

> [!important] A regra 2 é a que diferencia este TCC — e agora ela é verificável
> Antes, *"não reordene fora do limiar"* era instrução em linguagem natural que o servidor não tinha como checar. Com o `conjunto_empate` calculado pela engine e enviado no payload, **o servidor sabe exatamente quem podia ser promovido** e recusa o resto. Instrução no prompt **mais** revalidação no servidor: cinto e suspensório, igual à regra 1 contra alucinação de curso.

> [!note] A regra 4 é o eixo C, e é decisão ética antes de ser de UX
> Um sistema que fala com a mesma segurança sobre 0,98 e sobre 0,31 está mentindo para um estudante sobre a formação profissional dele. A banda vem calculada do motor; a regra 4 é o que faz ela chegar ao texto.

> [!warning] O bloco `respostas` novo tem um efeito colateral bom e um risco
> **Bom:** a LLM passa a poder justificar a escolha citando o que a pessoa respondeu — e é isso que torna a avaliação humana do Bloco 5 possível, porque dá para ver *por que* ela desempatou daquele jeito.
> **Risco:** hoje as alternativas são fechadas e escritas pelo grupo, então não há superfície de injeção. **No dia em que entrar um campo aberto no quiz**, esse texto vira dado não confiável e precisa ser delimitado, não concatenado direto no bloco de dados.

---

## 2️⃣ F2-05 — o carregador

### `quiz/prompt.py` — arquivo novo

```python
"""Carrega o prompt do disco e injeta os dados. Nenhuma f-string de prompt em view."""

import json
from pathlib import Path

from django.conf import settings

PROMPTS_DIR = Path(__file__).resolve().parent / "prompts"


def carregar(versao=None):
    versao = versao or settings.LLM_PROMPT_VERSION
    caminho = PROMPTS_DIR / f"entrega_{versao}.md"
    if not caminho.exists():
        raise FileNotFoundError(f"Prompt nao encontrado: {caminho}")
    return caminho.read_text(encoding="utf-8")


def montar(payload, versao=None):
    """Payload do contrato C1 -> texto do prompt pronto para o provedor."""
    texto = carregar(versao)

    for chave, bloco in (
        ("PERFIL_JSON", payload["perfil"]),
        ("RESPOSTAS_JSON", payload["respostas"]),
        ("CONFIANCA_JSON", payload["confianca"]),
        ("CANDIDATOS_JSON", payload["candidatos"]),
    ):
        texto = texto.replace(
            "{{" + chave + "}}", json.dumps(bloco, ensure_ascii=False, indent=2)
        )

    if "{{" in texto:
        raise ValueError(f"Placeholder nao substituido no prompt {versao or 'padrao'}.")

    return texto
```

> [!success] A guarda do `{{` vale mais que parece
> Um placeholder escrito errado no `.md` — `{{CANDIDATOS}}` em vez de `{{CANDIDATOS_JSON}}` — não daria erro nenhum: o texto seguiria para o modelo **com a chave literal no lugar dos dados**, e ele responderia alguma coisa plausível sobre cursos que nunca viu. Três linhas que transformam um bug silencioso e caro em exceção imediata.

> [!note] Lê o arquivo a cada chamada, de propósito
> Dá para cachear com `functools.lru_cache` e economizar um `read_text` de alguns KB. **Não vale a pena agora**: ler toda vez significa que editar o prompt e recarregar a página basta para ver o efeito, sem reiniciar o `runserver` — que é exatamente o ciclo de trabalho do Bloco 4, quando você vai iterar no texto. Se um dia virar gargalo (não vai, num TCC), o cache é um decorador.

---

## 3️⃣ F2-06 — o validador

### `quiz/serializers.py` — acrescente no fim

```python
class ItemEntregaSerializer(serializers.Serializer):
    """Base dos dois itens. O limite de texto muda entre principal e alternativa."""

    course_id = serializers.IntegerField()
    texto = serializers.CharField(allow_blank=False, trim_whitespace=True)


class PrincipalSerializer(ItemEntregaSerializer):
    texto = serializers.CharField(allow_blank=False, trim_whitespace=True, max_length=600)


class AlternativaSerializer(ItemEntregaSerializer):
    texto = serializers.CharField(allow_blank=False, trim_whitespace=True, max_length=220)


class DeliverySerializer(serializers.Serializer):
    """As 5 regras do contrato C2. Nada aqui grava: quem grava e o delivery."""

    principal = PrincipalSerializer()
    alternativas = AlternativaSerializer(many=True)

    def __init__(self, *args, ids_permitidos=None, conjunto_empate=None, **kwargs):
        self.ids_permitidos = list(ids_permitidos or [])
        self.conjunto_empate = list(conjunto_empate or [])
        super().__init__(*args, **kwargs)

    def validate(self, attrs):
        ids = [attrs["principal"]["course_id"]] + [
            item["course_id"] for item in attrs["alternativas"]
        ]

        # Regra 1 - nenhum id que nao foi enviado no prompt
        fantasmas = set(ids) - set(self.ids_permitidos)
        if fantasmas:
            raise serializers.ValidationError(f"Curso fora dos candidatos: {sorted(fantasmas)}")

        # Regra 2 - sem repeticao
        if len(ids) != len(set(ids)):
            raise serializers.ValidationError("Curso repetido na resposta.")

        # Regra 3 - nenhum candidato descartado
        sumidos = set(self.ids_permitidos) - set(ids)
        if sumidos:
            raise serializers.ValidationError(f"Candidato omitido: {sorted(sumidos)}")

        # Regra 5 - o principal tem que estar no conjunto de empate
        principal = attrs["principal"]["course_id"]
        if principal not in self.conjunto_empate:
            raise serializers.ValidationError(
                f"Principal {principal} fora do conjunto de empate {self.conjunto_empate}."
            )

        return attrs
```

> [!note] A regra 4 não aparece no `validate` — ela vive nos campos
> Texto vazio é barrado por `allow_blank=False`, e o tamanho pelos `max_length` diferentes de principal e alternativa. Regra de campo se escreve no campo; só o que depende de **relação entre campos** desce para o `validate`.

> [!tip] "Exatamente 5 ids" seria redundante
> As regras 1 e 3 juntas fixam o conjunto exatamente (nada a mais, nada a menos) e a 2 fixa a multiplicidade. Uma quarta checagem de contagem não recusaria nada que já não estivesse recusado — e checagem redundante em validador é o tipo de código que sobrevive a mudanças de regra e passa a recusar coisa certa.

### `quiz/delivery.py` — parse e a porta de entrada

```python
import json
import re

from quiz.serializers import DeliverySerializer

CERCA = re.compile(r"^\s*```(?:json)?\s*|\s*```\s*$", re.MULTILINE)


class EntregaInvalida(Exception):
    """Saida do provedor recusada. Quem chama cai no fallback da engine."""


def parse_saida(texto_cru):
    """Tolera cerca de codigo. Qualquer outra sujeira vira EntregaInvalida."""
    limpo = CERCA.sub("", texto_cru or "").strip()
    try:
        return json.loads(limpo)
    except (json.JSONDecodeError, TypeError) as erro:
        raise EntregaInvalida(f"JSON invalido: {erro}") from erro


def validar_entrega(texto_cru, payload):
    """Texto cru do provedor -> dados validados, ou EntregaInvalida."""
    dados = parse_saida(texto_cru)

    serializer = DeliverySerializer(
        data=dados,
        ids_permitidos=[c["course_id"] for c in payload["candidatos"]],
        conjunto_empate=payload["confianca"]["conjunto_empate"],
    )
    if not serializer.is_valid():
        raise EntregaInvalida(serializer.errors)

    return serializer.validated_data
```

> [!important] Tolerar a cerca de código é decisão consciente, não preguiça
> A regra 11 do prompt proíbe cerca. Modelos desobedecem essa regra específica o tempo todo — é o comportamento mais treinado que existe em resposta a "devolva JSON". Cair em fallback por causa de três crases seria descartar uma resposta perfeitamente boa e **contaminar a taxa de fallback do experimento** com um problema de formatação. Toleramos a cerca e recusamos o resto.

> [!warning] Uma exceção só para todos os motivos de recusa
> `EntregaInvalida` cobre JSON quebrado, id fantasma, repetição, omissão, texto vazio e principal fora do empate. Quem chama, no Bloco 4, faz um `except EntregaInvalida` e cai no fallback — **sem precisar saber qual regra falhou**. O motivo vai para o log e para os metadados; a decisão é sempre a mesma. Multiplicar tipos de exceção aqui só criaria caminhos de código que ninguém testa.

---

## 4️⃣ Os testes — `quiz/tests_llm.py`

Estas são as classes que fecham o Definition of Done do bloco.

```python
class PromptTest(SimpleTestCase):
    def test_monta_substituindo_todos_os_placeholders(self):
        texto = montar(PAYLOAD_FALSO)
        self.assertNotIn("{{", texto)
        self.assertIn("Eletricista", texto)

    def test_placeholder_orfao_estoura_em_vez_de_ir_para_o_modelo(self):
        # Simule um prompt com chave errada e garanta o ValueError.
        ...


class ValidacaoTest(SimpleTestCase):
    """Cada teste representa uma forma de a LLM sair do combinado."""

    def test_saida_feliz_passa(self): ...
    def test_cerca_de_codigo_e_tolerada(self): ...
    def test_json_quebrado_recusa(self): ...
    def test_curso_fantasma_recusa(self): ...
    def test_curso_repetido_recusa(self): ...
    def test_candidato_omitido_recusa(self): ...
    def test_texto_vazio_recusa(self): ...
    def test_texto_gigante_recusa(self): ...
    def test_principal_fora_do_conjunto_de_empate_recusa(self): ...
```

O corpo dos oito de recusa é sempre o mesmo formato:

```python
    def test_curso_fantasma_recusa(self):
        saida = json.dumps({
            "principal": {"course_id": 999, "texto": "texto valido aqui."},
            "alternativas": [...],
        })
        with self.assertRaises(EntregaInvalida):
            validar_entrega(saida, PAYLOAD_FALSO)
```

> [!danger] Nenhum desses testes pode terminar em 500 — e nenhum pode passar
> São as duas metades do mesmo Definition of Done. Um teste que espera exceção e recebe `AssertionError` do próprio serializer, ou que passa quando deveria recusar, quebra a promessa que sustenta a camada inteira: *"desligue a internet em qualquer ponto depois da engine e o TCC continua respondendo"*.

> [!success] Use o `FakeProvider(promover_indice=...)` no teste da regra 5
> Foi para isso que o parâmetro existe. `FakeProvider(promover_indice=3)` promove o quarto candidato — que, num payload com `conjunto_empate` de dois, está fora. É a forma de exercitar a regra 5 com o mesmo dublê da suíte, sem escrever JSON na mão nem depender de sorte.

> [!note] Monte um `PAYLOAD_FALSO` no topo do arquivo e reaproveite
> Um dicionário no formato do C1, com 5 candidatos e `conjunto_empate` de 2. Todos os testes deste bloco usam o mesmo — e ele é a documentação executável do contrato, ao lado do teste `test_payload_bate_o_contrato_c1` que já existe do Bloco 1.

---

## 5️⃣ Verificação

```bash
./.venv/Scripts/python.exe manage.py makemigrations --check --dry-run
PYTHONIOENCODING=utf-8 ./.venv/Scripts/python.exe manage.py test
```

**~40 testes verdes.** Nenhum modelo mudou, então o `--check` tem que sair limpo.

E o olho humano — agora com o prompt de verdade:

```bash
PYTHONIOENCODING=utf-8 ./.venv/Scripts/python.exe manage.py shell
```

```python
from quiz.models import QuizAttempt
from quiz.delivery import build_payload
from quiz.prompt import montar

print(montar(build_payload(QuizAttempt.objects.first())))
```

**Leia o prompt inteiro uma vez, do começo ao fim.** É literalmente o texto que vai para o Gemini no Bloco 4. Confira três coisas: os quatro blocos de dados foram preenchidos, o `conjunto_empate` aparece dentro de `CONFIANCA_JSON`, e não sobrou nenhuma chave `{{...}}`.

---

## 6️⃣ Commit e PR

```bash
git add -A && git commit -m "feat(ia): prompt v1 em arquivo, carregador e validacao da saida"
git push -u origin feat/ia-prompt-e-validacao
```

```markdown
## Toca contrato de outra frente? ( ) não (x) sim → C2
Implementa as 5 regras de validacao da saida da LLM. A regra 2 do prompt
passou a falar em conjunto_empate, conforme o eixo D ratificado pelo grupo.
Nada muda no site: a camada continua desligada por padrao.
```

> [!important] Depois do merge, congele a v1 e corrija a nota do vault
> Duas coisas, na ordem:
> 1. Atualize [[prompt-padrao-recomendacao|📝 Prompt v1]] com o texto que foi para o arquivo — hoje ela ainda diz 0,05 na regra 2.
> 2. A partir do momento em que o Bloco 4 rodar a primeira bateria de perfis reais, **a v1 está congelada**. Toda mudança depois disso cria `entrega_v2.md`, porque os resultados gravados apontam para a versão por `prompt_version`. O `git diff` entre v1 e v2 vira o trecho de metodologia sobre iteração de prompt — e ele vale mais que um prompt que nasceu pronto.

## ▶️ Próxima ação

**Bloco 4** — `feat/ia-entrega`: `delivery.deliver()` com fallback, `GeminiProvider` com timeout e cache, e o endpoint do contrato C3. É o primeiro bloco que precisa de credencial — e o único item dele que a Frente 4 está esperando é o **F2-09**, então priorize.

## 📎 Veja também

- [[plano-execucao-f1-f2|🗂️ Plano de execução F1+F2]] · [[spec-motor-e-ia-frentes-1-2|🧭 Spec F1+F2]]
- [[passo-a-passo-f2-01-f2-03|🔌 Bloco 2]] · [[comando-ver-payload|🔍 ver_payload]] · [[passo-a-passo-f1-04-f1-03|🎚️ Bloco 1]]
- [[prompt-padrao-recomendacao|📝 Prompt v1]] · [[camada-ia-plano-implementacao|🧩 Plano da camada de IA]]
- **Conceitos:** [[ia-engenharia-de-prompt|Engenharia de prompt]] · [[ia-alucinacao-e-grounding|Alucinação e grounding]] · [[ia-avaliacao-de-llm|Avaliação de LLM]]
- [[TCC|🎓 TCC]] · [[Projetos|🚀 Projetos]]
