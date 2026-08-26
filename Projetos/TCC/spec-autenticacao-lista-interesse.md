---
title: "Spec — autenticação e lista de interesse"
aliases: ["Spec de autenticação", "Lista de interesse", "Registro de interesse", "Spec AUT", "Conta de usuário TCC"]
tags: [tcc, spec, autenticacao, django, produto, planejamento, seguranca]
status: proposto
projeto: TCC
criado: 2026-08-25
---

> [!info] Projeto: [[TCC|🎓 TCC]] · Escopo: [[escopo-fluxo-educmatch|🗺️ EducMatch]] · Divisão: [[divisao-de-trabalho-tcc|👥 4 frentes]] · Spec irmã: [[spec-motor-e-ia-frentes-1-2|🧭 Spec F1+F2]] · Modelagem: [[modelagem-dados-quiz|🗃️ Modelagem]]

# 🔐 Spec — autenticação e lista de interesse

> [!abstract] O que este documento é
> A **especificação executável** do item *"registro de interesse"* que o quadro do grupo desenhou e que [[escopo-fluxo-educmatch|🗺️ o recorte de escopo]] marcou como **entra no TCC**. Para a pessoa poder guardar um curso, ela precisa ter onde guardar — e é aí que entra a autenticação, que **não existe hoje** no sistema.
> Auditoria do repositório feita em **2026-08-25**, sobre o commit `1068eb0`.

> [!important] A frase que define o recorte desta spec
> **A autenticação não é uma funcionalidade do TCC — é a condição para a única métrica de saída que o trabalho tem.** O quiz mede afinidade calculada; a lista de interesse mede **o que a pessoa realmente quis** depois de ler a recomendação. Sem conta, esse dado não existe, e o capítulo de avaliação da F4 volta a ser só "quantas pessoas responderam".

---

## 1️⃣ Estado real do código hoje

> [!note] Verificado no repositório, não nas notas

| Peça | Arquivo | Estado |
|---|---|---|
| `django.contrib.auth` instalado | `config/settings.py` | ✅ já está em `INSTALLED_APPS` |
| `AuthenticationMiddleware` + `SessionMiddleware` | `config/settings.py` | ✅ ativos |
| Context processor `auth` (o `{{ user }}` no template) | `config/settings.py` | ✅ ativo |
| Validadores de senha | `config/settings.py` | ✅ os 4 padrões |
| Usuários no banco | `auth_user` | **0 registros** — a janela do User customizado ainda está aberta |
| App `accounts` | — | ❌ não existe |
| Telas de login / cadastro / logout | — | ❌ não existem |
| Link de sessão no topo do site | `templates/base.html` | ❌ o header só tem a marca e uma tag |
| Dono da tentativa | `quiz/models.py::QuizAttempt` | ❌ só `respondent_name` (texto livre, opcional) |
| Modelo de interesse | — | ❌ não existe |
| Proteção da página de resultado | `quiz/web_views.py::result_page` | ⚠️ `get_object_or_404(QuizAttempt, pk=pk)` — **qualquer pk é público** |
| `LOGIN_URL`, `LOGIN_REDIRECT_URL` | `config/settings.py` | ❌ não definidos |

> [!success] Metade do trabalho já está paga
> `django.contrib.auth` inteiro já está ligado — sessão, hash de senha, validadores, `login_required`, `{{ user.is_authenticated }}` no template. **Nada disso precisa ser escrito.** O que falta é app, modelo de interesse, quatro telas e as regras de quem-vê-o-quê. É por isso que esta spec cabe em duas semanas sem tirar tempo do motor.

> [!warning] O `result_page` é um IDOR esperando dono
> Hoje `/resultado/7/` abre para qualquer visitante que digite o número. Enquanto tudo é anônimo, isso é no máximo um vazamento de "alguém tirou 96% em Eletricista". **No dia em que a tentativa passa a ter dono, vira dado pessoal ligado a uma pessoa identificada** — e a correção precisa entrar no mesmo PR que cria o vínculo, nunca depois. É a tarefa AUT-05.

---

## 2️⃣ As quatro decisões que esta spec propõe

> [!important] Precisam ser ratificadas antes de AUT-01
> Nenhuma delas trava as outras frentes. Todas travam esta.
> **Estado em 2026-08-25:** D2 ✅ ratificada · D1, D3 e D4 em aberto. A execução de AUT-01 a AUT-03 está escrita em [[passo-a-passo-aut-01-aut-03|🔑 passo a passo]], assumindo a **opção A** da D1 e com o Plano B marcado no lugar exato em que ele muda o código.

### D1 — User customizado agora, com login por e-mail

| Opção | Custo hoje | Custo depois |
|---|---|---|
| **A. `accounts.User(AbstractUser)` com login por e-mail** ⭐ | ~30 linhas + apagar o `db.sqlite3` e rodar os seeds de novo | zero |
| B. `django.contrib.auth.User` padrão (login por username) | zero | trocar o modelo de usuário depois é **migração de dados manual**, tabela por tabela |

**Recomendação: A.** O banco tem **0 usuários** e o resto dele é seed reproduzível por comando — o custo de trocar é literalmente `rm db.sqlite3 && migrate && seed_*`. Essa janela fecha no primeiro cadastro do piloto com respondentes reais.

```python
# accounts/models.py
class User(AbstractUser):
    username = None
    email = models.EmailField("e-mail", unique=True)

    USERNAME_FIELD = "email"
    REQUIRED_FIELDS = []

    objects = UserManager()   # ⚠️ obrigatório
```

> [!warning] A armadilha clássica do `username = None`
> Remover o `username` **quebra o `createsuperuser`** se o manager não for reescrito junto: o `UserManager` padrão do Django chama `create_user(username, email, password)`. São ~15 linhas de `BaseUserManager` com `create_user` e `create_superuser` normalizando o e-mail. Quem pular isso descobre no meio da defesa que não consegue criar o admin numa máquina nova.

> [!success] A decisão D1 é reversível de graça — e é isso que a torna segura
> Todo o resto desta spec referencia `settings.AUTH_USER_MODEL`, **nunca o modelo direto**. Se o grupo escolher B, muda uma linha no `settings.py` e o `accounts/models.py` some. Nenhuma outra tarefa, contrato ou teste desta spec muda uma vírgula.

### D2 — O quiz continua anônimo ✅ ratificada em 2026-08-25

**Recomendação: sim, e sem discussão.** Exigir conta antes das 6 perguntas mata a coleta com respondentes reais que alimenta o experimento da F2 e os dashboards da F4. O login aparece **uma única vez**: no clique de *"quero este curso"*, na tela de resultado.

```
visitante → quiz → resultado → [salvar na minha lista] → login/cadastro → salvo
   └── sem conta, vê tudo, sai quando quiser ────────┘
```

### D3 — Uma lista por pessoa, não múltiplas listas

**Recomendação: uma lista, plana.** "Criar coleções" é feature de produto maduro; aqui só adiciona um modelo, uma tela e zero valor para a banca. O que a lista precisa ter é **status por item** (`interessado · contatado · descartado`), porque é isso que a conversa sobre integração com o SGSET vai exigir.

### D4 — Salvar é sempre `POST`, e a intenção sobrevive ao login

**Recomendação: sim.** Nada de `/salvar/12/` em link `GET`. Quem clica sem estar logado tem a intenção guardada na **sessão** e, ao voltar do login, a tela de resultado mostra um banner *"você queria salvar Eletricista Instalador — salvar agora"* com um botão de `POST`.

> [!note] Por que não salvar automático depois do login
> Seria um clique a menos e uma mudança de estado disparada por um `GET` de redirect — o tipo de atalho que funciona e que ninguém consegue defender quando a banca pergunta sobre CSRF. O banner custa uma condição no template.

---

## 3️⃣ Os três contratos que esta spec congela

### 📄 C4 — `InterestItem` · o registro de interesse

```python
# accounts/models.py
class InterestItem(models.Model):
    STATUS = [
        ("interessado", "Interessado"),
        ("contatado", "Contatado"),
        ("descartado", "Descartado"),
    ]

    user = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE, related_name="interesses")
    course = models.ForeignKey("catalog.Course", on_delete=models.PROTECT, related_name="interessados")
    source_attempt = models.ForeignKey("quiz.QuizAttempt", null=True, blank=True, on_delete=models.SET_NULL, related_name="interesses")

    # foto do momento do clique — o curso e a engine mudam, o registro não
    score_snapshot = models.FloatField(null=True, blank=True)
    rank_snapshot = models.PositiveSmallIntegerField(null=True, blank=True)
    was_primary = models.BooleanField(default=False)

    status = models.CharField(max_length=20, choices=STATUS, default="interessado")
    note = models.TextField(blank=True)
    created_at = models.DateTimeField(auto_now_add=True)

    class Meta:
        unique_together = ("user", "course")
        ordering = ["-created_at"]
```

| Escolha | Por quê |
|---|---|
| `course` com `on_delete=PROTECT` | Interesse é **evidência institucional de demanda**. Apagar um curso não pode apagar em silêncio a prova de que 14 pessoas o queriam |
| `source_attempt` com `SET_NULL` | O interesse sobrevive ao apagamento da tentativa; a rastreabilidade é bônus, não requisito |
| `unique_together (user, course)` | O botão vira alternador (salvar/remover), não gerador de duplicata |
| `score_snapshot` / `rank_snapshot` / `was_primary` | **É aqui que a spec vira capítulo** — ver o quadro abaixo |

> [!success] Os três campos de snapshot são a razão de esta spec existir
> Com eles, três perguntas que hoje não têm resposta viram consulta de uma linha:
> 1. **Taxa de conversão** — de quem viu o resultado, quantos salvaram alguma coisa?
> 2. **Aderência à recomendação** — a pessoa salvou o **1º colocado** ou desceu para o 3º? (`rank_snapshot`)
> 3. **A IA melhorou a entrega?** — quando a LLM promoveu outro curso ao topo (`diverged=True` em [[camada-ia-plano-implementacao|🧩 camada de IA]]), a taxa de salvamento subiu ou caiu? (`was_primary` cruzado com `diverged`)
>
> A terceira é **a métrica de resultado do experimento da F2** — e ela não existe sem a lista de interesse. Sem esses campos, o TCC compara engine × LLM só por avaliação humana do próprio grupo; com eles, compara por **comportamento de usuário real**. É a diferença entre "achamos que ficou melhor" e um número.

### 📄 C5 — Rotas e quem pode entrar

| Método | Rota | Nome | Acesso |
|---|---|---|---|
| `GET` `POST` | `/entrar/` | `entrar` | público |
| `GET` `POST` | `/criar-conta/` | `criar-conta` | público |
| `POST` | `/sair/` | `sair` | logado (Django 5 **exige** POST) |
| `GET` | `/minha-lista/` | `interesse-lista` | logado |
| `POST` | `/minha-lista/salvar/` | `interesse-salvar` | logado · corpo: `course_id`, `attempt_id` |
| `POST` | `/minha-lista/<pk>/remover/` | `interesse-remover` | **dono do item** |
| `GET` | `/meus-testes/` | `minhas-tentativas` | logado |

> [!warning] `login_required` não é autorização
> `login_required` responde *"é alguém?"*. O que protege a lista é o **queryset filtrado pelo dono**: `InterestItem.objects.filter(user=request.user)`. Buscar por `pk` e só depois comparar o dono é o caminho pelo qual esse bug entra em todo projeto — e ele é exatamente o mesmo do `result_page` de hoje.

### 📄 C6 — Vínculo da tentativa anônima (o *claim*)

```
1. visitante responde o quiz
   └── web_views grava attempt e empilha o id em request.session["minhas_tentativas"]

2. clica "salvar na minha lista"  (POST, sem sessão de usuário)
   └── session["interesse_pendente"] = {"course_id": 12, "attempt_id": 34}
   └── redirect → /entrar/?next=/resultado/34/

3. faz login ou cria conta
   └── no login: para cada id em session["minhas_tentativas"],
       se attempt.user is None → attempt.user = request.user     ← o claim

4. volta em /resultado/34/
   └── banner: "você queria salvar Eletricista Instalador" [ Salvar agora ]  (POST)
```

> [!important] A regra que impede roubar tentativa alheia
> O claim só acontece para tentativas que **estão na sessão daquele navegador** *e* cujo `user` ainda é `None`. Sem a primeira condição, qualquer pessoa logada adotaria a tentativa dos outros mandando ids na mão. Sem a segunda, adotaria as já adotadas. As duas juntas, e o vínculo vira uma linha defensável.

#### Campo novo em `QuizAttempt`

```python
user = models.ForeignKey(settings.AUTH_USER_MODEL, null=True, blank=True,
                         on_delete=models.SET_NULL, related_name="tentativas")
```

`null=True` é o que preserva a D2: **tentativa sem dono continua sendo cidadã de primeira classe**. O `respondent_name` fica — vira o `first_name` pré-preenchido de quem está logado e continua servindo para quem não está.

---

## 4️⃣ Backlog — AUT-01 a AUT-12

| # | Tarefa | Arquivos | Depende | Pronto quando |
|---|---|---|---|---|
| **AUT-01** | App `accounts` + `User` + `UserManager` + `AUTH_USER_MODEL` + apagar e recriar o banco | `accounts/`, `config/settings.py` | **D1** | `createsuperuser` funciona com e-mail; `manage.py test` verde; seeds rodam |
| **AUT-02** | Telas de cadastro, login e logout (views nativas do Django + 3 templates) | `accounts/views.py`, `accounts/urls.py`, `templates/accounts/` | AUT-01 | Cria conta, sai e entra de novo sem tocar no `/admin` |
| **AUT-03** | Bloco de sessão no header + `LOGIN_URL` / `LOGIN_REDIRECT_URL` | `templates/base.html`, `config/settings.py` | AUT-02 | Deslogado vê "Entrar"; logado vê o nome e "Minha lista" |
| **AUT-04** | `QuizAttempt.user` + migration + `session["minhas_tentativas"]` | `quiz/models.py`, `quiz/web_views.py` | AUT-01 | Tentativa antiga (sem dono) abre no site sem erro |
| **AUT-05** | **Fechar o `result_page`**: dono, ou sessão, ou 404 | `quiz/web_views.py` | AUT-04 | Teste: usuário B recebe 404 no resultado do usuário A |
| **AUT-06** | Claim no login (contrato C6) | `accounts/views.py` ou signal `user_logged_in` | AUT-04 | Teste: responde anônimo, cria conta, a tentativa aparece em `/meus-testes/` |
| **AUT-07** | Modelo `InterestItem` + migration + admin (contrato C4) | `accounts/models.py`, `accounts/admin.py` | AUT-01 | Admin lista interesses com filtro por curso e por status |
| **AUT-08** | Salvar / remover interesse (POST, alternador, snapshots preenchidos) | `accounts/views.py`, `templates/quiz/result.html` | AUT-07 | Salvar duas vezes o mesmo curso não duplica; `rank_snapshot` gravado |
| **AUT-09** | Tela `/minha-lista/` + `/meus-testes/` | `templates/accounts/lista.html` | AUT-08 | Lista com nome, área, % do dia do clique e botão remover |
| **AUT-10** | Intenção pendente + banner pós-login (D4) | sessão + `result.html` | AUT-08 | Deslogado clica em salvar, cria conta, volta e conclui em 1 clique |
| **AUT-11** | Bateria de testes (seção 5) | `accounts/tests.py` | AUT-01 a AUT-10 | ~11 testes novos, todos verdes |
| **AUT-12** | Aviso de privacidade + correção do rodapé + parágrafo de LGPD | `templates/`, [[defesa-monografia-tcc\|🎤 defesa]] | AUT-02 | Nenhuma tela afirma algo falso sobre os dados |

> [!warning] AUT-12 não é burocracia — hoje o rodapé mente
> `templates/base.html` diz, em toda página: *"Nenhum dado é compartilhado"*. Estava correto enquanto tudo era anônimo. **No minuto em que existe cadastro com e-mail, o sistema passa a armazenar dado pessoal identificável** e essa frase vira uma declaração falsa exibida ao usuário — numa tela que a banca vai ver projetada. Trocar por "seus dados ficam nesta instituição e você pode apagar sua conta" custa dez minutos e vira parágrafo de LGPD na monografia.

> [!tip] O caminho mais curto até algo demonstrável são AUT-01, 02, 07, 08
> Com essas quatro, existe cadastro e existe lista. AUT-04/05/06 (vínculo e proteção) e AUT-09/10 (polimento da jornada) podem vir no PR seguinte — **exceto o AUT-05, que não pode atravessar o fim de semana depois do AUT-04**: é o único par onde adiar cria um bug de privacidade em vez de uma pendência de feature.

---

## 5️⃣ Testes — os ~11 que fecham a spec

| Teste | O que prova |
|---|---|
| `test_cadastro_cria_conta_e_loga` | O fluxo feliz existe |
| `test_email_duplicado_recusado` | `unique=True` chega na tela como mensagem, não como 500 |
| `test_createsuperuser_sem_username` | O `UserManager` foi reescrito (a armadilha da D1) |
| `test_quiz_anonimo_continua_funcionando` | **A D2 não foi quebrada por acidente** |
| `test_resultado_de_outro_usuario_da_404` | O IDOR do `result_page` está fechado |
| `test_claim_vincula_so_tentativa_da_sessao` | O contrato C6, no caminho feliz |
| `test_claim_nao_rouba_tentativa_com_dono` | O contrato C6, no caminho hostil |
| `test_salvar_interesse_exige_login` | `login_required` no lugar certo |
| `test_salvar_duas_vezes_nao_duplica` | `unique_together` + alternador |
| `test_snapshots_gravados_no_clique` | `score/rank/was_primary` preenchidos — sem eles não há métrica |
| `test_remover_item_de_outro_usuario_da_404` | Autorização por queryset, não por `pk` |

> [!success] Definition of Done da spec inteira
> `manage.py test` sai de ~22 testes (depois da [[spec-motor-e-ia-frentes-1-2|🧭 spec F1+F2]]) para **~33**, e a suíte continua rodando **offline e sem `.env`** — nada aqui toca rede.

---

## 6️⃣ Encaixe no plano de 5 semanas

| Semana | O que entra | Por que aí |
|---|---|---|
| **2** | AUT-01, AUT-02, AUT-03 | Conta de pé cedo, enquanto o motor está em migração e ninguém mais mexe no `settings.py` |
| **3** | AUT-04, AUT-05, AUT-06, AUT-07 | Vínculo + modelo prontos **antes** da semana 4 |
| **4** | AUT-08, AUT-09, AUT-10, AUT-12 | Bate com *"Registro de interesse"* da F3 e *"tela de interesse"* da F4 no cronograma atual |
| **5** | AUT-11 + a métrica de conversão no capítulo da F4 | Junto com os outros capítulos |

> [!warning] A dependência que ninguém vê até travar
> A [[divisao-de-trabalho-tcc|👥 divisão]] põe *"registro de interesse"* na **semana 4** da F3 — sem notar que ele **não existe sem conta de usuário**. Se a autenticação só começar na semana 4, a F3 perde a semana inteira esperando. **AUT-01 tem que sair na semana 2**, e é a única coisa desta spec com data crítica.

> [!important] AUT-01 troca o `AUTH_USER_MODEL` e manda todo mundo apagar o banco
> É uma mudança que **para as quatro frentes por dez minutos** — cada pessoa roda `rm db.sqlite3 && migrate && seed_areas && seed_courses && seed_questions`. Fazer isso na semana 2, com aviso no grupo, é um recado no WhatsApp. Fazer na semana 4, com o catálogo real da F3 já cadastrado à mão por alguém, é perder trabalho de gente. Se os 18 cursos reais forem cadastrados pelo admin em vez de por seed, **a janela da D1 fecha antes do prazo**.

---

## 7️⃣ Riscos

| Risco | Por que aparece | Mitigação |
|---|---|---|
| Trocar `AUTH_USER_MODEL` com dados no banco | Alguém cadastra o catálogo real antes da AUT-01 | Fazer AUT-01 na semana 2; catálogo real **sempre por seed**, nunca pelo admin |
| Login virar barreira e derrubar a coleta | Tentação de exigir conta antes do quiz | D2 escrita e testada (`test_quiz_anonimo_continua_funcionando`) |
| Dado pessoal sem base declarada | Cadastro com e-mail é dado identificável | AUT-12: aviso na tela, rodapé corrigido, botão de apagar conta, parágrafo na monografia |
| Vazamento de resultado alheio | `result_page` aberto hoje | AUT-05 no mesmo PR do AUT-04 |
| Senha fraca no piloto com colegas | Vão usar `123456` | Os 4 validadores padrão já estão ligados — **só não desligar** por conveniência de demo |
| A lista virar enfeite | Ninguém consulta o dado depois | Os snapshots do C4 + a métrica de conversão entrarem no capítulo da F4 desde o início |

---

## 8️⃣ O que fica de fora (e vira limitação declarada)

| Item | Por quê |
|---|---|
| Recuperação de senha por e-mail | Precisa de SMTP configurado. **Alternativa barata:** as views nativas do Django com `EMAIL_BACKEND` de console em desenvolvimento, declarado como "não configurado para produção" |
| Verificação de e-mail no cadastro | Mesma dependência de SMTP; no piloto com colegas não paga o custo |
| Login social (Google) | Dependência externa + `django-allauth`; contradiz *"o valor está na engine, não na infraestrutura"* |
| JWT / API pública autenticada | Sessão resolve o site inteiro. Se a F4 precisar da API para dashboards, `SessionAuthentication` + `IsAuthenticated` bastam |
| Perfil com foto, bio, preferências | Nenhuma pergunta da monografia depende disso |
| Papel de "atendente da instituição" que vê os interesses | **Tentador e fora de escopo** — o `/admin` com filtro por curso e status já entrega isso na defesa |

## ▶️ Próxima ação

**Ratificar D1 na próxima reunião** — é a única decisão com prazo de validade. Todas as outras podem ser tomadas com o código já andando; essa, não.

## 📎 Veja também

- [[escopo-fluxo-educmatch|🗺️ EducMatch — fluxo e recorte de escopo]] — onde o "registro de interesse" foi aprovado
- [[divisao-de-trabalho-tcc|👥 Divisão de trabalho]] — F3 é dona da regra, F4 da tela
- [[spec-motor-e-ia-frentes-1-2|🧭 Spec F1+F2]] — a spec irmã, mesmo formato
- [[camada-ia-plano-implementacao|🧩 Camada de IA]] — o `diverged` que a taxa de conversão vai cruzar
- [[modelagem-dados-quiz|🗃️ Modelagem de dados]] · [[front-templates-django|🎨 Front em templates]] · [[testes-e-validacao-tcc|✅ Testes]]
- [[defesa-monografia-tcc|🎤 Defesa e monografia]] · [[TCC|🎓 TCC]] · [[Projetos|🚀 Projetos]]
