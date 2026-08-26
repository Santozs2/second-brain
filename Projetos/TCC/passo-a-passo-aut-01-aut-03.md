---
title: "Passo a passo — AUT-01 a AUT-03 (conta de usuário e telas de acesso)"
aliases: ["Passo a passo AUT", "Como fazer AUT-01", "User customizado Django", "Telas de login do TCC"]
tags: [tcc, autenticacao, django, execucao, passo-a-passo, migration]
status: em-andamento
projeto: TCC
criado: 2026-08-25
---

> [!info] Spec: [[spec-autenticacao-lista-interesse|🔐 Spec de autenticação e lista de interesse]] · Front: [[front-templates-django|🎨 Templates]] · Testes: [[testes-e-validacao-tcc|✅ Testes]] · Nota irmã: [[passo-a-passo-f1-01-f1-02|🔧 Passo a passo F1-01 e F1-02]]

# 🔑 Passo a passo — AUT-01 a AUT-03

> [!abstract] O que esta nota é
> O **como** das três primeiras tarefas da [[spec-autenticacao-lista-interesse|spec de autenticação]], para executar com o editor aberto. Arquivo por arquivo, com o código no lugar certo e os comandos que provam que funcionou.
> Escrito contra o repositório no commit `1068eb0`.

> [!warning] Nada disto foi executado
> O código abaixo foi escrito contra a leitura do repositório, não rodado. **A seção 5 é o que prova que deu certo** — não pule nenhum dos comandos.

## 🧭 Estado das decisões

| | Decisão | Estado |
|---|---|---|
| **D1** | User customizado com login por e-mail | ⚠️ **assumida como opção A** nesta nota — ver o Plano B na seção 1 |
| **D2** | Quiz continua anônimo | ✅ **ratificada** — e virou teste automatizado na seção 4 |
| D3 | Uma lista plana com status | ainda não importa (é da AUT-07) |
| D4 | Salvar por POST + intenção na sessão | ainda não importa (é da AUT-08/10) |

> [!success] Por que AUT-03 entrou nesta nota
> A AUT-02 termina com *"cria conta, sai e entra de novo sem tocar no `/admin`"* — e isso é **indemonstrável** enquanto nenhuma tela tiver link para o login. AUT-03 são doze linhas de template e três de settings. Separar em outro PR seria entregar uma porta sem maçaneta.

---

## 🚦 Antes de abrir o editor

```bash
git checkout -b feat/autenticacao
```

> [!danger] Esta branch apaga o `db.sqlite3` de todo mundo
> Trocar o `AUTH_USER_MODEL` só é grátis com o banco vazio de usuários — é exatamente a janela que a [[spec-autenticacao-lista-interesse|spec]] diz para não perder. **As 5 tentativas de teste que existem hoje vão embora.** Isso é aceitável porque elas são de desenvolvimento e o resto do banco nasce de `seed_*`. Se alguém quiser guardá-las:
> ```bash
> ./.venv/Scripts/python.exe manage.py dumpdata quiz.QuizAttempt quiz.Answer quiz.Recommendation --indent 2 > tentativas-antigas.json
> ```
> O `loaddata` só devolve certo se os ids dos cursos baterem — e batem, desde que você rode os três seeds na mesma ordem de sempre.

Avise o grupo **antes de abrir o PR**, não depois. Texto pronto para colar na seção 6.

---

## 1️⃣ AUT-01 — o app `accounts` e o modelo de usuário

```bash
./.venv/Scripts/python.exe manage.py startapp accounts
```

### `accounts/models.py` — substitua o arquivo inteiro

```python
from django.contrib.auth.models import AbstractUser, BaseUserManager
from django.db import models


class UserManager(BaseUserManager):
    """O manager padrao do Django exige username; o nosso identifica pelo e-mail."""

    use_in_migrations = True

    def _create_user(self, email, password, **extra):
        if not email:
            raise ValueError("E-mail e obrigatorio.")
        user = self.model(email=self.normalize_email(email), **extra)
        user.set_password(password)
        user.save(using=self._db)
        return user

    def create_user(self, email, password=None, **extra):
        extra.setdefault("is_staff", False)
        extra.setdefault("is_superuser", False)
        return self._create_user(email, password, **extra)

    def create_superuser(self, email, password=None, **extra):
        extra.setdefault("is_staff", True)
        extra.setdefault("is_superuser", True)
        if extra.get("is_staff") is not True:
            raise ValueError("Superusuario precisa de is_staff=True.")
        if extra.get("is_superuser") is not True:
            raise ValueError("Superusuario precisa de is_superuser=True.")
        return self._create_user(email, password, **extra)


class User(AbstractUser):
    """Login por e-mail. O username sai de cena, nao fica vazio no banco."""

    username = None
    email = models.EmailField("e-mail", unique=True)
    first_name = models.CharField("nome", max_length=150, blank=True)

    USERNAME_FIELD = "email"
    REQUIRED_FIELDS = []

    objects = UserManager()

    def __str__(self):
        return self.first_name or self.email
```

> [!warning] `use_in_migrations = True` não é enfeite
> Sem essa linha, um `RunPython` que faça `User.objects.create_user(...)` numa migration futura recebe o manager genérico, que **não sabe criar usuário sem username**. É o tipo de erro que aparece meses depois, na máquina de outra pessoa, na hora de rodar as migrations do zero.

### `accounts/admin.py`

```python
from django.contrib import admin
from django.contrib.auth.admin import UserAdmin as BaseUserAdmin

from accounts.models import User


@admin.register(User)
class UserAdmin(BaseUserAdmin):
    # O UserAdmin nativo ordena e busca por username, que nao existe mais.
    ordering = ["email"]
    list_display = ["email", "first_name", "is_staff", "date_joined"]
    search_fields = ["email", "first_name"]
    fieldsets = (
        (None, {"fields": ("email", "password")}),
        ("Dados", {"fields": ("first_name", "last_name")}),
        (
            "Permissoes",
            {"fields": ("is_active", "is_staff", "is_superuser", "groups", "user_permissions")},
        ),
        ("Datas", {"fields": ("last_login", "date_joined")}),
    )
    add_fieldsets = (
        (None, {"classes": ("wide",), "fields": ("email", "password1", "password2")}),
    )
```

> [!warning] Registrar o User novo com o `UserAdmin` nativo **quebra o `/admin`**
> `BaseUserAdmin` traz `ordering = ("username",)` e `fieldsets` com `username` dentro. Herdar sem sobrescrever esses três atributos dá `FieldError` — e não é no import, é **quando alguém abre a lista de usuários**. Ou seja: passa nos testes e quebra na demonstração.

### `config/settings.py`

Em `INSTALLED_APPS`, junto dos outros apps do projeto:

```python
    'rest_framework',
    'quiz',
    'catalog',
    'accounts',
```

E no fim do arquivo, ao lado do `RECOMMENDATION_LIMIT`:

```python
AUTH_USER_MODEL = "accounts.User"
```

### Recriar o banco — nesta ordem

```bash
rm db.sqlite3
./.venv/Scripts/python.exe manage.py makemigrations accounts
./.venv/Scripts/python.exe manage.py migrate
./.venv/Scripts/python.exe manage.py seed_areas
./.venv/Scripts/python.exe manage.py seed_courses
./.venv/Scripts/python.exe manage.py seed_questions
./.venv/Scripts/python.exe manage.py createsuperuser
```

> [!important] `makemigrations accounts` **antes** do `migrate`, sempre
> Se o `migrate` rodar primeiro, o Django cria as tabelas de `auth` e `admin` apontando para o usuário padrão e depois reclama que `AUTH_USER_MODEL` mudou no meio do caminho. A mensagem que aparece é a famosa `Migration admin.0001_initial is applied before its dependency accounts.0001_initial` — e a saída dela é apagar o banco de novo, na ordem certa.

> [!note] Plano B — se o grupo ratificar a opção B da D1
> Pule esta seção inteira: sem app `accounts` para o modelo, sem `AUTH_USER_MODEL`, **sem apagar o banco**. O app `accounts` ainda nasce (ele guarda as views, os forms e, na AUT-07, o `InterestItem`), só que sem `models.User`. Nas seções seguintes, troque `accounts.models.User` por `django.contrib.auth.get_user_model()` — que é o que o código já usa — e no formulário de cadastro o campo passa a ser `username` em vez de `email`. **Nada mais muda**, e é isso que torna a D1 reversível.

---

## 2️⃣ AUT-02 — cadastro, login e logout

### `accounts/forms.py` — arquivo novo

```python
from django.contrib.auth.forms import UserCreationForm

from accounts.models import User


class CadastroForm(UserCreationForm):
    class Meta:
        model = User
        fields = ["first_name", "email"]
```

> [!tip] Se o Django reclamar de `username` aqui
> Em algumas versões o `UserCreationForm` carrega configuração presa ao campo `username`. A troca é de uma palavra: importe `BaseUserCreationForm` no lugar (existe do Django 4.2 em diante) e herde dele. O resto do formulário é idêntico.

### `accounts/views.py` — substitua o arquivo

```python
from django.contrib.auth import login
from django.urls import reverse
from django.views.generic import CreateView

from accounts.forms import CadastroForm


class CadastroView(CreateView):
    form_class = CadastroForm
    template_name = "accounts/cadastro.html"

    def form_valid(self, resposta_do_form):
        retorno = super().form_valid(resposta_do_form)
        # Quem acabou de criar a conta ja entra logado: pedir a senha de novo
        # na tela seguinte e a forma mais rapida de perder a pessoa.
        login(self.request, self.object)
        return retorno

    def get_success_url(self):
        return self.request.POST.get("next") or reverse("quiz-inicio")
```

> [!note] `login()` sem `backend=` só funciona porque há **um** backend
> O projeto usa o `ModelBackend` padrão e nada mais. No dia em que alguém acrescentar um segundo backend de autenticação, esta linha passa a exigir `backend="django.contrib.auth.backends.ModelBackend"` — e o erro que aparece (`ValueError: You have multiple authentication backends`) não diz que a culpa é daqui.

### `accounts/urls.py` — arquivo novo

```python
from django.contrib.auth import views as auth_views
from django.urls import path

from accounts.views import CadastroView

urlpatterns = [
    path(
        "entrar/",
        auth_views.LoginView.as_view(template_name="accounts/entrar.html"),
        name="entrar",
    ),
    path("sair/", auth_views.LogoutView.as_view(), name="sair"),
    path("criar-conta/", CadastroView.as_view(), name="criar-conta"),
]
```

> [!warning] No Django 5, sair é `POST` — link não funciona mais
> `LogoutView` recusa `GET` desde o Django 5.0. Por isso o header da AUT-03 tem um `<form>` e não um `<a>`. Quem tentar `<a href="{% url 'sair' %}">` vai levar **405 Method Not Allowed** e achar que errou a URL.

### `config/urls.py`

```python
urlpatterns = [
    path("admin/", admin.site.urls),
    path("api/quiz/", include("quiz.urls")),
    path("", include("accounts.urls")),
    path("", include("quiz.web_urls")),
]
```

> [!note] `accounts` antes de `quiz.web_urls`
> Não há conflito real (as rotas têm prefixos diferentes), mas `quiz.web_urls` registra a raiz `""`. Manter quem tem prefixo antes de quem tem raiz é a ordem que não dá surpresa quando alguém acrescentar rota nova.

### `templates/accounts/entrar.html` — arquivo novo

```html
{% extends "base.html" %}

{% block title %}Entrar{% endblock %}

{% block conteudo %}
<section class="passo">
    <p class="olho">Sua conta</p>
    <h1 class="titulo">Entrar</h1>

    <form method="post">
        {% csrf_token %}
        <input type="hidden" name="next" value="{{ next }}">

        {% if form.errors %}
        <p class="alerta" role="alert">E-mail ou senha nao conferem.</p>
        {% endif %}

        <label class="campo">
            <span class="campo__rotulo">E-mail</span>
            <input class="campo__entrada" type="email" name="username"
                   value="{{ form.username.value|default:'' }}" required autofocus>
        </label>

        <label class="campo">
            <span class="campo__rotulo">Senha</span>
            <input class="campo__entrada" type="password" name="password" required>
        </label>

        <div class="acoes">
            <button class="botao botao--principal" type="submit">Entrar</button>
            <a class="botao botao--fantasma" href="{% url 'criar-conta' %}">Criar conta</a>
        </div>
    </form>
</section>
{% endblock %}
```

> [!warning] O campo se chama `name="username"` mesmo com login por e-mail
> O `AuthenticationForm` sempre nomeia o campo de identificação como `username`, seja qual for o `USERNAME_FIELD`. O `type="email"` e o rótulo cuidam da parte que a pessoa vê; **trocar o `name` para `email` faz o login falhar silenciosamente**, com a tela dizendo "não conferem" mesmo com a senha certa. É o bug mais comum desta tarefa inteira.

### `templates/accounts/cadastro.html` — arquivo novo

```html
{% extends "base.html" %}

{% block title %}Criar conta{% endblock %}

{% block conteudo %}
<section class="passo">
    <p class="olho">Leva menos de um minuto</p>
    <h1 class="titulo">Criar conta</h1>
    <p class="subtitulo">
        A conta serve para guardar os cursos que voce quer fazer e rever seus resultados.
        Responder o quiz continua nao exigindo cadastro.
    </p>

    <form method="post">
        {% csrf_token %}
        <input type="hidden" name="next" value="{{ request.GET.next|default:'' }}">

        {{ form.non_field_errors }}

        {% for campo in form %}
        <label class="campo">
            <span class="campo__rotulo">{{ campo.label }}</span>
            <input class="campo__entrada" type="{{ campo.field.widget.input_type }}"
                   name="{{ campo.html_name }}" value="{{ campo.value|default:'' }}"
                   {% if campo.field.required %}required{% endif %}>
            {% if campo.errors %}<span class="aviso">{{ campo.errors.0 }}</span>{% endif %}
            {% if campo.help_text %}<span class="campo__dica">{{ campo.help_text|safe }}</span>{% endif %}
        </label>
        {% endfor %}

        <div class="acoes">
            <button class="botao botao--principal" type="submit">Criar conta</button>
            <a class="botao botao--fantasma" href="{% url 'entrar' %}">Ja tenho conta</a>
        </div>
    </form>

    <p class="campo__dica">
        Guardamos seu nome e e-mail para identificar sua lista. Voce pode apagar sua conta quando quiser.
    </p>
</section>
{% endblock %}
```

> [!tip] O `{% for campo in form %}` é de propósito aqui e não na tela de login
> No cadastro os campos vêm do `UserCreationForm` (nome, e-mail, senha, confirmação) e a lista **muda** se a D1 for para o Plano B. Percorrer o form deixa a tela funcionando nos dois casos. No login são dois campos fixos, e escrevê-los à mão é o que permite o `type="email"` com `name="username"` — que nenhum laço genérico produziria.

> [!note] O parágrafo do rodapé do formulário é a AUT-12 começando
> Uma frase dizendo o que é guardado, na hora em que a pessoa digita o e-mail. Custa nada agora e evita a discussão "onde a gente avisa sobre LGPD?" na semana da entrega.

---

## 3️⃣ AUT-03 — a maçaneta da porta

### `config/settings.py` — no fim, junto do `AUTH_USER_MODEL`

```python
LOGIN_URL = "entrar"
LOGIN_REDIRECT_URL = "quiz-inicio"    # vira "interesse-lista" na AUT-09
LOGOUT_REDIRECT_URL = "quiz-inicio"
```

> [!warning] Não aponte `LOGIN_REDIRECT_URL` para `interesse-lista` agora
> Essa rota só nasce na AUT-09. Apontar para ela hoje derruba **todo login bem-sucedido** com `NoReverseMatch` — um erro que aparece depois da senha certa, o que faz parecer problema de autenticação. O comentário na linha existe para você lembrar de voltar aqui.

### `templates/base.html` — dentro do `<header class="topo">`

Troque a linha do `topo__tag` por:

```html
        <nav class="sessao">
            <span class="topo__tag">Orientacao profissional</span>
            {% if user.is_authenticated %}
                <span class="sessao__nome">{{ user.first_name|default:user.email }}</span>
                <form method="post" action="{% url 'sair' %}" class="sessao__sair">
                    {% csrf_token %}
                    <button class="botao botao--fantasma" type="submit">Sair</button>
                </form>
            {% else %}
                <a class="botao botao--fantasma" href="{% url 'entrar' %}">Entrar</a>
            {% endif %}
        </nav>
```

> [!success] O `{{ user }}` já funciona sem você fazer nada
> O `django.contrib.auth.context_processors.auth` **já está** no `settings.py` desde o primeiro dia do projeto. Nenhum `context` precisa ser passado nas views para o header funcionar em todas as telas.

### `static/css/style.css` — no fim do arquivo

```css
.sessao {
    display: flex;
    align-items: center;
    gap: 0.75rem;
}

.sessao__nome {
    font-size: 0.875rem;
    opacity: 0.8;
}

.sessao__sair {
    margin: 0;
}
```

### `templates/base.html` — o rodapé que hoje mente

```html
        <p>Recomendacoes calculadas por similaridade de perfil. Seus dados ficam nesta instituicao.</p>
```

> [!danger] Esta linha não é opcional
> O rodapé atual afirma *"Nenhum dado e compartilhado"* em **toda página do sistema**. A partir do momento em que existe cadastro com e-mail, isso é uma declaração falsa exibida ao usuário — projetada na parede no dia da defesa, numa banca que pode perguntar sobre LGPD. Trocar custa dez segundos e faz parte desta tarefa, não da AUT-12.

---

## 4️⃣ Os quatro testes novos

### `accounts/tests.py` — substitua o arquivo

```python
from io import StringIO

from django.contrib.auth import get_user_model
from django.core.management import call_command
from django.test import TestCase
from django.urls import reverse

from quiz.models import Question


class ContaTest(TestCase):
    def test_cadastro_cria_conta_e_ja_entra_logado(self):
        resposta = self.client.post(
            reverse("criar-conta"),
            {
                "first_name": "Ana",
                "email": "ana@exemplo.com",
                "password1": "afinidade-2026",
                "password2": "afinidade-2026",
            },
        )
        self.assertEqual(resposta.status_code, 302)
        self.assertTrue(get_user_model().objects.filter(email="ana@exemplo.com").exists())
        self.assertIn("_auth_user_id", self.client.session)

    def test_email_duplicado_volta_o_formulario_e_nao_500(self):
        get_user_model().objects.create_user(email="ana@exemplo.com", password="afinidade-2026")
        resposta = self.client.post(
            reverse("criar-conta"),
            {
                "first_name": "Outra Ana",
                "email": "ana@exemplo.com",
                "password1": "afinidade-2026",
                "password2": "afinidade-2026",
            },
        )
        self.assertEqual(resposta.status_code, 200)
        # Nao comparamos o texto da mensagem: ele vem traduzido e muda com o locale.
        self.assertTrue(resposta.context["form"].errors)
        self.assertEqual(get_user_model().objects.count(), 1)

    def test_createsuperuser_funciona_sem_username(self):
        """A armadilha da D1: sem o UserManager proprio, este teste explode."""
        call_command(
            "createsuperuser",
            email="chefe@exemplo.com",
            interactive=False,
            stdout=StringIO(),
        )
        chefe = get_user_model().objects.get(email="chefe@exemplo.com")
        self.assertTrue(chefe.is_staff)
        self.assertTrue(chefe.is_superuser)


class QuizAnonimoTest(TestCase):
    @classmethod
    def setUpTestData(cls):
        silencio = StringIO()
        for comando in ("seed_areas", "seed_courses", "seed_questions"):
            call_command(comando, stdout=silencio)

    def test_quiz_anonimo_continua_respondendo(self):
        """D2 ratificada: nenhuma tela do quiz pode passar a exigir conta."""
        dados = {"respondent_name": "Visitante"}
        for question in Question.objects.filter(is_active=True):
            dados[f"question_{question.id}"] = question.choices.first().id

        resposta = self.client.post(reverse("quiz-inicio"), dados)

        self.assertEqual(resposta.status_code, 302)
        self.assertIn("/resultado/", resposta["Location"])
```

> [!important] `test_quiz_anonimo_continua_respondendo` é o guardião da D2
> Ele parece bobo hoje — obviamente o quiz responde, ninguém mexeu nele. O valor aparece na AUT-08, quando alguém for proteger a tela de salvar e puser um `login_required` um nível acima do que devia. **A D2 é uma decisão de produto; este teste é a única coisa no repositório que a defende sozinha.**

---

## 5️⃣ Verificação — é aqui que você descobre se deu certo

```bash
./.venv/Scripts/python.exe manage.py makemigrations --check --dry-run
```

Tem que dizer que **não há mudanças pendentes**. Se ele quiser criar migration para `accounts`, o `models.py` mudou depois do `makemigrations` da seção 1.

```bash
PYTHONIOENCODING=utf-8 ./.venv/Scripts/python.exe manage.py test
```

**18 testes verdes** — os 14 de antes mais os 4 novos.

```bash
./.venv/Scripts/python.exe manage.py runserver 8010
```

E os cinco cliques que nenhum teste automatizado faz por você:

| # | O quê | Sinal de que passou |
|---|---|---|
| 1 | Abrir `/` deslogado | Botão **Entrar** no topo, quiz funcionando normalmente |
| 2 | `/criar-conta/` → criar conta | Volta para a home **já logada**, com seu nome no topo |
| 3 | Clicar em **Sair** | Volta para a home com o botão Entrar |
| 4 | `/entrar/` com a conta criada | Entra; **com a senha errada**, mostra o alerta e não 500 |
| 5 | `/admin/` → **Users** | A lista abre e ordena por e-mail |

> [!important] O clique 5 é o que os testes nunca pegam
> O `FieldError` do `UserAdmin` (seção 1) só acontece quando a página da lista é renderizada. A suíte inteira pode estar verde com o `/admin` quebrado — e o `/admin` é onde a F3 vai cadastrar os 18 cursos e onde a defesa mostra os interesses registrados.

---

## 6️⃣ Commit e o recado para o grupo

```bash
git add -A && git commit -m "feat(contas): usuario com login por e-mail, telas de acesso e sessao no topo"
```

Recado pronto para colar no grupo, **antes do merge**:

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

> [!warning] O aviso à F3 é a parte que não pode ser esquecida
> Se alguém cadastrar o catálogo real pelo `/admin` antes de a branch ser mesclada, esse trabalho evapora no `rm db.sqlite3` — e é trabalho humano de julgar 7 pesos por curso, não código que se roda de novo. A [[spec-autenticacao-lista-interesse|spec]] chama isso de "a janela da D1 fechando antes do prazo".

## ▶️ Próxima ação

**AUT-04 e AUT-05, no mesmo PR.** O campo `QuizAttempt.user` e o fechamento do `result_page` são o único par desta spec que não pode ser separado: assim que a tentativa ganha dono, `/resultado/<pk>/` aberto deixa de ser detalhe e vira vazamento de dado pessoal.

## 📎 Veja também

- [[spec-autenticacao-lista-interesse|🔐 Spec de autenticação e lista de interesse]] — o que e o porquê destas tarefas
- [[passo-a-passo-f1-01-f1-02|🔧 Passo a passo de F1-01 e F1-02]] — a nota irmã, mesmo formato
- [[front-templates-django|🎨 Front em templates Django]] · [[testes-e-validacao-tcc|✅ Testes e validação]] · [[bugs-e-licoes-tcc|🐛 Bugs e lições]]
- [[divisao-de-trabalho-tcc|👥 Divisão de trabalho]] — o aviso da seção 6 é para a F3
- **Conceitos:** [[Migrations|Migrations]] · [[Models|Models]] · [[tst-testes-django|Testes em Django]]
- [[TCC|🎓 TCC]] · [[Projetos|🚀 Projetos]]
