---
title: "Passo a passo — F2-08 (GeminiProvider, cache, timeout e metadados)"
aliases: ["Passo a passo F2-08", "GeminiProvider", "Cache da camada de IA", "Timeout do Gemini"]
tags: [tcc, ia, llm, gemini, django, cache, execucao, passo-a-passo]
status: em-andamento
projeto: TCC
criado: 2026-08-27
---

> [!info] Spec: [[spec-motor-e-ia-frentes-1-2|🧭 Spec das Frentes 1 e 2]] · Plano: [[camada-ia-plano-implementacao|🧩 Camada de IA]] · Decisão: [[decisao-camada-ia|🤖 Decisão da camada de IA]] · Custo: [[ia-tokens-e-custo|💸 Tokens e custo]]
> **Vem depois de:** [[passo-a-passo-f2-07|🎛️ F2-07]] e [[passo-a-passo-f2-09|🔌 F2-09]] · **Habilita:** F2-10 (experimento)

# 🌐 Passo a passo — F2-08

> [!abstract] O que esta nota é
> O **como** da única tarefa da Frente 2 que toca a rede: o provedor real do Gemini, com timeout, cache e os metadados que o experimento do Passo 11 vai medir.
> Escrito contra o repositório no commit `1b4f48e`, **assumindo F2-07 e F2-09 aplicadas**.

> [!warning] Nada disto foi executado
> Código escrito contra a leitura do repositório, não rodado. E, diferente das outras duas notas, **um trecho aqui não é verificável offline**: a chamada ao SDK do Gemini (seção 2️⃣) depende da versão instalada do pacote. Ela está isolada num arquivo só, de propósito.

> [!danger] Esta é a tarefa bloqueada — confirme antes de começar
> A pergunta 3 do kickoff (*quem guarda a credencial da API*) continua **em aberto** na [[decisao-camada-ia|🤖 nota de decisão]]. Sem `LLM_API_KEY`, as seções 1️⃣ a 3️⃣ até rodam (o provedor recusa de forma limpa e o `deliver()` cai em fallback), mas a verificação da seção 6️⃣ não fecha. Se a chave ainda não saiu, **faça a F1-07 ou a F1-05 e volte aqui depois** — é exatamente o contorno que a spec desenhou para a semana 3.

---

## 🚦 Antes de abrir o editor

```bash
git checkout -b feat/ia-gemini
```

```bash
./.venv/Scripts/python.exe -m pip install google-genai
```

E fixe a versão que caiu na sua máquina — não invente o número:

```bash
./.venv/Scripts/python.exe -m pip freeze | grep -i genai >> requirements.txt
```

> [!warning] A chave nunca entra no repositório
> Ela vive no `.env`, que já está no `.gitignore`. O `.env.example` ganha a linha com o valor **vazio**. Se a chave vazar num commit, não basta apagar no commit seguinte: ela fica no histórico, e o certo passa a ser revogar no console do Google e gerar outra.

---

## 1️⃣ Cache de verdade no settings

### `config/settings.py` — junto do bloco `LLM_*`

```python
LLM_CACHE_TTL = int(os.getenv("LLM_CACHE_TTL", "604800"))  # 7 dias

CACHES = {
    "default": {
        "BACKEND": "django.core.cache.backends.filebased.FileBasedCache",
        "LOCATION": BASE_DIR / ".cache",
    }
}
```

E no `.gitignore`:

```
.cache/
```

> [!important] Por que arquivo e não o `LocMemCache` padrão
> O cache em memória do Django **morre quando o processo morre**. Você popularia os perfis de demonstração na véspera e chegaria na defesa com o cache vazio, fazendo chamada real de rede na frente da banca — que é precisamente a cena que o cache existe para evitar. Cache em arquivo sobrevive a `Ctrl+C`, a reinício de máquina e a `runserver` recarregando sozinho.

> [!note] O que o cache resolve que `temperature=0` não resolve
> `temperature=0` reduz a variação, não a elimina, e o provedor pode atualizar o modelo por baixo sem avisar. O que garante à banca que a mesma tentativa dá a mesma resposta é o **cache**: mesma entrada → mesma saída, servida do disco, sem nova chamada. É também o que sustenta o orçamento de R$ 0 — [[ia-tokens-e-custo|💸 tokens e custo]].

---

## 2️⃣ `quiz/llm/gemini.py`

Arquivo novo. É o único do projeto que sabe que o Gemini existe.

```python
"""Provedor real. Importado so por get_provider(), nunca no topo de um pacote:
a camada inteira tem que continuar rodando sem este SDK instalado."""

from django.conf import settings

from quiz.llm.base import LLMTimeout, LLMUnavailable


class GeminiProvider:
    def __init__(self, client=None):
        self.ultimo_uso = {}
        if client is not None:  # injecao usada pelos testes
            self._client = client
            return

        if not settings.LLM_API_KEY:
            raise LLMUnavailable("LLM_API_KEY vazia: configure o .env ou desligue a camada.")
        try:
            from google import genai
        except ImportError as erro:
            raise LLMUnavailable("SDK google-genai nao instalado.") from erro

        self._client = genai.Client(api_key=settings.LLM_API_KEY)

    def complete(self, prompt: str) -> str:
        try:
            resposta = self._client.models.generate_content(
                model=settings.LLM_MODEL,
                contents=prompt,
                config={
                    "temperature": 0,
                    "http_options": {"timeout": settings.LLM_TIMEOUT * 1000},
                },
            )
        except Exception as erro:
            if _parece_timeout(erro):
                raise LLMTimeout(str(erro)) from erro
            raise LLMUnavailable(str(erro)) from erro

        uso = getattr(resposta, "usage_metadata", None)
        self.ultimo_uso = {
            "tokens_in": getattr(uso, "prompt_token_count", None),
            "tokens_out": getattr(uso, "candidates_token_count", None),
        }
        return resposta.text or ""


def _parece_timeout(erro):
    if isinstance(erro, TimeoutError):
        return True
    return any(p in str(erro).lower() for p in ("timeout", "timed out", "deadline"))
```

> [!warning] Este é o trecho que você tem que conferir contra o SDK instalado
> `genai.Client(...)`, `models.generate_content(...)`, `http_options.timeout` em **milissegundos** e `usage_metadata` são a API do pacote `google-genai`. Se a versão que o `pip` instalou tiver assinatura diferente, é aqui — e **só** aqui — que muda. Confira antes de rodar:
> ```bash
> ./.venv/Scripts/python.exe -c "from google import genai; help(genai.Client)"
> ```

> [!note] `except Exception` aqui, e não no `deliver()`
> Parece contradizer a regra da [[passo-a-passo-f2-07|F2-07]], mas é o oposto dela. Aqui o trabalho do bloco é **traduzir**: pegar qualquer coisa que um SDK de terceiro invente e converter para as duas exceções do nosso protocolo. É a fronteira do sistema, e é exatamente onde a captura larga tem que ficar — para que o `deliver()` possa continuar sendo estreito.

> [!tip] Classificar timeout por mensagem é aproximado, e tudo bem
> O `deliver()` trata `LLMTimeout` e `LLMUnavailable` **igual**: os dois viram fallback. Errar a classificação não muda comportamento nenhum, só troca um rótulo numa coluna do CSV do experimento. Precisão que não muda decisão não vale um `import` do módulo de erros do SDK aqui dentro — e esse import quebraria a promessa de o arquivo ser trocável.

> [!success] `ultimo_uso` mantém o protocolo com um método só
> `LLMProvider` promete `complete(prompt) -> str` e nada mais. Se os tokens virassem valor de retorno, todo provedor — inclusive o `FakeProvider` — teria que devolver tupla. Deixar o consumo num atributo lido com `getattr` faz o `deliver()` coletar quando existe e ignorar quando não existe. O protocolo continua de um método; o experimento continua tendo os números.

---

## 3️⃣ Cache e tokens no `deliver()`

### `quiz/delivery.py` — no topo

```python
import hashlib

from django.core.cache import cache
```

### Uma função de chave, antes do `deliver`

```python
def chave_cache(prompt):
    digest = hashlib.sha256(prompt.encode("utf-8")).hexdigest()[:32]
    return f"entrega:{settings.LLM_PROMPT_VERSION}:{settings.LLM_MODEL}:{digest}"
```

### O miolo do `deliver`, no lugar do `try` de hoje

```python
    prompt = montar(payload)
    chave = chave_cache(prompt)
    cru = cache.get(chave)
    veio_do_cache = cru is not None

    inicio = time.monotonic()
    try:
        if not veio_do_cache:
            cru = provider.complete(prompt)
        dados = validar_entrega(cru, payload)
    except (LLMTimeout, LLMUnavailable, EntregaInvalida):
        cache.delete(chave)
        return _fallback(attempt, motivo="provedor ou contrato")

    if not veio_do_cache:
        cache.set(chave, cru, settings.LLM_CACHE_TTL)

    attempt.cache_hit = veio_do_cache
    uso = getattr(provider, "ultimo_uso", {}) or {}
    attempt.tokens_in = uso.get("tokens_in")
    attempt.tokens_out = uso.get("tokens_out")
    return _aplicar(attempt, dados, latency_ms=int((time.monotonic() - inicio) * 1000))
```

E acrescente os três campos ao `update_fields` do `_aplicar`:

```python
    attempt.save(
        update_fields=[
            "diverged", "used_fallback", "llm_model", "prompt_version", "latency_ms",
            "cache_hit", "tokens_in", "tokens_out",
        ]
    )
```

> [!important] O cache guarda o texto **cru**, não o validado
> Guardar o resultado já validado pareceria mais econômico e seria um erro: no dia em que uma regra do C2 mudar, o cache continuaria servindo saídas que a regra nova rejeitaria — e você teria um bug que só aparece em máquina com cache quente. Guardando o cru, **a validação roda sempre**, inclusive em cache hit. O que se economiza é a rede, que é o caro; a validação é microssegundos.

> [!note] `cache.delete` no caminho de erro
> Se a saída em cache não passa mais na validação (regra nova, prompt novo), ela é lixo. Apagar na hora faz a próxima tentativa chamar a rede em vez de repetir o mesmo fallback para sempre.

> [!warning] A chave é o hash do **prompt montado**, não das respostas
> O prompt já contém perfil, respostas, confiança e candidatos — e muda sozinho quando qualquer um deles muda. Hashear só as respostas deixaria duas tentativas com o mesmo quiz e catálogos diferentes dividindo a mesma entrada de cache, o que é errado de um jeito silencioso. Versão do prompt e modelo entram no prefixo porque mudar qualquer um dos dois **invalida tudo** — e prefixo legível deixa você conferir isso com `ls .cache/`.

### `.env.example`

```
LLM_CACHE_TTL=604800
```

---

## 4️⃣ O cache quebra os testes da F2-07 — conserte junto

> [!danger] Leia antes de rodar a suíte
> `DeliverTest` tem dois testes que usam o **mesmo perfil elétrico**. Mesmo perfil → mesmo payload → mesmo prompt → **mesma chave de cache**. Sem limpeza, o segundo teste recebe a saída que o primeiro gravou, e um deles falha de um jeito que não tem nada a ver com o que ele testa. É o preço de um cache global, e ele se paga em duas linhas.

Em `quiz/tests_llm.py`, na `DeliverTest`:

```python
from django.core.cache import cache


@override_settings(
    LLM_ENABLED=True,
    LLM_PROVIDER="fake",
    CACHES={"default": {"BACKEND": "django.core.cache.backends.locmem.LocMemCache"}},
)
class DeliverTest(TestCase):
    def setUp(self):
        cache.clear()
```

> [!tip] `LocMemCache` nos testes, arquivo em desenvolvimento
> Teste não pode escrever no `.cache/` da sua máquina: ele passaria a depender do que sobrou da última execução, e um dia passaria por engano. Memória, limpa no `setUp`, é o único cache honesto numa suíte.

---

## 5️⃣ Os testes novos

Ainda em `quiz/tests_llm.py`:

```python
class GeminiProviderTest(SimpleTestCase):
    """Tudo aqui roda sem SDK, sem chave e sem rede."""

    @override_settings(LLM_API_KEY="")
    def test_sem_credencial_recusa_antes_de_tocar_no_sdk(self):
        from quiz.llm.gemini import GeminiProvider

        with self.assertRaises(LLMUnavailable):
            GeminiProvider()

    def test_erro_do_sdk_vira_llm_unavailable(self):
        from quiz.llm.gemini import GeminiProvider

        class ClienteQuebrado:
            class models:
                @staticmethod
                def generate_content(**kwargs):
                    raise RuntimeError("429 quota exceeded")

        with self.assertRaises(LLMUnavailable):
            GeminiProvider(client=ClienteQuebrado()).complete("prompt")

    def test_estouro_de_prazo_vira_llm_timeout(self):
        from quiz.llm.gemini import GeminiProvider

        class ClienteLento:
            class models:
                @staticmethod
                def generate_content(**kwargs):
                    raise TimeoutError("deadline exceeded")

        with self.assertRaises(LLMTimeout):
            GeminiProvider(client=ClienteLento()).complete("prompt")


class CacheDaEntregaTest(TestCase):
    """O cache visto pelo deliver(), com o FakeProvider contando chamadas."""

    ELETRICO = ("Montar e ligar", "Chão de fábrica", "Eletricidade", "alicate")

    @classmethod
    def setUpTestData(cls):
        silencio = StringIO()
        for comando in ("seed_areas", "seed_courses", "seed_questions"):
            call_command(comando, stdout=silencio)

    def setUp(self):
        cache.clear()

    def tentativa(self):
        attempt = QuizAttempt.objects.create(respondent_name="teste")
        for trecho in self.ELETRICO:
            choice = Choice.objects.get(text__icontains=trecho)
            Answer.objects.create(attempt=attempt, question=choice.question, choice=choice)
        recommend(attempt)
        attempt.refresh_from_db()
        return attempt

    @override_settings(
        LLM_ENABLED=True,
        LLM_PROVIDER="fake",
        CACHES={"default": {"BACKEND": "django.core.cache.backends.locmem.LocMemCache"}},
    )
    def test_segundo_perfil_igual_sai_do_cache_sem_nova_chamada(self):
        contador = FakeProvider()
        chamadas = []
        original = contador.complete
        contador.complete = lambda prompt: (chamadas.append(prompt), original(prompt))[1]

        with mock.patch("quiz.delivery.get_provider", return_value=contador):
            primeira = self.tentativa()
            deliver(primeira)
            segunda = self.tentativa()
            deliver(segunda)

        primeira.refresh_from_db()
        segunda.refresh_from_db()
        self.assertEqual(len(chamadas), 1)
        self.assertFalse(primeira.cache_hit)
        self.assertTrue(segunda.cache_hit)

    @override_settings(LLM_PROMPT_VERSION="v1", LLM_MODEL="gemini-2.0-flash")
    def test_versao_do_prompt_muda_a_chave(self):
        v1 = chave_cache("mesmo prompt")
        with override_settings(LLM_PROMPT_VERSION="v2"):
            v2 = chave_cache("mesmo prompt")
        self.assertNotEqual(v1, v2)
```

Imports novos no topo: `from quiz.delivery import chave_cache` e `from quiz.llm.base import LLMTimeout, LLMUnavailable`.

> [!note] O teste do cache é sobre **não chamar**, não sobre o texto
> A asserção que importa é `len(chamadas) == 1`. Comparar os dois textos não provaria nada: o `FakeProvider` é determinístico e devolveria o mesmo texto com ou sem cache. O que se quer garantir é que a **rede não foi tocada** — e a única evidência disso é a contagem de chamadas.

> [!warning] Duas tentativas diferentes compartilhando cache é o comportamento desejado
> Pode assustar: a `segunda` tentativa recebe texto gerado para a `primeira`. Mas o prompt é idêntico — mesmo perfil, mesmas respostas, mesmos candidatos, mesma confiança. Se o texto não servisse para as duas, o problema estaria no prompt, não no cache. É essa propriedade que faz os perfis de demonstração ficarem instantâneos na defesa.

---

## 6️⃣ Verificação

```bash
PYTHONIOENCODING=utf-8 ./.venv/Scripts/python.exe manage.py test
```

**55 testes verdes** — os 50 da [[passo-a-passo-f2-09|F2-09]] mais os 5 novos.

> [!important] Agora a parte que só a chave prova
> `.env` com `LLM_ENABLED=true`, `LLM_PROVIDER=gemini` e a chave preenchida:
> ```bash
> ./.venv/Scripts/python.exe manage.py runserver 8010
> ```
> Responda o quiz. Depois, no `/admin`, abra a tentativa: `llm_model` preenchido, `latency_ms` com valor plausível (centenas a poucos milhares), `tokens_in`/`tokens_out` diferentes de nulo, `used_fallback=False`. **Responda o mesmo quiz de novo, com as mesmas alternativas:** a segunda tentativa tem que sair com `cache_hit=True` e `latency_ms` de uma ordem de grandeza menor. Esse par de tentativas é o critério de pronto da F2-08 na spec — e vale um print para o capítulo de resultados.

E o teste que ninguém gosta de fazer, mas é o que a banca vale:

> [!tip] Arranque o cabo de rede com a IA ligada
> Desligue o Wi-Fi e responda o quiz com `LLM_PROVIDER=gemini`. A página **tem que abrir**, com a recomendação do motor e `used_fallback=True` no admin. Se ela demorar mais que os 8 segundos do `LLM_TIMEOUT` ou devolver 500, o timeout não está sendo respeitado pelo SDK — e é melhor descobrir isso agora do que no auditório.

---

## 7️⃣ Commit

```bash
git add -A && git commit -m "feat(ia): provedor gemini com timeout, cache em arquivo e metadados de uso"
```

Confirme que a chave não foi junto:

```bash
git show --stat HEAD && git diff HEAD~1 -- .env.example
```

## 📎 Veja também

- [[passo-a-passo-f2-07|🎛️ F2-07 — orquestração da entrega]] · [[passo-a-passo-f2-09|🔌 F2-09 — endpoint C3]]
- [[spec-motor-e-ia-frentes-1-2|🧭 Spec das Frentes 1 e 2]] · [[camada-ia-plano-implementacao|🧩 Plano da camada de IA]] · [[decisao-camada-ia|🤖 Decisão da camada de IA]]
- **Conceitos:** [[ia-tokens-e-custo|Tokens, custo e latência]] · [[ia-llm-fundamentos|Fundamentos de LLM]] · [[ia-avaliacao-de-llm|Avaliação de LLM]]
- [[defesa-monografia-tcc|🎤 Defesa]] — onde o par de tentativas com `cache_hit` vira evidência
- [[TCC|🎓 TCC]] · [[Projetos|🚀 Projetos]]
