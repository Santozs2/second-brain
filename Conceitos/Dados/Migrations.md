---
type: concept
area: Conceitos
status: estavel
tags:
  - concept
created: 2026-06-30
updated: 2026-08-24
---
# Migrations

## Definição

Histórico versionado de alterações no esquema do banco de dados, gerado automaticamente a partir das mudanças nos [[Models|Models]].

Cada migration é um arquivo Python que descreve uma transformação do esquema, com dependência declarada da anterior — formando um grafo dirigido que o Django percorre para levar o banco do estado atual ao desejado.

## Quando usar

Sempre que um Model muda (novo campo, nova tabela, relacionamento), gera-se uma migration e ela é aplicada ao banco — em vez de alterar tabelas manualmente.

## Comandos

```bash
python manage.py makemigrations              # gera a partir dos models
python manage.py makemigrations --dry-run -v3  # mostra sem escrever
python manage.py migrate                     # aplica as pendentes
python manage.py showmigrations              # o que já foi aplicado
python manage.py sqlmigrate quiz 0002        # mostra o SQL gerado
python manage.py migrate quiz 0001           # volta para a 0001
```

> [!tip] `sqlmigrate` antes de aplicar em produção
> Ele imprime o SQL exato sem executar nada. Em tabela grande, ver um `ALTER TABLE` que reescreve a tabela inteira antes de rodá-lo é a diferença entre um deploy tranquilo e uma indisponibilidade de vinte minutos.

## Boas práticas

- **Nunca editar uma migration já aplicada em produção** — o Django registra o que rodou; alterar o arquivo cria divergência silenciosa entre o esquema real e o esperado
- **Revisar a migration gerada antes de aplicar**
- Versionar as migrations no Git, sempre — elas fazem parte do código
- Nomear com intenção: `python manage.py makemigrations -n adiciona_peso_curso`
- Uma migration por mudança lógica, não uma por semana de trabalho

## Migrations de dados

Além do esquema, migrations podem transformar dados existentes.

```python
from django.db import migrations

def preencher_slugs(apps, schema_editor):
    # Use apps.get_model — NUNCA importe o model diretamente
    Course = apps.get_model("catalog", "Course")
    for c in Course.objects.filter(slug=""):
        c.slug = slugify(c.nome)
        c.save(update_fields=["slug"])

def reverter(apps, schema_editor):
    apps.get_model("catalog", "Course").objects.update(slug="")

class Migration(migrations.Migration):
    dependencies = [("catalog", "0003_course_slug")]
    operations = [migrations.RunPython(preencher_slugs, reverter)]
```

> [!warning] Importar o model diretamente em uma migration é um bug esperando o futuro
> `from catalog.models import Course` traz a versão **atual** do model. Quando alguém rodar essa migration meses depois, o model terá campos que não existiam na época — e a migration quebra. `apps.get_model()` devolve a versão **histórica**, congelada naquele ponto do grafo.

## O deploy em duas fases

O padrão que evita indisponibilidade quando esquema e código mudam juntos:

```
Fase 1 — migração compatível com as duas versões
         (adiciona coluna nullable; NÃO remove nada)
              ↓
Fase 2 — deploy do código novo
              ↓
Fase 3 — migração de limpeza (remove o antigo), em deploy posterior
```

> [!important] Remover coluna no mesmo deploy do código é a receita do erro 500
> Durante o deploy existe um intervalo em que código antigo e esquema novo coexistem. Se a coluna já foi removida e o código antigo ainda a consulta, toda requisição falha. **Adicionar é seguro; remover exige uma etapa separada.**

## Problemas comuns

| Problema | Causa | Solução |
|---|---|---|
| Migrations conflitantes | Duas branches criaram a mesma numeração | `makemigrations --merge` |
| `InconsistentMigrationHistory` | Ordem de aplicação divergente | Conferir `showmigrations`; corrigir dependência |
| Migration não detectada | App fora de `INSTALLED_APPS` | Registrar o app |
| Trocar `AUTH_USER_MODEL` depois | Migrations já aplicadas | Muito trabalhoso — defina **antes** da primeira `migrate` |
| Campo não-nulo sem default | Linhas existentes ficariam inválidas | Adicionar com `null=True`, preencher, depois tornar obrigatório |

> [!warning] `AUTH_USER_MODEL` é a decisão mais cara de adiar em um projeto Django
> Trocar o model de usuário depois de aplicar a primeira migration exige recriar o banco ou uma sequência delicada de migrations manuais. **Se há qualquer chance de você querer um `User` customizado, crie-o no primeiro dia** — mesmo que idêntico ao padrão.

## Squashing

Depois de dezenas de migrations, a aplicação em banco novo fica lenta.

```bash
python manage.py squashmigrations quiz 0001 0042
```

Consolida o intervalo em uma migration só, mantendo as antigas como referência até que todos os ambientes tenham passado do ponto.

## Conceitos relacionados

- [[Models|Models]] · [[ORM|ORM]] · [[dad-modelagem-relacional|🗺️ Modelagem relacional]]
- [[dad-transacoes-acid|🔒 Transações e ACID]] · [[CI-CD|CI/CD]]
- [[Banco de Dados|Banco de Dados]]

## Veja também

- [[Documentações|Documentações]]
