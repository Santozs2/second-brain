---
title: "Fundamentação teórica — recomendação por conteúdo e escolha do cosseno"
aliases: ["Fundamentação do TCC", "Cold start e VSM", "Por que cosseno"]
tags: [tcc, monografia, artigo, fundamentacao, cosseno, cold-start, vsm, recomendacao]
status: em-andamento
projeto: TCC
criado: 2026-08-20
---

> [!info] Projeto: [[TCC|🎓 TCC]] · Metodologia: [[artigo-secao-calculo-cosseno|✍️ Seção do cálculo]] · Código: [[engine-matching-cosseno|🧮 Engine de matching]]

# 📖 Fundamentação teórica — por que recomendação por conteúdo e por que cosseno

> [!abstract] O que é esta nota
> Rascunho da **fundamentação teórica** da monografia. Responde *por que* essa família de técnica e *por que* essa medida. A nota irmã [[artigo-secao-calculo-cosseno|✍️ Seção do cálculo]] responde *como* o cálculo é formalizado e verificado.
> Adapte a numeração (2.X) e a costura com as seções vizinhas ao chegar no documento final.

> [!important] Duas decisões distintas — não misture
> O ***cold start*** justifica a **família de técnica** (recomendação por conteúdo). O **cosseno** justifica a **medida dentro da família**. São argumentos separados, e a ordem importa: cold start vem **antes**.

---

## 2.X Recomendação baseada em conteúdo e o problema da partida a frio

> [!quote] Texto para o artigo
> O sistema proposto enquadra-se na categoria dos sistemas de recomendação baseados em conteúdo, nos quais os itens são sugeridos a partir da correspondência entre seus atributos e um perfil de preferências do usuário (PAZZANI; BILLSUS, 2007; LOPS; DE GEMMIS; SEMERARO, 2011). A escolha dessa família de técnicas decorre de uma restrição estrutural do domínio: o público-alvo — candidatos que ainda não ingressaram na instituição — não possui histórico de interações com os cursos do catálogo, o que inviabiliza a filtragem colaborativa, dependente de dados de comportamento pregresso de usuários semelhantes. Esse cenário corresponde ao problema clássico de partida a frio (*cold start*), para o qual a literatura aponta as abordagens baseadas em conteúdo como alternativa adequada (ADOMAVICIUS; TUZHILIN, 2005; GOODFELLOW; BENGIO; COURVILLE, 2016).

## 2.X.1 Representação vetorial de perfis e cursos

> [!quote] Texto para o artigo
> A modelagem fundamenta-se no Modelo de Espaço Vetorial (*Vector Space Model* — VSM), proposto por Salton, Wong e Yang (1975) no contexto da recuperação de informação. Embora formulado originalmente para a comparação entre documentos e consultas, os próprios autores observam que o modelo se aplica a qualquer conjunto de entidades descritas por vetores de pesos, o que legitima sua instanciação no presente domínio.
>
> Aqui, o espaço vetorial é definido pelo conjunto de sete áreas profissionais: elétrica, costura, inteligência artificial, tecnologia da informação, mecânica automotiva, mecânica e eletromecânica. Cada curso *c* é representado por um vetor cujas componentes são os pesos que expressam a intensidade da relação entre o curso e cada área, definidos por curadoria pedagógica. O perfil do candidato *p* é construído no mesmo espaço: cada alternativa do questionário carrega pesos por área, e o vetor de perfil resulta da soma dos pesos das alternativas selecionadas. Perfil e curso tornam-se, portanto, objetos matematicamente comparáveis.

## 2.X.2 Definição da medida

> [!quote] Texto para o artigo
> A afinidade entre um perfil *p* e um curso *c* é calculada pela similaridade por cosseno:
>
> $$\operatorname{sim}(p, c) = \cos \theta = \frac{p \cdot c}{\|p\| \cdot \|c\|}$$
>
> em que $p \cdot c$ é o produto interno dos vetores e $\|\cdot\|$ denota a norma euclidiana. A expressão decorre diretamente da identidade do produto interno, $x^\top y = \|x\|\,\|y\|\cos\theta$, em que $\theta$ é o ângulo entre os vetores (GOODFELLOW; BENGIO; COURVILLE, 2016, cap. 2). A medida quantifica, assim, o quanto os dois vetores apontam na mesma direção do espaço de áreas, independentemente de suas magnitudes.

## 2.X.3 Justificativa da escolha

Quatro propriedades tornam o cosseno adequado ao problema.

> [!quote] Invariância à magnitude
> O cosseno atua como uma normalização de comprimento na comparação entre vetores (MANNING; RAGHAVAN; SCHÜTZE, 2008). No sistema proposto, essa propriedade é decisiva: o vetor de perfil resulta da soma dos pesos de seis respostas, enquanto o vetor de curso possui escala própria, definida na curadoria — as magnitudes são, por construção, incomparáveis. A informação relevante está na *proporção* entre as áreas de interesse, exatamente o que o cosseno captura ao medir direção e descartar magnitude.

> [!quote] Contradomínio interpretável
> Como todos os pesos do sistema são não negativos, o ângulo entre os vetores não excede 90°, e a medida fica restrita ao intervalo [0, 1] (MANNING; RAGHAVAN; SCHÜTZE, 2008). O escore pode então ser lido diretamente como grau de afinidade — 0 para ausência de relação, 1 para alinhamento total — o que atende tanto à ordenação dos cursos quanto à exibição do resultado ao usuário final.

> [!quote] Adequação a vetores esparsos
> Cada perfil e cada curso ativam apenas um subconjunto das áreas. No produto interno, somente as dimensões compartilhadas contribuem para o resultado, de modo que o cálculo tem custo proporcional à interseção dos vetores — propriedade explorada na implementação do módulo `quiz/engine.py`, cuja representação por dicionário esparso é descrita na seção de desenvolvimento.

> [!quote] Interpretabilidade da recomendação
> O numerador da medida decompõe-se aditivamente em contribuições por área (peso do perfil × peso do curso em cada dimensão). Essa decomposição permite identificar quais áreas mais aproximaram o candidato de cada curso, viabilizando o requisito de justificar cada recomendação — a mesma operação que produz o escore produz a explicação, sem mecanismos adicionais.

> [!success] A quarta propriedade é o slide-chave da apresentação
> Mostre o `explain()` ao lado do numerador da fórmula. É o momento *"a matemática e o requisito de UX são a mesma coisa"*. A decomposição numérica pronta está em [[artigo-secao-calculo-cosseno|✍️ Seção do cálculo]].

## 2.X.4 Comparação com medidas alternativas

> [!quote] Texto para o artigo
> A distância euclidiana foi descartada por ser sensível à magnitude dos vetores, que neste domínio não carrega informação; a literatura de recuperação de informação a apresenta como a tentativa ingênua que a normalização do cosseno corrige (MANNING; RAGHAVAN; SCHÜTZE, 2008), e avaliações empíricas em sistemas de recomendação reportam erro médio consistentemente menor para o cosseno (SARWAR et al., 2001). O coeficiente de Jaccard opera sobre conjuntos binários e descartaria a informação dos pesos, núcleo da modelagem proposta. A correlação de Pearson — equivalente ao cosseno aplicado a vetores centrados na média — justifica-se quando há viés de escala entre avaliadores, como em matrizes de notas; os pesos deste sistema, não negativos e definidos por curadoria, não apresentam viés a remover, tornando a forma pura do cosseno a escolha apropriada (SARWAR et al., 2001).

> [!tip] Reforce com dado do próprio domínio
> Este parágrafo descarta a euclidiana **por citação**. Em [[artigo-secao-calculo-cosseno|✍️ Seção do cálculo]] há a mesma comparação **rodada no catálogo real**, com as três patologias numeradas. Citar o resultado empírico aqui responde de antemão à pergunta *"e no seu caso, você testou?"*.

## 2.X.5 Limitações e evolução

> [!quote] Texto para o artigo
> A principal limitação da modelagem está na origem dos vetores, definidos por curadoria manual — decisão adequada à escala do problema (questionário fixo, catálogo reduzido, ausência de dados históricos de treinamento), mas que não captura nuances fora do vocabulário de áreas. Como evolução, descrições de cursos e respostas abertas poderiam ser convertidas em representações vetoriais aprendidas (*representation learning*), na linha consolidada por Goodfellow, Bengio e Courville (2016), mantendo-se o cosseno como medida de comparação — configuração padrão dos sistemas modernos de similaridade semântica.

> [!note] Duas limitações a mais para considerar
> A seção de metodologia levanta ainda a **independência entre áreas** (eletromecânica é interseção de duas outras, mas o modelo as trata como ortogonais) e a **granularidade da entrada** ($4^6 = 4096$ perfis possíveis). Decida se entram aqui ou lá — mas não nos dois lugares.

---

## 📚 Referências

- ADOMAVICIUS, G.; TUZHILIN, A. Toward the next generation of recommender systems: a survey of the state-of-the-art and possible extensions. **IEEE Transactions on Knowledge and Data Engineering**, v. 17, n. 6, p. 734–749, 2005.
- GOODFELLOW, I.; BENGIO, Y.; COURVILLE, A. **Deep learning**. Cambridge, MA: MIT Press, 2016. Disponível em: https://www.deeplearningbook.org. Acesso em: [data].
- LOPS, P.; DE GEMMIS, M.; SEMERARO, G. Content-based recommender systems: state of the art and trends. In: RICCI, F. et al. (org.). **Recommender systems handbook**. New York: Springer, 2011. p. 73–105.
- MANNING, C. D.; RAGHAVAN, P.; SCHÜTZE, H. **Introduction to information retrieval**. Cambridge: Cambridge University Press, 2008. Disponível em: https://nlp.stanford.edu/IR-book. Acesso em: [data].
- PAZZANI, M. J.; BILLSUS, D. Content-based recommendation systems. In: BRUSILOVSKY, P.; KOBSA, A.; NEJDL, W. (org.). **The adaptive web**. Berlin: Springer, 2007. p. 325–341.
- SALTON, G.; WONG, A.; YANG, C. S. A vector space model for automatic indexing. **Communications of the ACM**, v. 18, n. 11, p. 613–620, 1975.
- SARWAR, B. et al. Item-based collaborative filtering recommendation algorithms. In: INTERNATIONAL CONFERENCE ON WORLD WIDE WEB, 10., 2001, Hong Kong. **Proceedings** [...]. New York: ACM, 2001. p. 285–295.

> [!todo] Pendência
> Preencher as datas de acesso das duas fontes online antes de fechar o documento.

---

## 🎤 Notas de defesa

*(não entram no texto)*

- **Cada subseção tem um papel:** enquadra o problema → formaliza a representação → define a medida → justifica → compara → assume limitações. Se a banca atacar qualquer ponto, existe uma subseção respondendo.
- **A ordem importa:** o *cold start* vem ANTES do cosseno, porque justifica a família de técnica; o cosseno justifica a medida dentro da família. São duas decisões distintas.
- **Slide-chave:** a propriedade 4 (interpretabilidade) — mostre o `explain()` ao lado do numerador da fórmula.

## Veja também

- [[TCC|🎓 TCC]]
- [[artigo-secao-calculo-cosseno|✍️ Artigo: seção do cálculo de cosseno]] — o *como*, com exemplo numérico verificável
- [[engine-matching-cosseno|🧮 Engine de matching (cosseno)]] — o código
- [[defesa-monografia-tcc|🎤 Defesa e monografia]]
- [[decisao-camada-ia|🤖 Camada de IA (em decisão)]]
- [[cs-linear-algebra|Álgebra linear]]
