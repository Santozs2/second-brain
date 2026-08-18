---
title: "Defesa e monografia — argumentos, roadmap e pendências"
aliases: ["Defesa do TCC", "Monografia TCC", "Banca TCC"]
tags: [tcc, monografia, defesa, roadmap, apresentacao]
status: em-andamento
projeto: TCC
criado: 2026-08-17
---

> [!info] Projeto: [[TCC|🎓 TCC]] · Base técnica: [[engine-matching-cosseno|🧮 Engine]] · [[modelagem-dados-quiz|🗃️ Modelagem]]

# 🎤 Defesa e monografia

> [!abstract] Para que serve esta nota
> Consolidar as respostas para as perguntas que a banca provavelmente vai fazer, e listar o que ainda falta. Cada argumento aqui tem **evidência no código** — teste, decisão de modelagem ou resultado medido.

## 🎯 A tese em uma frase

> Um sistema de recomendação de cursos baseado em **similaridade vetorial** entrega resultados melhores e **explicáveis** em comparação a regras condicionais fixas, e permanece correto quando o catálogo cresce — sem precisar reescrever a lógica.

## ❓ Perguntas prováveis da banca

### "Por que não usar `if`? Seria mais simples."

Com 12 cursos e 7 áreas, uma árvore de regras precisaria cobrir as combinações de perfil na mão. **Cada curso novo exigiria reescrever a lógica.** Na abordagem vetorial, cadastrar um curso no admin com seus pesos já o coloca no ranking — a regra é o cálculo, não o código.
E o `if` não sabe responder *"por quê"*: o cosseno entrega a contribuição de cada área.

### "Por que cosseno e não somar os pontos?"

Porque a soma bruta premia o **curso gordo**. "Automação Industrial" pesa 5 áreas; "Costura Industrial" pesa 2. Numa soma, Automação venceria até em perfil 100% costura, só por ter mais termos. Dividir pelas normas neutraliza o tamanho e sobra a direção — a afinidade real.
**Evidência:** `test_escala_nao_muda_o_score`, que quebra se alguém trocar o cosseno por soma.

### "Como sei que o resultado está certo?"

Quatro critérios de aceite viraram testes automatizados. O decisivo: **perfil automotivo + elétrico coloca "Injeção Eletrônica" (0.968) acima de "Motores a Combustão" (0.891)** — resultado que só emerge se o sistema entender combinação de áreas. Nenhuma regra foi escrita para esse caso.

### "E se dois cursos empatarem?"

Desempate determinístico por nome (`(-score, name)`). Sem isso, a mesma entrada poderia gerar ordens diferentes entre execuções. **Evidência:** `test_resultado_e_reprodutivel`.

### "O sistema é uma caixa-preta?"

Não. Cada recomendação guarda um `explanation` com as três áreas que mais contribuíram e o percentual de cada uma, exibido na tela sob *"por que este curso"*. **Recomendação explicável** é o diferencial do trabalho.

### "Como validou sem usuários reais?"

Perfis simulados representando arquétipos (puro elétrico, misto automotivo-elétrico, costura, TI), executados por comando com rollback. É validação de **corretude do algoritmo**; validação com usuários reais está no roadmap.

### "Por que SQLite?"

O objeto de estudo é o algoritmo de recomendação, não a infraestrutura. O ORM abstrai o banco: trocar por PostgreSQL é mudar `DATABASES` no settings. Escolha consciente de escopo.

## 🏛️ Argumentos de arquitetura

| Decisão | Argumento na monografia | Evidência |
|---|---|---|
| Funções puras separadas do ORM | Algoritmo testável sem banco e independente do framework | `SimpleTestCase` roda 4 testes sem criar tabela |
| Pesos em tabela relacional | Integridade referencial e consulta; JSON perderia as duas | FK + `unique_together` |
| Recomendação persistida | Reprodutibilidade e histórico analisável | `Recommendation` com `explanation` congelado |
| Seeds idempotentes | Base reconstruível em um comando; vira fixture dos testes | `update_or_create` + `exclude().delete()` |
| Duas portas (HTML e JSON) | Regra de negócio isolada da apresentação | Ambas chamam `recommend()` |
| `prefetch_related` + `bulk_create` | ~40 consultas → 3 | Otimização de acesso a dados |

## 📚 Estrutura sugerida da monografia

1. **Introdução** — o problema de escolher curso técnico sem orientação
2. **Referencial teórico** — sistemas de recomendação, modelo de espaço vetorial, similaridade de cosseno ([[cs-linear-algebra|álgebra linear]]), recomendação explicável
3. **Metodologia** — modelagem de áreas como dimensões; construção dos pesos; instrumento de coleta (o quiz)
4. **Desenvolvimento** — arquitetura Django, engine, API e interface
5. **Resultados** — os 4 perfis validados, tabela de scores, análise dos pares híbridos
6. **Considerações finais** — limitações e trabalhos futuros

> [!tip] Figuras que valem a pena
> Diagrama do modelo de dados ([[modelagem-dados-quiz|🗃️]]), representação gráfica de dois vetores e o ângulo entre eles, tabela de pesos do catálogo ([[catalogo-areas-e-cursos|📚]]) e captura da tela de resultado com o "por que este curso".

## ⚠️ Limitações a declarar (antes que a banca aponte)

- **Pesos definidos por julgamento do autor**, não por validação estatística com especialistas.
- **Amostra de validação sintética** — perfis simulados, não respondentes reais.
- **Catálogo reduzido** (12 cursos) e cadastrado manualmente.
- **Sem aprendizado**: o sistema não se ajusta a partir de escolhas anteriores; a recomendação é determinística.
- **Sem tratamento de perfis ambíguos**: um perfil igualmente distribuído entre 7 áreas recebe ranking apertado, sem sinalizar baixa confiança.

> [!note] Declarar limitação é força, não fraqueza
> A banca vai encontrá-las de qualquer forma. Antecipá-las com o motivo ("fora do escopo por X") demonstra domínio do próprio trabalho.

## 🚀 Roadmap

- [ ] **Scraping do catálogo real** — alimenta só o `catalog`, sem tocar no quiz (o tema original do TCC)
- [ ] **Histórico de tentativas** — tela listando respostas anteriores e seus rankings
- [ ] **Validação com respondentes reais** — comparar a recomendação com a escolha declarada
- [ ] **Indicador de confiança** — sinalizar quando o perfil for indeciso (scores muito próximos)
- [ ] **Painel de calibração** — visualizar o impacto de mudar um peso antes de salvar
- [ ] **Deploy** para demonstração fora da máquina local

## 🎬 Roteiro de demonstração ao vivo

1. Abrir `/` e responder o quiz como perfil misto automotivo + elétrico.
2. Mostrar o resultado destacando o *"por que este curso"*.
3. Abrir o admin e mostrar os pesos de "Injeção Eletrônica".
4. Rodar `manage.py test_engine` no terminal — os 4 perfis com scores.
5. Rodar `manage.py test` — 12 testes, OK.

> [!warning] Preparação
> Rodar em `runserver 8010` (a 8000 costuma estar ocupada) e conferir os seeds antes. Ter o terminal já no diretório, com `PYTHONIOENCODING=utf-8`.

## Veja também

- [[TCC|🎓 TCC]]
- [[engine-matching-cosseno|🧮 Engine de matching]]
- [[testes-e-validacao-tcc|✅ Testes e validação]]
