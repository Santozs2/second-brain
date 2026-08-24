---
type: concept
area: Conceitos
status: estavel
difficulty: intermediate
id: tst-testes-django
category: Testes
tags:
  - testes
  - concept
  - backend
created: 2026-08-24
updated: 2026-08-24
---
# 🐍 Testes em Django

> O Django traz uma suíte completa embutida. Para a maioria dos projetos, não é preciso instalar nada.

---

## 🧱 As classes de teste

| Classe | Banco | Transação | Uso |
|---|---|---|---|
| `SimpleTestCase` | ❌ bloqueia acesso | — | Lógica pura, utilitários |
| `TestCase` | ✅ | Rollback ao fim | **O padrão** |
| `TransactionTestCase` | ✅ | Commit real, TRUNCATE ao fim | Testar transações |
| `LiveServerTestCase` | ✅ | Sobe servidor real | Selenium / E2E |

```python
from django.test import TestCase, SimpleTestCase

class TestEngine(SimpleTestCase):
    """Sem banco — roda em milissegundos."""
    def test_cosseno_identico(self):
        self.assertAlmostEqual(cosine_similarity([1, 2], [1, 2]), 1.0)
```

> [!tip] Use `SimpleTestCase` para tudo que não toca o banco
> `TestCase` cria e destrói transação a cada teste. Em uma suíte com centenas de testes de lógica pura, isso é a diferença entre 2 segundos e 40. E o bloqueio de acesso ao banco é um recurso: se o teste falhar por tentar consultar, você descobriu um acoplamento indevido.

---

## 🗄️ Fixtures e dados de teste

```python
class TestRecomendacao(TestCase):

    @classmethod
    def setUpTestData(cls):
        """Roda UMA vez para a classe inteira — muito mais rápido."""
        cls.area = Area.objects.create(nome="Mecânica", slug="mecanica")
        cls.curso = Course.objects.create(nome="Mecânica Industrial")
        CourseAreaWeight.objects.create(course=cls.curso, area=cls.area, peso=5)

    def setUp(self):
        """Roda antes de CADA teste — use só para o que muda."""
        self.client = Client()
```

> [!warning] `setUpTestData` versus `setUp`: a diferença é de performance e de isolamento
> `setUpTestData` roda uma vez por classe; os objetos são recarregados do banco a cada teste dentro de uma transação que sofre rollback. Mais rápido, e seguro **desde que você não mute os objetos em memória** esperando isolamento. Objetos mutados em atributo de classe vazam entre testes.

---

## 🔌 Testar API com DRF

```python
from rest_framework.test import APITestCase
from rest_framework import status

class TestQuizAPI(APITestCase):

    def test_listar_perguntas(self):
        resp = self.client.get("/api/quiz/perguntas/")
        self.assertEqual(resp.status_code, status.HTTP_200_OK)
        self.assertEqual(len(resp.data), 6)

    def test_enviar_respostas_devolve_ranking(self):
        payload = {"respostas": [{"pergunta": 1, "valor": 5}]}
        resp = self.client.post("/api/quiz/respostas/", payload, format="json")
        self.assertEqual(resp.status_code, status.HTTP_201_CREATED)
        self.assertIn("ranking", resp.data)

    def test_endpoint_exige_autenticacao(self):
        self.client.force_authenticate(user=None)
        resp = self.client.get("/api/protegido/")
        self.assertEqual(resp.status_code, status.HTTP_401_UNAUTHORIZED)
```

---

## 🏢 Testar isolamento multi-tenant

O teste mais importante em sistema multi-inquilino: provar que um tenant **não** enxerga dados de outro.

```python
class TestIsolamento(APITestCase):

    def setUp(self):
        self.org_a = Organization.objects.create(nome="A")
        self.org_b = Organization.objects.create(nome="B")
        self.recurso_b = Recurso.objects.create(org=self.org_b, nome="secreto")
        self.user_a = criar_usuario(org=self.org_a)
        self.client.force_authenticate(self.user_a)

    def test_nao_lista_recurso_de_outra_org(self):
        resp = self.client.get("/api/recursos/")
        ids = [r["id"] for r in resp.data]
        self.assertNotIn(self.recurso_b.id, ids)

    def test_nao_acessa_recurso_de_outra_org_por_id(self):
        """Teste de invasão direta — o mais importante dos dois."""
        resp = self.client.get(f"/api/recursos/{self.recurso_b.id}/")
        self.assertEqual(resp.status_code, status.HTTP_404_NOT_FOUND)
```

> [!important] Retorne 404, não 403, para recurso de outro tenant
> `403 Forbidden` confirma que o recurso existe — é vazamento de informação por canal lateral. `404` não distingue "não existe" de "não é seu". Este detalhe costuma ser exatamente o que uma auditoria de segurança procura.

---

## ⚡ Executar a suíte

```bash
python manage.py test                          # tudo
python manage.py test quiz                     # um app
python manage.py test quiz.tests.TestEngine    # uma classe
python manage.py test --parallel               # em paralelo
python manage.py test --keepdb                 # reaproveita o banco de teste
python manage.py test --failfast               # para no primeiro erro
```

> [!tip] `--keepdb` e `--parallel` juntos mudam a experiência
> Recriar o banco a cada execução domina o tempo em projetos com muitas migrations. `--keepdb` reaproveita; `--parallel` distribui entre os núcleos. Uma suíte que levava um minuto passa a levar segundos — e suíte rápida é suíte que se roda.

---

## ⚙️ Configuração para teste

```python
# settings de teste — banco em memória acelera muito
if "test" in sys.argv:
    DATABASES["default"] = {
        "ENGINE": "django.db.backends.sqlite3",
        "NAME": ":memory:",
    }
    PASSWORD_HASHERS = ["django.contrib.auth.hashers.MD5PasswordHasher"]
    CELERY_TASK_ALWAYS_EAGER = True
```

O hasher MD5 é seguro **apenas em teste**: reduz o custo de criar usuários de dezenas de milissegundos para quase zero.

---

## ✅ Asserções específicas do Django

```python
self.assertContains(resp, "Mecânica")           # status 200 + texto presente
self.assertRedirects(resp, "/resultado/")
self.assertTemplateUsed(resp, "quiz/result.html")
self.assertQuerySetEqual(qs, [...])
self.assertNumQueries(3)                        # detecta problema N+1
```

> [!success] `assertNumQueries` é a defesa contra o problema N+1
> Envolver uma view em `with self.assertNumQueries(3):` transforma uma regressão de performance em teste vermelho. É a forma mais barata de impedir que um `select_related` esquecido vire 200 consultas em produção. Ver [[dad-otimizacao-consultas|⚡ Otimização de consultas]].

---

## 🔗 Conceitos relacionados

- [[tst-piramide-de-testes|🔺 Pirâmide de testes]] · [[tst-mocks-e-dubles|🎭 Mocks e dublês]]
- [[tst-cobertura|📊 Cobertura]] · [[Django|Django]] · [[Django REST Framework|DRF]]
- [[dad-otimizacao-consultas|⚡ Otimização de consultas]]

## Veja também

- [[Conceitos|🧠 Conceitos]]

---

**Status:** ✅ Completo
