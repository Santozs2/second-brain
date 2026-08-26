---
title: "Passo a passo — F2-01 a F2-03 (protocolo, FakeProvider e configuração)"
aliases: ["Passo a passo Bloco 2", "FakeProvider", "Camada de IA offline", "get_provider"]
tags: [tcc, ia, llm, django, execucao, passo-a-passo, testes]
status: em-andamento
projeto: TCC
criado: 2026-08-26
---

> [!info] Plano: [[plano-execucao-f1-f2|🗂️ Plano de execução F1+F2]] · Spec: [[spec-motor-e-ia-frentes-1-2|🧭 Spec F1+F2]] · Plano da IA: [[camada-ia-plano-implementacao|🧩 Camada de IA]] · Bloco anterior: [[passo-a-passo-f1-04-f1-03|🎚️ F1-04 e F1-03]]

# 🔌 Passo a passo — F2-01 a F2-03

> [!abstract] O que esta nota é
> O **Bloco 2**: a camada de IA inteira de pé, **sem uma linha de rede**. Protocolo do provedor, o dublê que devolve resposta válida offline, e a configuração por `.env` que permite desligar tudo.
> Escrito contra o repositório no commit `28b191e`.

> [!warning] Nada disto foi executado
> Código escrito contra a leitura do repositório, não rodado. **A seção 5 é o que prova que deu certo.**

> [!success] Este é o bloco que blinda o cronograma
> Nenhuma das três tarefas depende de credencial, cota ou internet. Quando a chave do Gemini chegar, o Bloco 4 é plugar — e se ela não chegar, os Blocos 2 e 3 já entregaram a camada testada. É por isso que ele vem antes, e não depois.

---

## 🚦 Antes de abrir o editor

```bash
git checkout main && git pull
git checkout -b feat/ia-provider-offline
```

```bash
./.venv/Scripts/python.exe -m pip install python-dotenv
./.venv/Scripts/python.exe -m pip freeze | grep -i dotenv >> requirements.txt
```

> [!danger] O `settings.py` é o único arquivo que você disputa com a autenticação
> O colega escreve no bloco dele, logo abaixo de `AUTH_PASSWORD_VALIDATORS`. **Escreva sempre no fim do arquivo**, depois do `RECOMMENDATION_LIMIT`, num bloco com cabeçalho comentado. Regiões diferentes, merge sem conflito. Se os dois escreverem no mesmo lugar, o Git pede intervenção manual à toa.

---

## 1️⃣ F2-01 — o protocolo

Crie o pacote:

```
quiz/llm/
├── __init__.py
├── base.py
└── fake.py
```

### `quiz/llm/base.py`

```python
"""Contrato do provedor de LLM. Nenhum SDK e importado aqui."""

from typing import Protocol


class LLMTimeout(Exception):
    """O provedor nao respondeu dentro do limite configurado."""


class LLMUnavailable(Exception):
    """Sem credencial, sem cota, provedor desconhecido ou erro do servico."""


class LLMProvider(Protocol):
    """Um metodo so: recebe o prompt montado, devolve o texto cru da resposta."""

    def complete(self, prompt: str) -> str: ...
```

> [!important] Um método só, e ele recebe `str`
> A tentação é passar o payload já estruturado, porque o `FakeProvider` iria adorar. Mas aí o dublê deixa de exercitar o mesmo caminho do modelo real: **o Gemini recebe texto e devolve texto**, e é nesse formato que os bugs de parsing acontecem. Protocolo que facilita a vida do dublê esconde exatamente o que o dublê deveria estar testando.

> [!note] `Protocol`, não classe base abstrata
> Ninguém precisa herdar de `LLMProvider` para satisfazê-lo — basta ter o método. Isso mantém `quiz/llm/gemini.py` livre para ser uma casca fina em volta do SDK, sem acoplamento de herança, e permite trocar de provedor sem tocar em `delivery.py`.

---

## 2️⃣ F2-02 — o `FakeProvider`

### `quiz/llm/fake.py`

```python
"""Dublê offline. Devolve C2 valido a partir dos ids que achou no prompt."""

import json
import re

from quiz.llm.base import LLMUnavailable

PADRAO_ID = re.compile(r'"course_id"\s*:\s*(\d+)')


class FakeProvider:
    def __init__(self, promover_indice=0):
        # 0 = respeita a ordem da engine. Outro valor promove aquele candidato,
        # que e como os testes do Bloco 3 exercitam a divergencia.
        self.promover_indice = promover_indice

    def complete(self, prompt: str) -> str:
        ids = self._ids_do_prompt(prompt)
        if not ids:
            raise LLMUnavailable("Nenhum course_id encontrado no prompt.")

        indice = min(self.promover_indice, len(ids) - 1)
        principal = ids[indice]
        alternativas = [i for i in ids if i != principal]

        return json.dumps(
            {
                "principal": {
                    "course_id": principal,
                    "texto": (
                        "Esse curso conversa com o que voce respondeu. "
                        "Ele trabalha as areas que mais apareceram no seu perfil. "
                        "E um bom ponto de partida para a sua formacao."
                    ),
                },
                "alternativas": [
                    {"course_id": i, "texto": f"Tambem combina com o seu perfil (curso {i})."}
                    for i in alternativas
                ],
            },
            ensure_ascii=False,
        )

    @staticmethod
    def _ids_do_prompt(prompt):
        """Ordem de aparicao, sem repetir: e a ordem do rank_engine no payload."""
        vistos, ordenados = set(), []
        for achado in PADRAO_ID.findall(prompt):
            numero = int(achado)
            if numero not in vistos:
                vistos.add(numero)
                ordenados.append(numero)
        return ordenados
```

> [!important] Ler os ids do prompt não é gambiarra — é o ponto
> O dublê **não recebe o payload**: ele extrai os ids do texto, exatamente como o modelo real faz para saber entre quem escolher. Se ele recebesse a lista pronta por outro caminho, o teste passaria mesmo com o `build_payload` gerando um prompt em que os ids não aparecem — e o Gemini receberia esse mesmo prompt quebrado no Bloco 4.

> [!success] `promover_indice` é o que torna a divergência testável
> Com o padrão `0`, o dublê sempre respeita a ordem da engine — que é o comportamento correto e o que a suíte inteira usa. Mas `FakeProvider(promover_indice=1)` promove o segundo candidato, e é assim que o Bloco 3 vai testar a **regra 5 do contrato C2** (principal fora do `conjunto_empate` cai em fallback) sem precisar de rede nem de sorte.

> [!warning] Cuidado com o `min()` na hora de copiar
> `min(self.promover_indice, len(ids) - 1)` existe para o dublê não estourar `IndexError` quando alguém pedir para promover o 5º de uma lista de 3. Sem essa linha, um teste mal calibrado derruba a suíte com erro que não tem nada a ver com o que ele queria provar.

---

## 3️⃣ F2-03 — configuração e `get_provider()`

### `quiz/llm/__init__.py`

```python
from django.conf import settings

from quiz.llm.base import LLMProvider, LLMTimeout, LLMUnavailable
from quiz.llm.fake import FakeProvider

__all__ = ["LLMProvider", "LLMTimeout", "LLMUnavailable", "FakeProvider", "get_provider"]


def get_provider():
    """Provider configurado, ou None quando a camada esta desligada."""
    if not settings.LLM_ENABLED:
        return None

    nome = settings.LLM_PROVIDER
    if nome == "fake":
        return FakeProvider()
    if nome == "gemini":
        # Import tardio de proposito: o SDK so existe a partir do Bloco 4.
        from quiz.llm.gemini import GeminiProvider

        return GeminiProvider()

    raise LLMUnavailable(f"LLM_PROVIDER desconhecido: {nome}")
```

> [!danger] O import do Gemini tem que ficar dentro da função
> Se ele subir para o topo do arquivo, `import quiz.llm` passa a exigir o SDK instalado — e a promessa de desenvolver a camada inteira offline morre na primeira linha. Pior: quebra na máquina de quem clonar o repositório sem rodar `pip install`, com um `ModuleNotFoundError` que não diz que a culpa é de um import mal colocado.

> [!note] `get_provider()` devolve `None`, não levanta exceção, quando está desligado
> `LLM_ENABLED=false` é estado normal e esperado — é o padrão do projeto e o modo do CI. Quem chama trata `None` como "não há camada de IA hoje" e segue com o resultado do motor. Transformar isso em exceção obrigaria um `try/except` em volta de cada chamada para expressar o caso mais comum.

### `config/settings.py` — no topo

```python
import os
from pathlib import Path

from dotenv import load_dotenv

BASE_DIR = Path(__file__).resolve().parent.parent
load_dotenv(BASE_DIR / ".env")
```

### `config/settings.py` — no **fim** do arquivo, depois do `RECOMMENDATION_LIMIT`

```python
# --- Camada de IA ---------------------------------------------------------
# LLM_ENABLED=false desliga tudo e o sistema volta a responder so com a engine.
# E o padrao, e e o modo em que a suite de testes e o CI rodam.
LLM_ENABLED = os.getenv("LLM_ENABLED", "false").lower() == "true"
LLM_PROVIDER = os.getenv("LLM_PROVIDER", "fake")
LLM_MODEL = os.getenv("LLM_MODEL", "gemini-2.0-flash")
LLM_API_KEY = os.getenv("LLM_API_KEY", "")
LLM_TIMEOUT = int(os.getenv("LLM_TIMEOUT", "8"))
LLM_PROMPT_VERSION = os.getenv("LLM_PROMPT_VERSION", "v1")
```

> [!important] O padrão é **desligado**, e isso é decisão de projeto
> Quem clonar o repositório e rodar a suíte não chama rede, não gasta cota e não precisa de chave. Ligar é uma linha no `.env` local. O contrário — padrão ligado, com todo mundo dependendo de `.env` correto para os testes passarem — é como um TCC começa a falhar na máquina do avaliador.

### `.env.example` — arquivo novo, **versionado**

```
LLM_ENABLED=true
LLM_PROVIDER=fake
LLM_MODEL=gemini-2.0-flash
LLM_API_KEY=
LLM_TIMEOUT=8
LLM_PROMPT_VERSION=v1
```

> [!warning] `.env.example` entra no Git; `.env` nunca
> O `.gitignore` já cobre o `.env` desde o primeiro dia. O `.example` existe para a próxima pessoa saber **quais** variáveis existem sem receber o valor de nenhuma — e é o que a [[git-fluxo-aplicado-tcc|nota de git]] chama de regra do segredo, que vale integralmente mesmo num TCC.

---

## 4️⃣ Os testes — `quiz/tests_llm.py`, arquivo novo

O runner do Django descobre qualquer `test*.py`, então o arquivo separado funciona e mantém o `quiz/tests.py` legível.

```python
import json
import sys

from django.test import SimpleTestCase, override_settings

from quiz.llm import FakeProvider, get_provider

PROMPT = """
"candidatos": [
  {"course_id": 7, "nome": "Eletricista Instalador"},
  {"course_id": 3, "nome": "Comandos Eletricos"},
  {"course_id": 11, "nome": "Costura Industrial"}
]
"""


class FakeProviderTest(SimpleTestCase):
    def test_devolve_json_valido_com_os_ids_do_prompt(self):
        saida = json.loads(FakeProvider().complete(PROMPT))
        ids = [saida["principal"]["course_id"]] + [
            a["course_id"] for a in saida["alternativas"]
        ]
        self.assertEqual(sorted(ids), [3, 7, 11])
        self.assertEqual(len(ids), len(set(ids)))

    def test_por_padrao_respeita_a_ordem_da_engine(self):
        saida = json.loads(FakeProvider().complete(PROMPT))
        self.assertEqual(saida["principal"]["course_id"], 7)

    def test_pode_forcar_divergencia_para_os_testes_do_bloco_3(self):
        saida = json.loads(FakeProvider(promover_indice=1).complete(PROMPT))
        self.assertEqual(saida["principal"]["course_id"], 3)
        self.assertEqual(len(saida["alternativas"]), 2)

    def test_textos_nao_sao_vazios(self):
        saida = json.loads(FakeProvider().complete(PROMPT))
        self.assertTrue(saida["principal"]["texto"].strip())
        self.assertTrue(all(a["texto"].strip() for a in saida["alternativas"]))


class GetProviderTest(SimpleTestCase):
    @override_settings(LLM_ENABLED=False)
    def test_desligado_nao_instancia_nada(self):
        """O botao de panico da defesa, testado em vez de presumido."""
        self.assertIsNone(get_provider())

    @override_settings(LLM_ENABLED=True, LLM_PROVIDER="fake")
    def test_ligado_devolve_o_fake(self):
        self.assertIsInstance(get_provider(), FakeProvider)

    def test_importar_o_pacote_nao_puxa_o_sdk_do_gemini(self):
        """A promessa de rodar offline vive nesta assercao."""
        self.assertNotIn("quiz.llm.gemini", sys.modules)
```

> [!success] O teste do `LLM_ENABLED=False` é o mais importante dos sete
> Ele é o **botão de pânico da defesa**: uma linha no `.env` desliga a camada inteira se a internet da instituição falhar cinco minutos antes da apresentação. Um botão de pânico que ninguém apertou desde que foi instalado não é mitigação, é esperança — e este teste é o que o aperta a cada `manage.py test`.

> [!question] O último teste é frágil de propósito, e vale entender por quê
> `assertNotIn("quiz.llm.gemini", sys.modules)` depende de nada mais ter importado o módulo antes na mesma execução. Se um dia o Bloco 4 acrescentar um teste que instancia o `GeminiProvider`, este aqui pode passar a falhar por ordem de execução. **Quando isso acontecer, não apague o teste** — mova-o para rodar isolado ou troque por uma checagem de import em subprocesso. A propriedade que ele defende (o pacote importa sem SDK) é uma das três que sustentam o cronograma.

---

## 5️⃣ Verificação

```bash
./.venv/Scripts/python.exe manage.py makemigrations --check --dry-run
```

**No changes detected** — este bloco não toca em modelo nenhum.

```bash
PYTHONIOENCODING=utf-8 ./.venv/Scripts/python.exe manage.py test
```

**28 testes verdes** — os 21 de antes mais os 7 novos.

E a prova real, que é o que dá sentido ao bloco inteiro:

```bash
PYTHONIOENCODING=utf-8 ./.venv/Scripts/python.exe manage.py shell
```

```python
import json
from quiz.models import QuizAttempt
from quiz.delivery import build_payload
from quiz.llm import FakeProvider

payload = build_payload(QuizAttempt.objects.first())
prompt = json.dumps(payload, ensure_ascii=False)      # o Bloco 3 monta isso direito
print(FakeProvider().complete(prompt))
```

O dublê tem que devolver JSON com os cinco `course_id` do payload, o principal sendo o `rank_engine: 1`. **Desligue o wi-fi antes de rodar** — se funcionar sem rede, o bloco cumpriu o que prometeu.

---

## 6️⃣ Commit e PR

```bash
git add -A && git commit -m "feat(ia): protocolo do provedor, FakeProvider e configuracao por env"
git push -u origin feat/ia-provider-offline
```

```markdown
## Toca contrato de outra frente? ( ) não (x) sim → toca o settings.py
Bloco novo no fim do arquivo (LLM_*), sem mexer no que ja existe.
Acrescenta python-dotenv ao requirements e um .env.example versionado.
Nada muda no comportamento do site: LLM_ENABLED e false por padrao.
```

> [!note] Avise que o `requirements.txt` mudou
> Quem atualizar precisa de `pip install -r requirements.txt` para o `python-dotenv`. Sem isso o `settings.py` quebra no import e **todo o projeto para** — inclusive para o colega da autenticação, que não tem nada a ver com a camada de IA. É o único efeito colateral deste bloco fora da sua trilha.

## ▶️ Próxima ação

**Bloco 3** — `feat/ia-prompt-e-validacao`: o `entrega_v1.md` em arquivo (com a **regra 2 reescrita** para falar em `conjunto_empate`, não em 0,05), o carregador de placeholders e o `DeliverySerializer` com as 5 regras.

## 📎 Veja também

- [[plano-execucao-f1-f2|🗂️ Plano de execução F1+F2]] · [[spec-motor-e-ia-frentes-1-2|🧭 Spec F1+F2]]
- [[passo-a-passo-f1-04-f1-03|🎚️ Passo a passo F1-04 e F1-03]] — o bloco anterior
- [[camada-ia-plano-implementacao|🧩 Plano da camada de IA]] · [[prompt-padrao-recomendacao|📝 Prompt v1]] · [[decisao-camada-ia|🤖 Decisão da camada de IA]]
- **Conceitos:** [[tst-mocks-e-dubles|Mocks e dublês]] · [[ia-llm-fundamentos|Fundamentos de LLM]] · [[ia-tokens-e-custo|Tokens, custo e latência]] · [[tst-testes-django|Testes em Django]]
- [[TCC|🎓 TCC]] · [[Projetos|🚀 Projetos]]
