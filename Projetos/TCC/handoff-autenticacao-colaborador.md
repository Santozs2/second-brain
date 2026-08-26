---
title: "Handoff — trilha de autenticação para outro colaborador (AUT-01 a AUT-12)"
aliases: ["Handoff autenticação", "Trilha AUT", "Ordem de serviço da autenticação", "Autenticação para colaborador"]
tags: [tcc, autenticacao, handoff, git, execucao, passo-a-passo, django]
status: em-andamento
projeto: TCC
criado: 2026-08-25
---

> [!info] Spec: [[spec-autenticacao-lista-interesse|🔐 Spec de autenticação e lista de interesse]] · Execução AUT-01→03: [[passo-a-passo-aut-01-aut-03|🔑 Passo a passo]] · Git: [[git-fluxo-aplicado-tcc|🎓 Fluxo de Git do TCC]] · Divisão: [[divisao-de-trabalho-tcc|👥 4 frentes]]

# 🤝 Handoff — trilha de autenticação

> [!abstract] Para quem é esta nota
> Para o **colaborador que vai assumir a autenticação inteira**, do zero até a lista de interesse funcionando. Ela é auto-suficiente: você não precisa ter participado das reuniões anteriores nem ler as specs do motor.
> Leia as três primeiras seções antes de escrever qualquer código. Depois disso, é seguir os blocos na ordem.
> Repositório auditado em **2026-08-25**, sobre o commit `1068eb0`.

> [!success] O tamanho real do trabalho
> `django.contrib.auth` já está ligado no projeto — sessão, hash de senha, validadores, `login_required`, `{{ user.is_authenticated }}` nos templates. **Nada disso precisa ser escrito.** Sua trilha é: um app, um modelo de usuário, um modelo de interesse, cinco telas e as regras de quem-vê-o-quê. São doze tarefas, cinco entregas, e nenhuma delas depende de internet, API paga ou dado que outra pessoa precise te entregar.

---

## 1️⃣ O que você recebe pronto

| Peça | Estado |
|---|---|
| Projeto Django 5.2 + DRF rodando | ✅ |
| Quiz, engine de recomendação, tela de resultado | ✅ funcionando fim a fim |
| `django.contrib.auth`, sessão, middlewares, validadores de senha | ✅ ligados desde o dia 1 |
| Suíte de testes | ✅ **14 verdes** — a sua trilha leva para ~25 |
| Usuários no banco | **0 registros** — é o que torna a AUT-01 barata |
| App `accounts`, telas de acesso, modelo de interesse | ❌ não existem — é o seu trabalho |
| `LOGIN_URL`, `LOGIN_REDIRECT_URL` | ❌ não definidos |

> [!warning] Uma coisa que está quebrada e você vai consertar
> `/resultado/7/` abre para **qualquer pessoa que digite o número na barra de endereços**. Hoje isso vaza no máximo "alguém tirou 96% em Eletricista". No minuto em que a tentativa ganha dono — que é a sua AUT-04 —, vira dado pessoal ligado a uma pessoa identificada. Por isso AUT-04 e AUT-05 são **uma entrega só** e não podem ser separadas.

---

## 2️⃣ Setup — cinco minutos

```bash
git clone https://github.com/Santozs2/tcc-train.git
cd tcc-train
python -m venv .venv
./.venv/Scripts/python.exe -m pip install -r requirements.txt
./.venv/Scripts/python.exe manage.py migrate
./.venv/Scripts/python.exe manage.py seed_areas
./.venv/Scripts/python.exe manage.py seed_courses
./.venv/Scripts/python.exe manage.py seed_questions
PYTHONIOENCODING=utf-8 ./.venv/Scripts/python.exe manage.py test
```

**14 testes verdes** antes de você tocar em qualquer coisa. Se não estiverem, o problema não é seu — avise o scrum master antes de começar.

> [!note] `PYTHONIOENCODING=utf-8` não é frescura
> Os nomes de curso têm acento e o console do Windows quebra sem isso. Vale para `manage.py test` e para `manage.py test_engine`.

---

## 3️⃣ Git — como você entrega

> [!important] O projeto usa **GitHub Flow**, não Git Flow
> Está decidido e documentado em [[git-fluxo-aplicado-tcc|🎓 Fluxo de Git do TCC]]: só `main` e branches curtas de tarefa. **Não crie `develop`, `release/*` nem `hotfix/*`** — o TCC não publica versões, ele demonstra a `main` na banca.

### A branch

Você trabalha numa branch só, sempre com o mesmo nome:

```bash
git checkout main
git pull
git checkout -b feat/autenticacao
```

Ao final de cada bloco: abre PR, mescla, e **recria a branch com o mesmo nome** a partir da `main` atualizada:

```bash
git checkout main && git pull
git branch -d feat/autenticacao
git checkout -b feat/autenticacao
```

> [!tip] Por que recriar em vez de manter uma branch longa
> Você pediu uma branch só para autenticação, e é isso que este fluxo dá — o nome é sempre `feat/autenticacao` e ninguém mais mexe nele. Mas mantê-la aberta por duas semanas acumularia conflito com o motor, que está mexendo em `quiz/models.py` na mesma janela. Recriar depois de cada merge custa três comandos e mantém a regra de branch curta que o grupo combinou.

### Os commits

[[git-commits-e-mensagens|Conventional Commits]], escopo `contas`:

```
feat(contas): usuario com login por e-mail e telas de acesso
feat(contas): vincula tentativa ao usuario e fecha o resultado
fix(contas): logout por POST no header
test(contas): cobre claim de tentativa anonima
```

### O PR

Template do grupo (`.github/pull_request_template.md`):

```markdown
## O que muda
## Por quê
## Como testar
## Toca contrato de outra frente? ( ) não ( ) sim → qual
```

> [!warning] Na sua trilha, a última linha é **sim** duas vezes
> **Bloco 2 (AUT-04/05)** mexe em `QuizAttempt`, que é o modelo da Frente 1 e o que a Frente 4 consome. **Bloco 4 (AUT-08)** mexe em `templates/quiz/result.html`, que é tela da Frente 4. Nesses dois, a regra do grupo exige aprovação **das duas pontas** — a sua e a da frente dona do arquivo. Marque na descrição e chame a pessoa; não mescle com uma aprovação só.

### Quem revisa

O anel de revisão da [[git-fluxo-aplicado-tcc|nota de git]] cobre as quatro frentes e a autenticação não é nenhuma delas. Regra para a sua trilha:

- **Revisor padrão:** o scrum master (dono das Frentes 1 e 2)
- **Blocos 2 e 4:** também a Frente 4
- 1 aprovação basta nos blocos 1, 3 e 5 · **2 nos blocos 2 e 4**
- SLA de 24h para o primeiro retorno · ninguém aprova o próprio PR

---

## 4️⃣ A trilha — doze tarefas, cinco entregas

| Bloco | Tarefas | O que existe ao final | PR |
|:---:|---|---|:---:|
| **1** | AUT-01 · AUT-02 · AUT-03 | Dá para criar conta, entrar e sair pelo site | #1 |
| **2** | **AUT-04 + AUT-05** | Tentativa tem dono e resultado alheio dá 404 | #2 |
| **3** | AUT-06 · AUT-07 | Tentativa anônima é adotada no login; modelo de interesse no admin | #3 |
| **4** | AUT-08 · AUT-09 · AUT-10 | Botão "quero este curso" e a lista da pessoa | #4 |
| **5** | AUT-11 · AUT-12 | Suíte fechada e nenhuma tela mentindo sobre dados | #5 |

> [!important] Por que AUT-04 e AUT-05 são uma entrega só
> AUT-04 dá dono à tentativa. AUT-05 fecha a página de resultado. **Separar as duas cria uma janela em que existe dado pessoal identificável numa URL pública** — e essa janela seria o intervalo entre dois PRs, ou seja, dias. Não é preciosismo de segurança: é o tipo de coisa que uma banca pergunta e que não tem resposta boa. Os dois entram no mesmo PR ou nenhum entra.

---

## 5️⃣ Bloco 1 — AUT-01 a AUT-03 · conta e telas de acesso

**O código completo já está escrito**, arquivo por arquivo, em [[passo-a-passo-aut-01-aut-03|🔑 Passo a passo AUT-01 a AUT-03]]. Siga aquela nota inteira; ela cobre o app `accounts`, o `User` com login por e-mail, o `UserManager`, o admin, as três telas, o header e os quatro testes.

Três coisas que aquela nota destaca e que valem repetir aqui, porque são as que fazem perder uma tarde:

> [!danger] Este bloco apaga o `db.sqlite3` de todo mundo
> Trocar `AUTH_USER_MODEL` só é grátis com o banco sem usuários — é a janela que a [[spec-autenticacao-lista-interesse|spec]] manda não perder. **Antes de abrir o PR**, mande no grupo o recado pronto que está na seção 6 daquela nota. Depois do merge é tarde: alguém já vai ter atualizado e tomado erro de migration sem saber por quê.

> [!warning] O campo de login se chama `name="username"` mesmo com login por e-mail
> O `AuthenticationForm` do Django sempre nomeia assim, independente do `USERNAME_FIELD`. Trocar para `name="email"` faz o login **falhar silenciosamente** — a tela diz "não conferem" mesmo com a senha certa. É o bug mais comum desta tarefa inteira.

> [!warning] `makemigrations accounts` **antes** do `migrate`, sempre
> Na ordem errada o Django cria as tabelas de `auth` e `admin` apontando para o usuário padrão e depois acusa `Migration admin.0001_initial is applied before its dependency accounts.0001_initial`. A saída é apagar o banco e refazer na ordem certa.

**Pronto quando:** `createsuperuser` funciona com e-mail · cria conta pelo site, sai e entra de novo sem tocar no `/admin` · `/admin/` → **Users** abre e ordena por e-mail · **18 testes verdes**.

---

## 6️⃣ Bloco 2 — AUT-04 + AUT-05 · dono da tentativa e resultado fechado

> [!success] Este bloco é o coração da sua trilha
> É o único que mexe em arquivo de outra frente, o único que fecha uma falha de privacidade existente, e o que a monografia vai citar no capítulo de LGPD.

### 6.1 `quiz/models.py` — o campo `user`

No topo do arquivo, junto dos outros imports:

```python
from django.conf import settings
```

E dentro de `QuizAttempt`, logo abaixo de `respondent_name`:

```python
    user = models.ForeignKey(
        settings.AUTH_USER_MODEL,
        null=True,
        blank=True,
        on_delete=models.SET_NULL,
        related_name="tentativas",
    )
```

> [!important] `null=True` é o que preserva a decisão D2
> A D2 (quiz continua anônimo) foi ratificada pelo grupo. **Tentativa sem dono continua sendo cidadã de primeira classe** — não é estado inválido, é o caso normal de quem responde sem cadastro. Se algum dia alguém quiser tornar o campo obrigatório, está mudando uma decisão de produto, não corrigindo um modelo frouxo.

> [!note] `settings.AUTH_USER_MODEL`, nunca `from accounts.models import User`
> Importar o modelo direto cria dependência circular entre os apps e trava a reversibilidade da decisão D1. A string do settings resolve na hora certa e é o padrão do Django para isso.

### 6.2 A migration — esta você **gera**, não escreve

```bash
./.venv/Scripts/python.exe manage.py makemigrations quiz
```

Sai `quiz/migrations/0003_quizattempt_user.py`. Confira que as dependências ficaram assim:

```python
    dependencies = [
        migrations.swappable_dependency(settings.AUTH_USER_MODEL),
        ("quiz", "0002_camada_ia"),
    ]
```

> [!tip] Por que aqui pode gerar e no motor não podia
> A [[passo-a-passo-f1-01-f1-02|migration do motor]] tinha um `RenameField` e precisava copiar valores de um campo para outro — o `makemigrations` faz pergunta interativa e não sabe copiar dado. **Aqui é um `AddField` puro com valor nulo**, que é exatamente o caso em que o autodetector acerta sozinho. Escrever à mão só adicionaria chance de erro.

> [!success] A numeração já está resolvida
> O `0002_camada_ia` entrou na `main` no commit `1068eb0` (tarefas F1-01 e F1-02 do motor). Desde que você tenha dado `git pull` antes de criar a branch, a sua é a `0003` e não vai colidir com ninguém.

### 6.3 `quiz/web_views.py` — sessão e autorização

No topo:

```python
from django.http import Http404
```

Logo depois dos imports, a chave da sessão e os dois auxiliares:

```python
SESSAO_TENTATIVAS = "minhas_tentativas"


def _registrar_na_sessao(request, attempt):
    """Guarda no navegador quais tentativas esta pessoa pode ver sem ter conta."""
    ids = request.session.get(SESSAO_TENTATIVAS, [])
    if attempt.pk not in ids:
        ids.append(attempt.pk)
        # Reatribuir e obrigatorio: a sessao nao percebe append em lista.
        request.session[SESSAO_TENTATIVAS] = ids


def _pode_ver(request, attempt):
    """Dono, ou quem fez a tentativa neste navegador. Ninguem mais."""
    if attempt.user_id and attempt.user_id == request.user.id:
        return True
    if attempt.user_id is None and attempt.pk in request.session.get(SESSAO_TENTATIVAS, []):
        return True
    return False
```

> [!danger] O `request.session[...] = ids` da penúltima linha é obrigatório
> `request.session.get(...)` devolve a lista, `append` altera o objeto **em memória**, e a sessão do Django não tem como perceber isso — ela só marca `modified` quando você atribui uma chave. Sem a reatribuição, a lista some no próximo request e a pessoa toma 404 no próprio resultado. É um bug que passa em teste unitário e falha no navegador.

Em `quiz_page`, troque a criação da tentativa:

```python
    with transaction.atomic():
        attempt = QuizAttempt.objects.create(
            respondent_name=respondent_name,
            user=request.user if request.user.is_authenticated else None,
        )
        Answer.objects.bulk_create(...)   # continua igual
        recommend(attempt)

    _registrar_na_sessao(request, attempt)
    return redirect("quiz-resultado", pk=attempt.pk)
```

E em `result_page`, três linhas no começo:

```python
def result_page(request, pk):
    attempt = get_object_or_404(QuizAttempt, pk=pk)
    if not _pode_ver(request, attempt):
        raise Http404("Tentativa nao encontrada.")
    recomendacoes = attempt.recommendations.select_related("course__main_area")
    ...   # o resto continua igual
```

> [!important] 404, não 403
> `403 Proibido` confirma que a tentativa 7 existe e é de outra pessoa. `404` não confirma nada — quem sonda a URL não descobre nem quantas tentativas o sistema tem. É a mesma razão pela qual a tela de login diz "e-mail ou senha não conferem" em vez de "esse e-mail não existe".

### 6.4 A mesma falha existe na API — e ninguém tinha visto

`quiz/views.py::AttemptResultView` é um `RetrieveAPIView` com `queryset = QuizAttempt.objects.all()`. **`GET /api/quiz/attempts/7/` devolve o resultado de qualquer pessoa**, exatamente como a página fazia. Fechar só o `result_page` deixaria a porta dos fundos aberta.

Como a API não usa sessão de navegador, a regra lá é mais simples:

```python
class AttemptResultView(generics.RetrieveAPIView):
    permission_classes = [permissions.IsAuthenticated]

    def get_queryset(self):
        return QuizAttempt.objects.filter(user=self.request.user)
```

> [!warning] Filtrar o queryset, não buscar e comparar depois
> `get_object_or_404(QuizAttempt, pk=pk)` e **depois** checar o dono é o caminho pelo qual esse bug entra em todo projeto — basta alguém mexer na view e esquecer a checagem. Filtrando o queryset, a autorização é a própria consulta: não existe caminho de código que devolva o objeto errado.

> [!note] Isso muda o contrato da API — avise a Frente 4
> `POST /api/quiz/submit/` continua público (D2), mas `GET /api/quiz/attempts/<pk>/` passa a exigir login. Se a F4 estiver usando esse endpoint em algum dashboard, ela precisa saber **antes** do merge. Marque no campo "toca contrato de outra frente" do PR.

### 6.5 Os testes deste bloco

Em `accounts/tests.py`, junto dos que o Bloco 1 criou:

```python
class ResultadoProtegidoTest(TestCase):
    @classmethod
    def setUpTestData(cls):
        silencio = StringIO()
        for comando in ("seed_areas", "seed_courses", "seed_questions"):
            call_command(comando, stdout=silencio)

    def _tentativa_de(self, user=None):
        attempt = QuizAttempt.objects.create(respondent_name="teste", user=user)
        recommend(attempt)
        return attempt

    def test_dono_ve_o_proprio_resultado(self):
        ana = get_user_model().objects.create_user(email="ana@exemplo.com", password="afinidade-2026")
        attempt = self._tentativa_de(ana)
        self.client.force_login(ana)
        self.assertEqual(self.client.get(reverse("quiz-resultado", args=[attempt.pk])).status_code, 200)

    def test_resultado_de_outro_usuario_da_404(self):
        """O IDOR que existia antes desta tarefa."""
        ana = get_user_model().objects.create_user(email="ana@exemplo.com", password="afinidade-2026")
        bruno = get_user_model().objects.create_user(email="bruno@exemplo.com", password="afinidade-2026")
        attempt = self._tentativa_de(ana)
        self.client.force_login(bruno)
        self.assertEqual(self.client.get(reverse("quiz-resultado", args=[attempt.pk])).status_code, 404)

    def test_visitante_ve_a_tentativa_que_acabou_de_fazer(self):
        """D2: sem conta, o fluxo do quiz continua inteiro."""
        dados = {"respondent_name": "Visitante"}
        for question in Question.objects.filter(is_active=True):
            dados[f"question_{question.id}"] = question.choices.first().id
        resposta = self.client.post(reverse("quiz-inicio"), dados, follow=True)
        self.assertEqual(resposta.status_code, 200)

    def test_tentativa_alheia_sem_dono_da_404(self):
        """Sem conta e sem sessao, adivinhar o pk nao abre mais nada."""
        attempt = self._tentativa_de()
        self.assertEqual(self.client.get(reverse("quiz-resultado", args=[attempt.pk])).status_code, 404)
```

> [!important] O quarto teste documenta uma mudança de comportamento, não só um bug fechado
> As tentativas de desenvolvimento que já estão no banco de todo mundo **param de abrir pelo link**. Isso é o correto, mas alguém do grupo vai estranhar. Escreva na descrição do PR: *"tentativa antiga só abre pelo `/admin` — é o comportamento novo, não um bug"*.

**Pronto quando:** os quatro testes passam · `/resultado/<pk>` de outra pessoa dá 404 no navegador · `GET /api/quiz/attempts/<pk>/` deslogado dá 403 · responder o quiz sem conta ainda leva ao resultado.

---

## 7️⃣ Bloco 3 — AUT-06 + AUT-07 · adoção da tentativa e modelo de interesse

### AUT-06 — o *claim*

Quando alguém responde o quiz sem conta e depois cria uma, as tentativas daquele navegador passam a ser dela. O contrato C6 da [[spec-autenticacao-lista-interesse|spec]] define a regra inteira:

```
adota a tentativa  ⟺  o pk está na sessão deste navegador  E  attempt.user is None
```

> [!danger] As duas condições, sempre as duas
> Sem a primeira, qualquer pessoa logada adota a tentativa dos outros mandando ids na mão. Sem a segunda, adota as que já têm dono. **É o mesmo tipo de furo da AUT-05, com outra roupa** — e a única diferença é que aqui ele é de escrita, não de leitura.

Implemente ouvindo o sinal `user_logged_in` (dispara tanto no login quanto no cadastro, então cobre os dois caminhos com um código só):

```python
# accounts/signals.py
from django.contrib.auth.signals import user_logged_in
from django.dispatch import receiver

from quiz.models import QuizAttempt
from quiz.web_views import SESSAO_TENTATIVAS


@receiver(user_logged_in)
def adotar_tentativas_da_sessao(sender, request, user, **kwargs):
    ids = request.session.get(SESSAO_TENTATIVAS, [])
    if ids:
        QuizAttempt.objects.filter(pk__in=ids, user__isnull=True).update(user=user)
```

Registre em `accounts/apps.py` (`def ready(self): from accounts import signals`).

**Testes:** responde anônimo → cria conta → a tentativa aparece como dela · tentativa **com** dono não muda de dono nem estando na sessão.

### AUT-07 — `InterestItem`

O modelo inteiro está escrito no contrato **C4** da [[spec-autenticacao-lista-interesse|spec]] — copie de lá, incluindo os comentários. Os pontos que não podem ser alterados sem falar com o grupo:

| Escolha | Por quê |
|---|---|
| `course` com `on_delete=PROTECT` | Interesse é evidência institucional de demanda. Apagar um curso não pode apagar em silêncio a prova de que 14 pessoas o queriam |
| `unique_together ("user", "course")` | O botão vira alternador, não gerador de duplicata |
| `score_snapshot` · `rank_snapshot` · `was_primary` | **É a razão de a spec existir** — sem eles não há taxa de conversão nem comparação engine × IA por comportamento real |

Mais o `admin.py` com `list_filter` por curso e por status: é por ali que a defesa mostra os interesses registrados.

**Pronto quando:** admin lista interesses filtrando por curso e status · testes do claim verdes.

---

## 8️⃣ Bloco 4 — AUT-08 + AUT-09 + AUT-10 · o botão e a lista

- **AUT-08** — salvar/remover por **POST**, alternador, com os três snapshots preenchidos no momento do clique. Toca `templates/quiz/result.html`, que é da Frente 4.
- **AUT-09** — `/minha-lista/` e `/meus-testes/`, ambas `login_required`.
- **AUT-10** — intenção pendente: quem clica em salvar sem conta tem `{"course_id", "attempt_id"}` guardado na sessão, faz login, volta e conclui em **um** clique, por um botão de POST num banner.

> [!warning] `login_required` responde "é alguém?", não "é o dono?"
> A proteção da lista é o **queryset filtrado**: `InterestItem.objects.filter(user=request.user)`. Buscar por `pk` e comparar o dono depois é o mesmo furo da AUT-05 pela terceira vez nesta trilha. Se você pegou o padrão aqui, pegou a ideia inteira.

> [!note] Por que não salvar automático depois do login
> Seria um clique a menos — e uma mudança de estado disparada por um `GET` de redirect. Funciona, e não tem defesa quando a banca pergunta sobre CSRF. O banner custa uma condição no template. Decisão **D4**, já registrada.

**Pronto quando:** salvar duas vezes não duplica · `rank_snapshot` gravado · deslogado clica em salvar, cria conta, volta e conclui em um clique.

---

## 9️⃣ Bloco 5 — AUT-11 + AUT-12 · fechar a suíte e parar de mentir

**AUT-11** — a bateria completa está listada na seção 5 da [[spec-autenticacao-lista-interesse|spec]]: onze testes. Boa parte você já escreveu ao longo dos blocos; aqui é conferir o que falta e fechar.

**AUT-12** — o aviso de privacidade. E uma correção que **não é opcional**:

> [!danger] Hoje o rodapé mente em toda página do sistema
> `templates/base.html` afirma *"Nenhum dado e compartilhado"*. Estava correto enquanto tudo era anônimo. A partir do seu Bloco 1 existe cadastro com e-mail, e essa frase vira **declaração falsa exibida ao usuário** — projetada na parede no dia da defesa, numa banca que pode perguntar sobre LGPD. O passo a passo do Bloco 1 já troca essa linha; a AUT-12 fecha o resto: parágrafo no cadastro, botão de apagar conta e o texto da monografia.

**Pronto quando:** **~25 testes verdes** · nenhuma tela afirma algo falso sobre os dados.

---

## 🔟 Coordenação — o que avisar e o que não tocar

### O recado do Bloco 1, antes do PR

> Subi a branch `feat/autenticacao`. Ela troca o modelo de usuário do Django (login por e-mail), então **quando você atualizar, precisa apagar o banco local e popular de novo**:
>
> ```
> rm db.sqlite3
> python manage.py migrate
> python manage.py seed_areas
> python manage.py seed_courses
> python manage.py seed_questions
> python manage.py createsuperuser
> ```
>
> São dois minutos e só precisa ser feito uma vez. As tentativas de teste que estavam no seu banco somem — nenhuma delas era dado real.
>
> ⚠️ **F3:** a partir de agora, curso cadastrado na mão pelo `/admin` se perde nesse processo. Os 18 cursos reais precisam entrar por `seed_courses`, não pelo painel.

> [!danger] O aviso à F3 é a parte que não pode ser esquecida
> Se alguém cadastrar o catálogo real pelo `/admin` antes desta branch ser mesclada, esse trabalho evapora no `rm db.sqlite3` — e é **trabalho humano de julgar 7 pesos por curso**, não código que se roda de novo.

### O que não é seu

| Arquivo | Dono | Regra |
|---|---|---|
| `quiz/engine.py` | Frente 1 | Não toque. Nenhuma tarefa sua precisa dele |
| `quiz/delivery.py`, `quiz/llm/` | Frente 2 | Não existem ainda; quando existirem, não são seus |
| `catalog/` e os `seed_*` | Frente 3 | Não cadastre curso, nem por seed nem pelo admin |
| `templates/quiz/result.html` | Frente 4 | Você **precisa** mexer na AUT-08 — combine antes, revise junto |
| `quiz/models.py` | Frente 1 | Você acrescenta **um** campo na AUT-04. Nada mais |

### A ordem que não pode inverter

```
Bloco 1 ──► Bloco 2 ──► Bloco 3 ──► Bloco 4 ──► Bloco 5
   │            │
   │            └── AUT-04 e AUT-05 no mesmo PR, sempre
   └── quanto antes, melhor: fecha a janela do AUTH_USER_MODEL
```

---

## 1️⃣1️⃣ Perguntas que só o scrum master responde

Leve estas para a primeira conversa. Nenhuma trava o Bloco 1 — mas as três primeiras travam o resto.

1. **A decisão D1 está ratificada?** (User customizado com login por e-mail.) O passo a passo assume a opção A e marca o Plano B no lugar exato em que o código muda. **Esta é a única com prazo de validade** — ela expira no primeiro cadastro do piloto com respondentes reais.
2. **Quem é seu revisor fixo**, e quem representa a Frente 4 nos blocos 2 e 4?
3. **A API `/api/quiz/attempts/<pk>/` pode passar a exigir login?** Se a F4 já consome esse endpoint em algum dashboard, a mudança da seção 6.4 precisa ser combinada antes.
4. D3 (uma lista plana com status) e D4 (salvar por POST) — só importam a partir do Bloco 3.
5. **`LOGIN_REDIRECT_URL` aponta para `quiz-inicio` até a AUT-09.** Quando `/minha-lista/` existir, alguém precisa lembrar de voltar no `settings.py`.

## 📎 Veja também

- [[spec-autenticacao-lista-interesse|🔐 Spec de autenticação e lista de interesse]] — o que e o porquê de cada tarefa, contratos C4/C5/C6
- [[passo-a-passo-aut-01-aut-03|🔑 Passo a passo AUT-01 a AUT-03]] — o código do Bloco 1, arquivo por arquivo
- [[git-fluxo-aplicado-tcc|🎓 Fluxo de Git do TCC]] · [[git-convencao-de-branches|🏷️ Convenção de branches]] · [[git-pull-request-e-code-review|🔍 PR e code review]]
- [[divisao-de-trabalho-tcc|👥 Divisão de trabalho]] · [[spec-motor-e-ia-frentes-1-2|🧭 Spec F1+F2]] · [[passo-a-passo-f1-01-f1-02|🔧 Passo a passo F1-01 e F1-02]]
- [[front-templates-django|🎨 Front em templates]] · [[testes-e-validacao-tcc|✅ Testes]] · [[modelagem-dados-quiz|🗃️ Modelagem]]
- **Conceitos:** [[cs-authentication|Autenticação]] · [[Migrations|Migrations]] · [[tst-testes-django|Testes em Django]] · [[cs-csrf|CSRF]]
- [[TCC|🎓 TCC]] · [[Projetos|🚀 Projetos]]
