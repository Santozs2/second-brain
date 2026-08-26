---
title: "Ferramenta — manage.py ver_payload (inspecionar o que a LLM recebe)"
aliases: ["ver_payload", "Comando ver_payload", "Inspecionar payload", "Demo do payload"]
tags: [tcc, ia, django, ferramenta, execucao, passo-a-passo, defesa]
status: em-andamento
projeto: TCC
criado: 2026-08-26
---

> [!info] Plano: [[plano-execucao-f1-f2|🗂️ Plano de execução F1+F2]] · Contrato C1: [[spec-motor-e-ia-frentes-1-2|🧭 Spec F1+F2]] · Bloco 2: [[passo-a-passo-f2-01-f2-03|🔌 F2-01 a F2-03]] · Engine: [[engine-matching-cosseno|🧮 Engine]]

# 🔍 Ferramenta — `manage.py ver_payload`

> [!abstract] O que é e por que existe
> Um comando que imprime **exatamente o que a camada de IA recebe**: o payload do contrato C1 e, opcionalmente, a resposta do provedor. No mesmo molde do `test_engine` que já existe — cria a tentativa, mostra o resultado e desfaz tudo com rollback.
> Não está no backlog da spec. É ferramenta de apoio, e se paga três vezes: nos Blocos 3 e 4 (depurar prompt e validação), no Bloco 5 (gerar amostra para o experimento) e **na defesa**, quando alguém pedir para ver o que o modelo enxerga.

> [!warning] Nada disto foi executado
> Escrito contra o repositório no commit da branch `feat/ia-provider-offline`. A seção 4 é a verificação.

---

## 1️⃣ Antes: tirar os perfis de dentro do comando

Os quatro perfis de demonstração hoje moram dentro de `quiz/management/commands/test_engine.py`. O comando novo precisa dos mesmos — e importar de dentro de um módulo de comando é o tipo de acoplamento que ninguém espera encontrar.

### `quiz/perfis_demo.py` — arquivo novo

```python
"""Perfis sinteticos usados pelos comandos de demonstracao.

Cada valor e a lista de trechos de texto das alternativas a marcar.
Sao os mesmos quatro criterios de aceite descritos na nota da engine.
"""

PERFIS = {
    "Puro eletrico": ["Montar e ligar", "Chão de fábrica", "Eletricidade", "alicate"],
    "Automotivo + eletrico": ["Desmontar máquinas", "Oficina", "Eletricidade", "diagnóstico"],
    "Costura": ["próprias mãos", "Ateliê", "Tecidos", "costura industrial"],
    "TI e dados": ["Resolver problemas", "home office", "Dados, lógica", "terminal"],
}
```

Em `test_engine.py`, apague o dicionário e troque por:

```python
from quiz.perfis_demo import PERFIS
```

> [!note] Por que os testes **não** vão importar daqui
> `quiz/tests.py` repete esses mesmos trechos inline, e à primeira vista isso parece a terceira cópia a eliminar. Não é: teste que lê a própria entrada de um arquivo de configuração compartilhado fica mais difícil de ler e passa a falhar quando alguém mexe na configuração por outro motivo. **Comando de demonstração compartilha dado; teste declara o dele.** A duplicação entre teste e demo é intencional.

---

## 2️⃣ O comando

### `quiz/management/commands/ver_payload.py` — arquivo novo

```python
# quiz/management/commands/ver_payload.py
"""Imprime o payload do contrato C1 e a resposta do provedor, sem gravar nada."""

import json

from django.core.management.base import BaseCommand, CommandError
from django.db import transaction

from quiz.delivery import build_payload
from quiz.engine import recommend
from quiz.llm import FakeProvider, get_provider
from quiz.models import Answer, Choice, QuizAttempt
from quiz.perfis_demo import PERFIS


class Command(BaseCommand):
    help = "Mostra o que a camada de IA recebe. Nao grava nada (rollback no final)."

    def add_arguments(self, parser):
        parser.add_argument(
            "--perfil",
            default="Puro eletrico",
            help=f"Perfil sintetico. Opcoes: {' | '.join(PERFIS)}",
        )
        parser.add_argument(
            "--attempt",
            type=int,
            help="Inspeciona uma tentativa real ja gravada, em vez de um perfil sintetico.",
        )
        parser.add_argument(
            "--json",
            action="store_true",
            help="So o payload, sem cabecalho, para redirecionar para arquivo.",
        )

    def handle(self, *args, **options):
        if options["attempt"]:
            self._tentativa_real(options)
        else:
            self._perfil_sintetico(options)

    # ------------------------------------------------------------------ #

    def _tentativa_real(self, options):
        """Somente leitura: a tentativa ja existe e ja foi calculada."""
        try:
            attempt = QuizAttempt.objects.get(pk=options["attempt"])
        except QuizAttempt.DoesNotExist:
            raise CommandError(f"Tentativa {options['attempt']} nao existe.")
        if not attempt.recommendations.exists():
            raise CommandError(
                f"Tentativa {attempt.pk} nao tem recomendacoes. "
                "Ela foi criada antes da engine rodar?"
            )
        self._imprimir(attempt, options, titulo=f"tentativa #{attempt.pk}")

    def _perfil_sintetico(self, options):
        nome = options["perfil"]
        if nome not in PERFIS:
            raise CommandError(f"Perfil '{nome}'. Opcoes: {' | '.join(PERFIS)}")

        with transaction.atomic():
            attempt = QuizAttempt.objects.create(respondent_name=f"[demo] {nome}")
            for trecho in PERFIS[nome]:
                encontradas = list(Choice.objects.filter(text__icontains=trecho))
                if len(encontradas) != 1:
                    raise CommandError(
                        f"Trecho '{trecho}' casou com {len(encontradas)} alternativas "
                        "(esperado 1). O catalogo mudou? Ajuste quiz/perfis_demo.py."
                    )
                choice = encontradas[0]
                Answer.objects.create(
                    attempt=attempt, question=choice.question, choice=choice
                )

            recommend(attempt)
            attempt.refresh_from_db()
            self._imprimir(attempt, options, titulo=nome)

            transaction.set_rollback(True)  # nao suja o banco

    # ------------------------------------------------------------------ #

    def _imprimir(self, attempt, options, titulo):
        payload = build_payload(attempt)

        if options["json"]:
            self.stdout.write(json.dumps(payload, indent=2, ensure_ascii=False))
            return

        self.stdout.write(self.style.WARNING(f"\n=== {titulo} ==="))
        self.stdout.write(json.dumps(payload, indent=2, ensure_ascii=False))

        confianca = payload["confianca"]
        self.stdout.write(
            self.style.WARNING(
                f"\n--- quem escolhe: {confianca['quem_escolhe']} "
                f"| banda: {confianca['banda']} "
                f"| empate: {confianca['conjunto_empate']} ---"
            )
        )

        provider = get_provider() or FakeProvider()
        prompt = json.dumps(payload, ensure_ascii=False)
        resposta = json.loads(provider.complete(prompt))

        self.stdout.write(self.style.WARNING("\n=== resposta do provedor ==="))
        self.stdout.write(json.dumps(resposta, indent=2, ensure_ascii=False))

        principal = resposta["principal"]["course_id"]
        dentro = principal in confianca["conjunto_empate"]
        marca = self.style.SUCCESS("OK") if dentro else self.style.ERROR("FORA")
        self.stdout.write(f"\nprincipal {principal} no conjunto de empate: {marca}")
```

> [!important] `get_provider() or FakeProvider()` — o comando funciona com a IA desligada
> `LLM_ENABLED` é `false` por padrão, e a ferramenta precisa servir exatamente nesse estado, que é o do dia a dia. Com a flag ligada, ela passa a mostrar a resposta **do provedor real** sem mudar nada no código — e é assim que ela vira a ferramenta de depuração do Bloco 4.

> [!success] A última linha é o que faz a ferramenta valer o esforço
> Ela verifica, a olho nu, a **regra 5 do contrato C2**: o principal escolhido está dentro do `conjunto_empate`? Com o `FakeProvider` sempre estará. Com o Gemini de verdade, no Bloco 4, é essa linha que vai mostrar quando o modelo saiu do combinado — antes de o validador rejeitar e cair em fallback silencioso. **Ver a violação é diferente de contar a violação.**

> [!warning] `--attempt` não recalcula, e isso é proposital
> Inspecionar uma tentativa real tem que mostrar o que **está gravado**, não o que a engine calcularia hoje. Se o comando rodasse `recommend()` de novo, ele apagaria e recriaria as recomendações — e a tentativa que você queria auditar deixaria de ser a que a pessoa viu. É a diferença entre inspecionar e adulterar a prova.

---

## 3️⃣ Um teste

Em `quiz/tests.py`, dentro de `RecomendacaoTest`:

```python
    def test_comando_ver_payload_roda_e_nao_suja_o_banco(self):
        antes = QuizAttempt.objects.count()
        saida = StringIO()
        call_command("ver_payload", perfil="Costura", stdout=saida)
        self.assertIn("conjunto_empate", saida.getvalue())
        self.assertEqual(QuizAttempt.objects.count(), antes)
```

> [!note] A segunda asserção é a que importa
> Que o comando imprime alguma coisa, você vê rodando. Que ele **não deixa lixo no banco** é o tipo de regressão que só aparece três semanas depois, quando alguém abre o `/admin` e encontra quarenta tentativas chamadas `[demo] Costura` no meio dos dados do piloto. O `set_rollback(True)` é uma linha fácil de perder num refactor.

---

## 4️⃣ Verificação

```bash
PYTHONIOENCODING=utf-8 ./.venv/Scripts/python.exe manage.py ver_payload
PYTHONIOENCODING=utf-8 ./.venv/Scripts/python.exe manage.py ver_payload --perfil "TI e dados"
PYTHONIOENCODING=utf-8 ./.venv/Scripts/python.exe manage.py ver_payload --json > payload-eletrico.json
```

> [!danger] O comando precisa do banco populado
> Ele lê `Choice` por trecho de texto. Num clone novo, o `db.sqlite3` não vem junto (está no `.gitignore`) e o comando falha com "casou com 0 alternativas" — que parece bug e é banco vazio. Antes de tudo:
> ```bash
> python manage.py seed_areas && python manage.py seed_courses && python manage.py seed_questions
> ```

E a suíte inteira:

```bash
PYTHONIOENCODING=utf-8 ./.venv/Scripts/python.exe manage.py test
```

**29 testes verdes** — os 28 do Bloco 2 mais o novo.

---

## 5️⃣ Onde isto se paga

| Momento | Para quê |
|---|---|
| **Bloco 3** | Ver o texto do prompt renderizado antes de escrever o validador — se o payload estiver estranho aqui, estará estranho no prompt |
| **Bloco 4** | Primeira chamada real ao Gemini: comparar a resposta dele com a do dublê no mesmo perfil, lado a lado |
| **Bloco 5** | `--json` gera as amostras do experimento e o **anexo da monografia** com um payload real |
| **Defesa** | Quando a banca perguntar *"o que exatamente vocês mandam para a IA?"*, a resposta é um comando, não um slide |

> [!tip] O `--json` redirecionado é o anexo pronto
> `ver_payload --json > payload-eletrico.json` produz o arquivo que entra como anexo ao lado do `entrega_v1.md`. Os dois juntos — a entrada e o prompt — são o que permite a alguém reproduzir o trabalho sem acesso ao sistema, que é o critério de reprodutibilidade que a metodologia cobra.

## 📎 Veja também

- [[passo-a-passo-f2-01-f2-03|🔌 Passo a passo F2-01 a F2-03]] — o `FakeProvider` que este comando usa
- [[passo-a-passo-f1-04-f1-03|🎚️ Passo a passo F1-04 e F1-03]] — o `build_payload` que ele imprime
- [[spec-motor-e-ia-frentes-1-2|🧭 Spec F1+F2]] — o contrato C1 e a regra 5 do C2
- [[engine-matching-cosseno|🧮 Engine de matching]] · [[prompt-padrao-recomendacao|📝 Prompt v1]] · [[defesa-monografia-tcc|🎤 Defesa e monografia]]
- **Conceitos:** [[ia-avaliacao-de-llm|Avaliação de LLM]] · [[tst-testes-django|Testes em Django]]
- [[TCC|🎓 TCC]] · [[Projetos|🚀 Projetos]]
