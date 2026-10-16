# Roteiro do vídeo — T1 Interpretabilidade (leitura direta, ≈ 12 min)

**Como usar:**
- Leia só o texto normal. É a fala completa, pronta para ler.
- As linhas `🎬 SLIDE N` dizem para qual slide avançar. Avance *antes* de começar o parágrafo.
- `(continua no SLIDE N)` significa que você não troca de slide.
- `⏸` é uma pausa curta para respirar.
- Lido em ritmo calmo, o texto dá cerca de 12 minutos.

**Slides:** https://claude.ai/artifact/U7o4oetRurYmrzzjRib7q7 — 22 slides, na ordem deste roteiro. Abra no modo Apresentar e grave a tela. A fala de cada slide também está nas notas do apresentador. Se preferir, baixe em PPTX ou PDF.


---

## 1. Abertura (≈ 45 s)

🎬 ➡️ SLIDE 1 — Capa

Olá, meu nome é Leonardo Gemin Pereira, e este é o meu trabalho sobre interpretabilidade de modelos de
aprendizado de máquina.

Imagine que você pede um empréstimo e o banco recusa. A primeira pergunta que você faz é: por quê?
E essa pergunta não é só curiosidade. A Lei Geral de Proteção de Dados garante o direito de revisão
de decisões automatizadas. ⏸

Neste trabalho eu treinei três classificadores — KNN, Naïve Bayes e Árvore de Decisão — para prever o
risco de crédito de clientes de um banco. E depois fui além das métricas. Investiguei se é possível
responder exatamente essa pergunta: por que o modelo chegou a essa conclusão?

---

## 2. Dataset e problema (≈ 1 min 15 s)

🎬 ➡️ SLIDE 2 — Dataset (German Credit)

O conjunto de dados é o German Credit, do repositório UCI. São mil clientes que receberam crédito em um
banco alemão. Cada um está classificado como bom pagador ou mau pagador: setecentos bons e trezentos maus.

São vinte atributos — sete numéricos, como prazo, valor do crédito e idade, e treze categóricos, como
saldo em conta, histórico de crédito e finalidade do empréstimo. Então ele atende a todos os requisitos:
é um problema de classificação, tem variáveis numéricas e categóricas e tem bem mais de cem instâncias. ⏸

Escolhi esse dataset por dois motivos. Primeiro, crédito é uma decisão regulada, onde explicar é
obrigação. Segundo, ele tem atributos sensíveis — idade, sexo e estado civil, e se a pessoa é
trabalhadora estrangeira. Isso me permite verificar se os modelos estão se apoiando neles.

Um detalhe: os valores originais vêm codificados, como “A11” e “A34”. Eu traduzi todos para rótulos
legíveis, como “saldo negativo” ou “sem conta corrente”. Uma explicação só é útil se a pessoa entende.

🎬 ➡️ SLIDE 3 — Taxa de maus pagadores por categoria

Na análise exploratória, o atributo que mais chama atenção é o saldo em conta corrente. Quem tem saldo
negativo tem quarenta e nove por cento de inadimplência. Quem não tem conta corrente, só doze por cento. ⏸

E aqui tem algo estranho. Olhe o histórico de crédito. Quem tem histórico “crítico” tem a *menor* taxa de
inadimplência, dezessete por cento. E quem pagou todos os créditos tem a maior. Isso contraria o bom
senso. Guarde essa informação, porque eu vou voltar nela mais adiante.

---

## 3. Metodologia de treino (≈ 1 min 30 s)

🎬 ➡️ SLIDE 4 — Um pré-processamento para cada modelo

Para o treinamento, eu dividi os dados em oitenta por cento para treino e vinte por cento para teste, de
forma estratificada. Assim, os dois conjuntos mantêm a proporção de trinta por cento de maus pagadores. O
conjunto de teste não foi usado em nenhuma decisão. Ele serve só para a avaliação final.

Não havia valores ausentes nem linhas duplicadas, então não foi necessário imputar dados. ⏸

Cada algoritmo tem premissas diferentes, então eu fiz um pré-processamento específico para cada um:

- **KNN:** ele calcula distâncias. Por isso padronizei as variáveis numéricas — senão o valor do crédito,
  que chega a dezoito mil, dominaria todo o resto. As categóricas foram transformadas com one-hot.
- **Naïve Bayes:** usei a versão categórica e transformei as variáveis numéricas em faixas. Assim o modelo
  aprende tabelas de probabilidade que dá para ler diretamente, o que vai ser essencial na parte de
  interpretabilidade.
- **Árvore de Decisão:** não precisa de escala. Só apliquei one-hot nas categóricas, para que cada nó faça
  uma pergunta simples, do tipo “o cliente tem tal característica?”.

Tudo isso foi feito dentro de pipelines, para evitar vazamento de dados.

🎬 (continua no SLIDE 4 — caixa “F1, não AUC”)

Os hiperparâmetros foram escolhidos com busca em grade e validação cruzada de cinco partes, otimizando o
F1 da classe “mau pagador”. ⏸

E aqui vale contar uma decisão. No começo eu otimizei pela AUC. O KNN escolheu noventa e três vizinhos.
Ele ordenava bem os clientes, mas na prática só detectava três por cento dos maus pagadores. Um modelo
inútil. Por isso troquei para o F1, que equilibra precisão e recall da classe que interessa. Também
limitei a árvore a no máximo cinco níveis, porque uma árvore enorme só é interpretável na teoria.

---

## 4. Avaliação (≈ 1 min 15 s)

🎬 ➡️ SLIDE 5 — Tabela de resultados

Agora os resultados no conjunto de teste. Primeiro, uma referência: um modelo que diz que todo mundo é bom
pagador já acerta setenta por cento. Por isso a acurácia sozinha engana, e eu usei também precisão,
recall, F1, acurácia balanceada e AUC. ⏸

- **Naïve Bayes:** teve a melhor AUC, zero vírgula setenta e oito, e a melhor precisão.
- **Árvore de Decisão:** teve o maior recall, oitenta e cinco por cento. Ela pega quase todos os maus
  pagadores, mas, em troca, recusa muitos bons clientes.
- **KNN:** ficou com o pior F1. Como setenta por cento da base é de bons pagadores, os vizinhos tendem a
  ser bons, e ele raramente prevê “mau”.

🎬 ➡️ SLIDE 6 — Matrizes de confusão, ROC e custo

O próprio dataset define um custo: aprovar um mau pagador custa cinco vezes mais que recusar um bom. Se eu
ajusto o limiar de decisão para esse custo, os três modelos ficam com custo total parecido. Ou seja, parte
da diferença entre eles é só escolha de limiar. ⏸

Os três modelos são claramente melhores que a referência. Então o desempenho é suficiente para a pergunta
principal: por que eles decidem assim?

---

## 5. Árvore de Decisão (≈ 1 min 15 s)

🎬 ➡️ SLIDE 7 — O cliente-exemplo

Para tornar a análise concreta, escolhi um cliente-exemplo do conjunto de teste. É um mau pagador real:
saldo negativo em conta, crédito de quarenta e cinco meses, quase doze mil marcos e poupança abaixo de
cem. Os três modelos o classificam como alto risco. Vou explicar esse cliente com cada técnica.

🎬 ➡️ SLIDE 8 — Árvore: 8 regras

Começando pela árvore, que é o modelo mais transparente: a própria estrutura é a explicação. O modelo
inteiro cabe em oito regras. ⏸

A primeira pergunta é: o cliente tem conta corrente? Se tem, e o prazo é maior que doze meses, ele já é
classificado como mau pagador. Se não tem conta corrente, ele só é considerado mau se já tem outro
parcelamento e pede mais de três mil e quatrocentos marcos.

🎬 ➡️ SLIDE 9 — O caminho do cliente-exemplo

Para o cliente-exemplo, o caminho é: tem conta, prazo de quarenta e cinco meses, poupança abaixo de cem.
Risco de setenta e dois por cento. É uma explicação completa e fiel, que qualquer analista consegue
conferir. ⏸

Outro ponto importante: a árvore usa só sete das vinte variáveis, e nenhuma delas é sensível. A limitação
é que essas regras são grosseiras. Todo mundo que cai na mesma folha recebe a mesma resposta. Por isso
ela tem a menor AUC dos três.

---

## 6. Naïve Bayes (≈ 1 min 30 s)

🎬 ➡️ SLIDE 10 — Naïve Bayes: soma de evidências

O Naïve Bayes usa o teorema de Bayes, assumindo que as variáveis são independentes. A vantagem é que, em
escala logarítmica, a decisão vira uma soma. Cada variável contribui com um número exato a favor ou contra
o cliente. ⏸

Nesta tabela estão as probabilidades condicionais do saldo em conta. Saldo negativo é muito mais comum
entre maus pagadores. Na prática, ele multiplica a chance de ser mau pagador por cerca de duas vezes. Já
não ter conta corrente divide essa chance por três.

🎬 ➡️ SLIDE 11 — Naïve Bayes: explicação do cliente

Para o cliente-exemplo, eu somo essas contribuições. O ponto de partida, a probabilidade a priori, vale
menos zero vírgula oitenta e cinco. As evidências somam mais dois vírgula setenta e oito. O total dá
oitenta e sete por cento de risco — exatamente o que o modelo responde. A explicação não é uma
aproximação, é a conta do próprio modelo. ⏸

🎬 ➡️ SLIDE 12 — Naïve Bayes: a regra estranha

E agora volto àquele ponto estranho. O Naïve Bayes aprendeu que ter histórico “crítico” *reduz* o risco, e
que ter pagado todos os créditos *aumenta* o risco. O modelo está sendo fiel aos dados. O problema está
nos dados. Isso provavelmente é viés de seleção: o banco só aprovou os clientes críticos que eram mais
seguros. Também existe um erro de codificação documentado nesse dataset, em um artigo de Grömping, de
2019. ⏸

Nenhuma métrica mostraria isso. Só a interpretabilidade. A limitação do Naïve Bayes é a premissa de
independência, que é falsa: variáveis correlacionadas, como prazo e valor, acabam sendo contadas em dobro.

---

## 7. KNN, SHAP e LIME (≈ 2 min)

🎬 ➡️ SLIDE 13 — KNN: os vizinhos

O KNN é o mais difícil de interpretar. Ele não aprende regras nem probabilidades. Ele só procura os
clientes mais parecidos e faz uma votação. ⏸

Aqui estão os cinco vizinhos do cliente-exemplo: três maus pagadores e dois bons. Isso me diz *quem* são
os clientes parecidos, mas não diz *quais características* pesaram. Para isso eu precisei de ferramentas
externas.

🎬 ➡️ SLIDE 14 — KNN: SHAP global (sexo/estado civil)

A primeira é o SHAP. Ele usa uma ideia da teoria dos jogos para dividir a previsão entre as variáveis, de
forma justa. Apliquei o SHAP sobre as vinte variáveis originais, para que a explicação fale a língua do
negócio. ⏸

E aqui está o achado mais importante do trabalho. Para o KNN, sexo e estado civil é a terceira variável
mais importante. Na árvore, ela nem é usada. No Naïve Bayes, fica em décimo lugar. ⏸

A causa é técnica. Com one-hot e distância, todas as variáveis categóricas pesam igual na hora de definir
quem é vizinho de quem. O KNN não sabe que essa variável é pouco informativa. Ou seja: um atributo
sensível está influenciando decisões de crédito por um efeito colateral do pré-processamento. E, de novo,
nenhuma métrica de desempenho revelaria isso.

🎬 ➡️ SLIDE 15 — KNN: SHAP do cliente

Para o cliente-exemplo, o SHAP mostra que o saldo negativo e o prazo de quarenta e cinco meses foram os
principais motivos do risco.

🎬 ➡️ SLIDE 16 — KNN: LIME

A segunda ferramenta é o LIME. Ele cria pequenas variações do cliente e ajusta um modelo linear simples ao
redor dele. O LIME concorda com o SHAP no principal: saldo negativo, prazo longo e pouca poupança. Mas
discorda nos fatores secundários. E o modelo linear local explicou só dezenove por cento do comportamento
do KNN nessa região, ou seja, é uma explicação pouco fiel. ⏸

Uma ressalva importante: SHAP e LIME explicam o comportamento do modelo, não a realidade. São aproximações
feitas depois do treino.

---

## 8. Comparação entre os modelos (≈ 1 min 45 s)

🎬 ➡️ SLIDE 17 — Todos concordam no topo

Para comparar os três modelos lado a lado, apliquei duas ferramentas iguais para todos: a importância por
permutação e o SHAP. ⏸

Eles concordam no topo. O saldo em conta corrente é a variável número um em todas as oito medidas que
calculei, nos três modelos. Prazo e poupança também aparecem entre as mais importantes na maioria.

🎬 ➡️ SLIDE 18 — Concordância entre rankings

Fora do topo, a concordância é só moderada. A correlação entre os rankings fica entre zero vírgula cinco e
zero vírgula seis. A finalidade do crédito e o histórico importam para o Naïve Bayes e para o KNN, mas a
árvore nem usa. ⏸

🎬 ➡️ SLIDE 19 — Mesmo cliente, três justificativas

E este gráfico resume tudo. É o mesmo cliente, explicado pelos três modelos. Todos concordam que ele é de
risco. Mas o motivo principal muda:

- para o **Naïve Bayes**, é o histórico de crédito — justamente a regra estranha;
- para a **árvore**, é o saldo negativo junto com a poupança baixa;
- para o **KNN**, o prazo pesa quase tanto quanto o saldo.

Se o banco tivesse que explicar a recusa para esse cliente, a resposta dependeria do modelo escolhido.
Isso tem nome: efeito Rashomon. Vários modelos igualmente bons, com raciocínios diferentes. ⏸

🎬 ➡️ SLIDE 20 — Cada modelo se explica de um jeito

Resumindo as limitações:

- a **árvore** só é legível se for rasa, e isso custa desempenho;
- o **Naïve Bayes** assume uma independência que não existe;
- o **KNN** depende de explicações aproximadas e da forma como a distância é calculada.

---

## 9. Importância da interpretabilidade e conclusão (≈ 1 min 30 s)

🎬 ➡️ SLIDE 21 — O que nenhuma métrica teria mostrado

Para fechar, a pergunta do desafio: como a interpretabilidade ajuda a usar previsões de forma mais
consciente? ⏸

Neste trabalho, ela fez quatro coisas que nenhuma métrica faria:

1. **Confiança:** confirmou que os três modelos se apoiam em variáveis financeiras legítimas — saldo,
   prazo e poupança.
2. **Problema nos dados:** revelou uma regra provavelmente errada, a do histórico de crédito.
3. **Viés:** detectou que o KNN usa sexo e estado civil como uma das variáveis mais importantes.
4. **Explicação ao cliente:** mostrou que a justificativa dada ao cliente depende do modelo.

🎬 ➡️ SLIDE 22 — Conclusão

Com base nisso, minha recomendação é a seguinte:

- usar o **Naïve Bayes**, que teve a melhor AUC e tem uma explicação exata — depois de revisar a variável
  de histórico de crédito;
- usar a **árvore** para comunicar regras aos gestores;
- **não usar o KNN** como está.

🎬 (continua no SLIDE 22 — “Ele acerta? / Por que ele acerta?”)

Avaliar um modelo de aprendizado de máquina é responder duas perguntas: ele acerta? E por que ele acerta?
A segunda pergunta é o que transforma uma previsão em apoio a uma decisão humana informada, e não em uma
resposta opaca. ⏸

O código e toda a documentação estão no repositório do GitHub. Obrigado!

---

### Checklist rápido
- [ ] Slides abertos no modo Apresentar, em tela cheia
- [ ] Gravar em 1080p; ler com calma (alvo de 11–13 min, limite de 10–15)
- [ ] Publicar no YouTube como **Não listado** e colar o link em `link_video.txt`
