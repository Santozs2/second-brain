---
title: "Artigo — como escrever a seção do cálculo de cosseno"
aliases: ["Seção de metodologia do TCC", "Cálculo de cosseno no artigo", "Protótipo da metodologia"]
tags: [tcc, monografia, artigo, metodologia, cosseno, escrita-cientifica]
status: em-andamento
projeto: TCC
criado: 2026-08-20
---

> [!info] Projeto: [[TCC|🎓 TCC]] · Código: [[engine-matching-cosseno|🧮 Engine de matching]] · Conceitos: [[cs-linear-algebra|Álgebra linear]]

# ✍️ Como o cálculo de cosseno vira seção de artigo científico

> [!abstract] O que é esta nota
> Protótipo da **seção de metodologia** da monografia. A nota [[engine-matching-cosseno|🧮 Engine de matching]] documenta o *código*; [[fundamentacao-teorica-recomendacao|📖 Fundamentação teórica]] responde *por que* cosseno; esta documenta como transformar o cálculo em **texto científico** — formalização, exemplo verificável, comparação com alternativas e limitações.
> Todos os números foram recalculados a partir dos seeds reais: **7 áreas, 12 cursos, 6 perguntas**.

> [!warning] A diferença que importa
> Artigo científico **não descreve código, descreve um modelo** — e depois demonstra que o código o implementa. Se a seção começar com `def cosine_similarity(a, b)`, está errada. Começa com o espaço vetorial.

## 🧭 Os seis movimentos, nesta ordem

| # | Movimento | Pergunta que responde |
|---|---|---|
| 1 | Definir o espaço | Sobre *o quê* os vetores estão definidos? |
| 2 | Definir os vetores | Como perfil e curso viram números? |
| 3 | Definir a medida | Qual a fórmula, e de onde ela vem? |
| 4 | Exemplificar | O leitor consegue refazer a conta? |
| 5 | Tratar o degenerado | O que acontece quando falha? |
| 6 | Verificar | Como se sabe que está certo? |

> [!tip] O erro mais comum
> Inverter 1 e 3 — apresentar a fórmula antes de dizer sobre qual espaço ela opera. A fórmula do cosseno é domínio público; **o que é autoral no trabalho é a construção do espaço vetorial de áreas profissionais.** Comece por ela.

---

## 1️⃣ Formalização do espaço

> [!quote] Texto para o artigo
> Seja $A = \{a_1, \ldots, a_n\}$ o conjunto finito de áreas profissionais que estrutura o domínio. No sistema desenvolvido, $n = 7$. Cada área é identificada por uma chave textual estável (*slug*), que funciona como índice da dimensão correspondente no espaço $\mathbb{R}^n_{\geq 0}$. Adota-se a restrição $w \in [0,5]$ para todo peso, faixa validada na camada de persistência.

> [!important] Diga "não negativos" explicitamente
> $\mathbb{R}^n_{\geq 0}$ é o que garante, mais adiante, escore em $[0,1]$ — e não em $[-1,1]$. Sem estabelecer isso aqui, a seção da medida fica com um buraco lógico que a banca cobra.

**Vetor de curso:** $\mathbf{c} = (c_1, \ldots, c_n)$, com $c_j$ atribuído por curadoria pedagógica. Peso nulo = ausência de relação, não armazenado (representação esparsa).

**Vetor de perfil:** soma vetorial das alternativas marcadas.

$$\mathbf{p} = \sum_{e \in E} \mathbf{e} \qquad\Longleftrightarrow\qquad p_j = \sum_{e \in E} e_j \tag{1}$$

> [!note] Uma linha que fecha uma porta
> Escreva que a soma é comutativa, logo **a ordem das respostas não afeta o perfil**. Parece supérfluo, mas antecipa a pergunta "e se responder na ordem inversa?" ao custo de uma frase.

---

## 2️⃣ A medida e suas três propriedades

$$\operatorname{sim}(\mathbf{p}, \mathbf{c}) = \cos\theta = \frac{\mathbf{p} \cdot \mathbf{c}}{\|\mathbf{p}\|\,\|\mathbf{c}\|} = \frac{\sum_j p_j c_j}{\sqrt{\sum_j p_j^2}\;\sqrt{\sum_j c_j^2}} \tag{2}$$

Origem: identidade do produto interno $\mathbf{x}^\top\mathbf{y} = \|\mathbf{x}\|\|\mathbf{y}\|\cos\theta$ (GOODFELLOW; BENGIO; COURVILLE, 2016, cap. 2).

**Propriedade 1 — invariância à escala.** $\operatorname{sim}(\alpha\mathbf{p}, \mathbf{c}) = \operatorname{sim}(\mathbf{p}, \mathbf{c})$ para $\alpha > 0$, pois $\|\alpha\mathbf{p}\| = \alpha\|\mathbf{p}\|$ cancela o $\alpha$ do numerador. Essencial aqui: o perfil soma 6 respostas e chega a magnitudes 5× maiores que as dos cursos, presos a $[0,5]$.

**Propriedade 2 — contradomínio.** Como todos os pesos são não negativos e vale Cauchy–Schwarz:

$$0 \leq \operatorname{sim}(\mathbf{p}, \mathbf{c}) \leq 1 \tag{3}$$

**Propriedade 3 — esparsidade.** Só dimensões compartilhadas contribuem para o numerador.

> [!success] Por que enunciar como "propriedades"
> Com prova de uma linha cada, o registro do texto muda de *descrição de software* para *argumentação matemática*. A Propriedade 2 é o que autoriza escrever "88% de compatibilidade" na tela — sem ela, esse número não tem fundamento.

---

## 3️⃣ O exemplo numérico — a seção que a banca confere

> [!danger] Não pule esta parte
> É a única seção em que a banca pode te conferir. Escolha **um** perfil, mostre **todas** as contas e apresente o ranking completo. Um exemplo verificável vale mais que três parágrafos de explicação.

Candidato de inclinação elétrica, respondendo **as 6 questões**:

| # | Alternativa | Pesos |
|---|---|---|
| 1 | Montar e ligar coisas que funcionam na hora | elétrica 4; eletromec. 3 |
| 2 | Chão de fábrica, indo até as máquinas | eletromec. 4; elétrica 3; mecânica 2 |
| 3 | Eletricidade, circuitos e comandos | elétrica 5; eletromec. 3 |
| 4 | Testo a parte elétrica com um multímetro | elétrica 4; eletromec. 3 |
| 5 | Manter a indústria funcionando sem parar | eletromec. 5; elétrica 3; mecânica 2 |
| 6 | Multímetro e alicate | elétrica 5; eletromec. 2 |

$$\mathbf{p} = (24_{\text{elétrica}},\; 20_{\text{eletromec.}},\; 4_{\text{mecânica}},\; 0, 0, 0, 0)$$
$$\|\mathbf{p}\| = \sqrt{576 + 400 + 16} = \sqrt{992} \approx 31{,}496$$

Contra *Comandos Elétricos e CLP*, $\mathbf{c} = (5, 4, 1, 1, 0, 0, 0)$:

$$\mathbf{p}\cdot\mathbf{c} = 120 + 80 + 4 = 204 \qquad \|\mathbf{c}\| = \sqrt{43} \approx 6{,}557$$
$$\operatorname{sim} = \frac{204}{206{,}53} \approx 0{,}9877 \;\rightarrow\; 99\%$$

### Ranking completo do catálogo

| Pos. | Curso | $\mathbf{p}\cdot\mathbf{c}$ | $\|\mathbf{c}\|$ | sim | Tela |
|---:|---|---:|---:|---:|---:|
| 1 | Comandos Elétricos e CLP | 204,0 | 6,557 | 0,9877 | 99% |
| 2 | Eletricista Instalador | 184,0 | 5,916 | 0,9875 | 99% |
| 3 | **Manutenção Eletromecânica Industrial** | **212,0** | 7,681 | 0,8763 | 88% |
| 4 | Automação Industrial | 204,0 | 7,616 | 0,8505 | 85% |
| 5 | Injeção Eletrônica Automotiva | 120,0 | 6,557 | 0,5810 | 58% |
| 6 | Usinagem CNC | 84,0 | 5,916 | 0,4508 | 45% |
| 7 | Redes e Infraestrutura de TI | 68,0 | 5,477 | 0,3942 | 39% |
| 8 | Mecânica de Motores a Combustão | 32,0 | 5,916 | 0,1717 | 17% |
| 9 | Costura Industrial | 4,0 | 5,099 | 0,0249 | 2% |
| 10 | Modelagem e Corte de Vestuário | 0,0 | 5,099 | 0,0000 | 0% |
| 11 | Python para Análise de Dados | 0,0 | 6,403 | 0,0000 | 0% |
| 12 | Fundamentos de IA | 0,0 | 6,403 | 0,0000 | 0% |

*O sistema persiste só as 5 primeiras; as demais entram na tabela para análise.*

> [!success] O parágrafo mais valioso do capítulo
> **Manutenção Eletromecânica tem o MAIOR produto interno do catálogo (212) e mesmo assim cai para 3º lugar.** Por espalhar pesos altos em 5 áreas, sua norma (7,681) o penaliza: é um curso generalista, cuja direção não coincide com a de um perfil concentrado em 2 áreas.
>
> Isso é a evidência, **nos seus próprios dados**, de que a normalização faz algo. Transforma a justificativa de "a literatura recomenda" em "eis a prova no meu domínio". Se a banca perguntar por que não usar só o produto interno, a resposta já está impressa.

> [!warning] Os números daqui ≠ os números de [[engine-matching-cosseno|🧮 Engine de matching]] — e está certo
> Aquela nota registra 0,995 / 0,985 com **Eletricista em 1º**. Vêm do `test_engine.py`, que marca **4 alternativas**. Esta usa **as 6**, como o formulário web exige.
>
> Com 6 respostas a ordem entre os dois primeiros **inverte** (0,9877 vs. 0,9875 — diferença de $2\times10^{-4}$). Ambos os cálculos estão corretos; são entradas diferentes. **Para o artigo, use o perfil de 6 respostas**: é o único que um usuário real consegue submeter pelo site.
>
> Consequência prática: `test_perfil_eletrico` afirma `ranking[0] == "Eletricista Instalador"` sobre um perfil de 4 respostas que o formulário nunca aceitaria. O teste não está errado, mas testa um caminho que só a API permite — vale registrar em [[testes-e-validacao-tcc|✅ Testes]].

---

## 4️⃣ A explicação é decomposição exata, não aproximação

$$\mathbf{p}\cdot\mathbf{c} = \sum_j \underbrace{p_j c_j}_{\text{contribuição de } a_j} \tag{4} \qquad\qquad \rho_j = \frac{p_j c_j}{\sum_k p_k c_k} \times 100\% \tag{5}$$

| Área | $p_j$ | $c_j$ | $p_j c_j$ | $\rho_j$ |
|---|---:|---:|---:|---:|
| Elétrica | 24 | 5 | 120,0 | 58,8% |
| Eletromecânica | 20 | 4 | 80,0 | 39,2% |
| Mecânica | 4 | 1 | 4,0 | 2,0% |
| TI | 0 | 1 | 0,0 | 0,0% |
| **Total** | — | — | **204,0** | **100%** |

> [!success] A frase que é o diferencial do trabalho
> *"A explicação não é uma reconstrução aproximada do escore, mas sua decomposição exata: a mesma operação que produz a ordenação produz a justificativa."*
>
> Em IA, explicabilidade se obtém **depois**, com técnicas separadas (LIME, SHAP). Aqui é gratuita, porque a medida é linear no numerador. Vale um parágrafo na conclusão também.

> [!danger] Armadilha de redação
> $\rho_j$ decompõe o **numerador**, não o escore final — a norma não é decomponível por área. Escrever "elétrica responde por 58,8% do escore" está tecnicamente errado. Escreva "da afinidade bruta" ou "do produto interno". Banca de exatas pega isso.

---

## 5️⃣ Descartar a euclidiana com dado, não com citação

Mesmo perfil, ordenado por $d(\mathbf{p},\mathbf{c}) = \|\mathbf{p}-\mathbf{c}\|$:

| Pos. | Curso | $d$ | Pos. por cosseno |
|---:|---|---:|---:|
| 1 | Comandos Elétricos e CLP | 25,040 | 1 |
| 1 | Manutenção Eletromecânica Industrial | 25,040 | 3 |
| 3 | Automação Industrial | 25,338 | 4 |
| 4 | Eletricista Instalador | 25,671 | **2** |
| 5 | Injeção Eletrônica Automotiva | 28,196 | 5 |
| 6 | Usinagem CNC | 29,309 | 6 |
| 7 | Redes e Infraestrutura de TI | 29,766 | 7 |
| 8 | Mecânica de Motores a Combustão | 31,032 | 8 |
| 9 | Costura Industrial | 31,780 | 9 |
| 10 | Modelagem e Corte de Vestuário | 31,906 | 10 |
| 11 | Fundamentos de IA | 32,140 | 11 |
| 11 | Python para Análise de Dados | 32,140 | 11 |

**Três patologias, com número:**

1. **Contradomínio comprimido** — as 12 distâncias cabem em $[25{,}0;\ 32{,}1]$, porque $\|\mathbf{p}\| \approx 31{,}5$ domina tudo. Mede magnitude do perfil, não orientação.
2. **Perda de discriminação** — *Motores a Combustão* (1 área em comum) dista 31,032; *Modelagem e Corte* (nenhuma área em comum) dista 31,906. Diferença de 2,8%, contra 17 pontos percentuais que o cosseno atribui aos mesmos dois.
3. **Empates espúrios** — cursos de conteúdo distinto recebem distâncias idênticas (25,040 e 32,140), tornando a ordenação arbitrária.

> [!tip] A estrutura do argumento
> Enuncia a alternativa → cita a literatura → **testa no próprio dado** → nomeia as patologias com número. Descartar só por citação ("a literatura diz que é pior") convida a pergunta "e no seu caso, você testou?".

---

## 6️⃣ Casos-limite — escrever "o sistema trata", não "poderia tratar"

| Caso | Tratamento | Justificativa para o texto |
|---|---|---|
| **Vetor nulo** | convenção $\operatorname{sim}(\mathbf{0}, \mathbf{c}) = 0$ | ausência de informação não deve gerar afinidade |
| **Empates** | desempate alfabético por nome | reprodutibilidade é requisito: mesma entrada, mesma saída |
| **Precisão** | 4 casas persistidas, inteiro na tela | pesos são ordinais de curadoria, não medidas |
| **Áreas ausentes** | chave ausente = zero | esparso equivale ao denso, sem condicional no cálculo |

> [!note] Onde tratar o empate visual 99% / 99%
> A tabela do ranking expõe dois cursos exibidos como 99%. **Traga você mesmo**, no parágrafo de precisão: os pesos de origem são atribuições ordinais, então 4 casas decimais de escore são precisão espúria em relação à precisão da entrada. É um argumento de metrologia, e é correto.

---

## 7️⃣ Custo, verificação e algoritmo

$$O\big(m \cdot \eta(\mathbf{p})\big) + O(m \log m) \tag{6}$$

com $\eta(\mathbf{v})$ = componentes não nulas, $m = 12$ cursos, $\eta(\mathbf{p}) \leq n = 7$.

> [!tip] Emoldure ou soará pretensioso
> Com 12 cursos, complexidade assintótica precisa de moldura: *"o dado importa não pelo desempenho atual, mas pela escalabilidade — como $\eta(\mathbf{p})$ é limitado por $n$, expandir o catálogo mantém o custo linear em $m$."*

> [!question] Otimização identificada e não implementada
> `cosine_similarity()` recalcula `norm(a)` — a norma do **perfil** — a cada curso: 12 cálculos idênticos por tentativa. Pré-computar a norma do perfil uma vez, e as dos cursos no cadastro, elimina a redundância **sem alterar nenhum resultado**.
> Irrelevante nessa escala, mas citar como "identificada e não implementada por irrelevância na escala atual" é leitura crítica do próprio código — melhor do que a banca achar sozinha.

**Verificação:** traduza os testes de [[testes-e-validacao-tcc|✅ Testes]] em afirmações sobre o modelo — nunca cole nomes de funções no corpo do artigo.

> [!success] O teste mais forte que você tem
> O perfil misto classificando *Injeção Eletrônica* acima de *Motores a Combustão*. É o único que **não** poderia ser satisfeito por uma heurística trivial de "área com maior peso". Dê destaque em negrito no texto e cite na apresentação.

**Algoritmo:** em pseudocódigo numerado, nunca em Python (código-fonte vai em apêndice). Anote cada linha com a equação que a fundamenta — `▷ Eq. (2)`. É barato e muda a percepção de rigor do capítulo inteiro.

---

## 8️⃣ Limitações — três, e a terceira é a que impressiona

1. **Origem dos pesos** — curadoria manual, não aprendizado. Adequado à escala, mas a qualidade da recomendação depende da qualidade da curadoria, não validada estatisticamente.
2. **Granularidade** — 6 questões × 4 alternativas limita o espaço de perfis a $4^6 = 4096$ combinações.
3. **Independência entre áreas** — o modelo trata as 7 áreas como ortogonais, mas *eletromecânica* é, por definição, interseção de duas outras. A soma vetorial e o produto interno assumem uma independência que o domínio não tem, superestimando similaridade de perfis correlacionados. **Distância de Mahalanobis** corrigiria, ao custo de dados para estimar a matriz de covariância — que não existem.

> [!important] Regra geral
> Limitação que **você** levanta é maturidade; limitação que a **banca** levanta é lacuna. O conteúdo é o mesmo — muda quem fala primeiro. A terceira é a que uma banca de exatas provavelmente levantaria: nomeie a solução e explique por que não a adotou.

---

## 📚 Referências da seção

- GOODFELLOW, I.; BENGIO, Y.; COURVILLE, A. **Deep learning**. Cambridge, MA: MIT Press, 2016. — identidade do produto interno (cap. 2), *representation learning*.
- MANNING, C. D.; RAGHAVAN, P.; SCHÜTZE, H. **Introduction to information retrieval**. Cambridge University Press, 2008. — normalização do cosseno, euclidiana como tentativa ingênua.
- SALTON, G.; WONG, A.; YANG, C. S. A vector space model for automatic indexing. **Communications of the ACM**, v. 18, n. 11, p. 613–620, 1975. — o VSM.
- SARWAR, B. et al. Item-based collaborative filtering recommendation algorithms. In: **WWW '01**. ACM, 2001. p. 285–295. — comparação empírica entre medidas.
- ADOMAVICIUS; TUZHILIN (2005) · LOPS; DE GEMMIS; SEMERARO (2011) · PAZZANI; BILLSUS (2007) — sustentam a fundamentação (*cold start* e recomendação por conteúdo).

---

## 🎤 Para a defesa

- **Três números de cor:** `0,9877` (escore do exemplo), `212` (produto interno do curso que o cosseno rebaixou), `4096` (tamanho do espaço de perfis).
- **Slide-chave:** o ranking completo com a linha do *Manutenção Eletromecânica* destacada, ao lado da fórmula (2) com o denominador circulado. É o instante em que a plateia entende o que a normalização faz.
- **"Por que não usou IA?"** → não há dados históricos para treinar, o catálogo é pequeno, e explicabilidade já vem de graça da linearidade. Um modelo aprendido resolveria um problema que você não tem. Ver [[decisao-camada-ia|🤖 Camada de IA]].
- **"Validou com usuários reais?"** → seja direto: o trabalho valida a *implementação* contra o *modelo*, não a *adequação pedagógica*, que exigiria estudo com candidatos e acompanhamento de matrícula. Trabalho futuro — dizer isso é mais forte que improvisar.

## Veja também

- [[TCC|🎓 TCC]]
- [[fundamentacao-teorica-recomendacao|📖 Fundamentação teórica]] — o *por quê*, que antecede esta seção
- [[engine-matching-cosseno|🧮 Engine de matching (cosseno)]] — o código que esta nota transforma em texto
- [[defesa-monografia-tcc|🎤 Defesa e monografia]]
- [[testes-e-validacao-tcc|✅ Testes e validação]]
- [[catalogo-areas-e-cursos|📚 Catálogo de áreas, cursos e perguntas]]
- [[cs-linear-algebra|Álgebra linear]]
