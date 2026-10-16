# Roteiro do vídeo — T1 Interpretabilidade (≈ 12–13 min)

**Formato sugerido:** slides com as figuras de `figures/` + trechos do notebook rolando na tela.
**Divisão para 3 integrantes:** A = blocos 1–3 · B = blocos 4–6 · C = blocos 7–9.
Os tempos são indicativos. Fale o texto com as suas palavras; não leia.

---

## 1. Abertura — (0:00–0:45) · A
**Tela:** título + nomes + pergunta em destaque: *“Por que o modelo chegou a essa conclusão?”*

> “Imagine que você pediu um empréstimo e o banco recusou. Você tem direito de saber o porquê, e a LGPD
> garante isso. Neste trabalho treinamos três classificadores — KNN, Naïve Bayes e Árvore de Decisão —
> para prever risco de crédito. Depois investigamos se dá para responder *por que* cada um decidiu o que decidiu.”

## 2. Dataset e problema — (0:45–2:00) · A
**Tela:** tabela de requisitos → `eda_categoricas.png`

- German Credit (UCI): 1000 clientes, 20 atributos (7 numéricos e 13 categóricos), 30% de maus pagadores.
- Por que este dataset: é uma decisão regulada e tem atributos sensíveis (idade, sexo/estado civil, estrangeiro).
- Traduzimos os códigos (`A11` → “saldo < 0 DM”) para que as explicações fossem legíveis.
- **Mostre no gráfico:** o saldo em conta é o atributo mais forte (49% de inadimplência com saldo negativo
  contra 12% sem conta).
- **Plante a dúvida:** “histórico *crítico* tem a *menor* inadimplência. Guardem isso, a gente volta nesse ponto.”

## 3. Metodologia de treino — (2:00–3:30) · A
**Tela:** tabela de pré-processamento por modelo + célula do GridSearch

- Divisão 80/20 estratificada. O teste não é usado em nenhuma decisão.
- Um pré-processamento para cada modelo, com o porquê:
  - KNN: padronização + one-hot, porque usa distância;
  - NB: tudo discretizado (`CategoricalNB`), para obter tabelas de probabilidade legíveis;
  - Árvore: one-hot sem escala, profundidade limitada para continuar legível.
- GridSearch com CV de 5 folds, otimizando o F1 da classe “mau”.
- **Decisão que vale contar:** “No começo otimizamos AUC. O KNN escolheu k = 93 e, no limiar 0,5, só
  detectava 3% dos maus pagadores. Por isso trocamos para F1.”

## 4. Avaliação — (3:30–4:45) · B
**Tela:** tabela de resultados + `avaliacao.png`

- Linha de base “sempre bom” = 70% de acurácia. Por isso a acurácia engana.
- NB: melhor AUC (0,78) e precisão. Árvore: maior recall (0,85) e menor custo, mas recusa muitos bons
  clientes. KNN: pior F1.
- Custo 5:1. Com o limiar de 1/6, os três ficam com custo parecido. “Parte da diferença entre modelos é
  escolha de limiar.”
- Transição: “Os números estão bons. Mas *por que* eles decidem assim?”

## 5. Árvore de Decisão — (4:45–6:00) · B
**Tela:** `arvore.png` → caminho do cliente-exemplo (célula com “nó 0… FOLHA”)

- Apresente o cliente-exemplo: um mau pagador real, com saldo negativo, 45 meses de prazo e 11.816 DM.
- Leia a árvore em voz alta: “Tem conta corrente? Prazo > 12 meses? Poupança < 100 DM? → risco 72%.”
- O modelo inteiro cabe em 8 regras. Usa 7 de 20 features, e nenhuma é sensível.
- Limitação: as regras são grosseiras (todo mundo com conta e prazo > 12 meses é “mau”), a árvore tem o
  menor AUC e é instável.

## 6. Naïve Bayes — (6:00–7:30) · B
**Tela:** tabela de `checking_status` → `nb_log_razao.png` → `nb_local.png`

- Fórmula na tela: o log-odds é a priori + uma soma de evidências. Cada feature contribui com um número exato.
- “Saldo negativo multiplica a chance de ‘mau’ por 2,1.”
- Decomposição do cliente: −0,85 + 2,78 = 1,93 → 87%, o mesmo valor do modelo. A explicação é exata.
- **Volte à dúvida:** o NB aprendeu que “todos os créditos pagos” *aumenta* o risco e “conta crítica” o
  *diminui*. O modelo é fiel aos dados, mas os dados provavelmente têm viés de seleção ou erro de
  codificação (Grömping, 2019).
- Limitação: a independência é falsa, então evidências correlacionadas são contadas em dobro.

## 7. KNN + SHAP + LIME — (7:30–9:30) · C
**Tela:** tabela de vizinhos → `knn_shap_global.png` → `knn_shap_local.png` → `knn_lime_local.png`

- Por que é difícil: não há parâmetros. “Seus 5 vizinhos mais parecidos: 3 maus, 2 bons.” Isso diz *quem*,
  não *o quê*.
- SHAP em uma frase: a teoria dos jogos divide a previsão entre as features.
- **Achado principal:** sexo/estado civil é a 3ª feature mais importante do KNN. A causa é o one-hot +
  distância, que dá peso igual a todas as categorias. “Nenhuma métrica mostraria isso.”
- LIME: concorda com o SHAP no principal (saldo, prazo, poupança) e diverge nos detalhes. O R² local é de
  0,19, então a explicação é pouco fiel.
- Ressalva: SHAP e LIME explicam o comportamento do modelo, não a causalidade.

## 8. Comparação — (9:30–11:15) · C
**Tela:** `agnosticas_3modelos.png` → `concordancia_rankings.png` → `shap_local_3modelos.png`

- Concordam no topo: o saldo em conta é o nº 1 em todas as 8 medidas.
- Fora do topo, a concordância é moderada (Spearman de 0,5 a 0,6): `purpose` e `credit_history` importam
  para o NB e o KNN, mas não para a árvore.
- **Mesmo cliente, três justificativas:** o NB cita o histórico, a árvore cita saldo + poupança e o KNN
  cita o prazo. “Se o banco tivesse que explicar a recusa, a resposta dependeria do modelo.”
  (efeito Rashomon)
- Limitações em uma linha cada:
  - Árvore: só é legível se for rasa;
  - NB: independência falsa;
  - KNN: depende de explicações post-hoc e aproximadas.

## 9. Importância da interpretabilidade e conclusão — (11:15–12:45) · C (ou todos)
**Tela:** quatro bullets do desafio final

> “Avaliar um modelo é responder duas perguntas: *ele acerta?* e *por que ele acerta?*”

1. Confirmou que os modelos usam variáveis financeiras legítimas, o que gera confiança.
2. Revelou uma regra espúria nos dados (histórico de crédito).
3. Detectou o uso de um atributo sensível pelo KNN.
4. Mostrou que a explicação dada ao cliente depende do modelo.

- Recomendação:
  - Naïve Bayes, pelo equilíbrio entre desempenho e explicação exata;
  - a árvore, para comunicar regras a gestores;
  - o KNN não deve ser usado como está.
- Fechamento: “A interpretabilidade transforma a previsão em apoio a uma decisão humana informada, e não em
  uma resposta opaca.” + link do repositório.

---

### Checklist de gravação
- [ ] Preencher os nomes no notebook, no README e no slide de abertura
- [ ] Gravar em 1080p, com o zoom do notebook ≥ 125% para ficar legível
- [ ] Cronometrar: alvo de 12–13 min (limite entre 10 e 15)
- [ ] Subir no YouTube como **Não listado** e colar o link em `link_video.txt`
- [ ] Conferir se os pontos da Etapa 5 foram cobertos: dataset ✔ metodologia ✔ avaliação ✔ interpretabilidade ✔ comparação ✔ importância ✔ conclusões ✔
