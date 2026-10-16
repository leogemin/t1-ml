# T1 — Interpretabilidade de Modelos de Aprendizado de Máquina

KNN × Naïve Bayes × Árvore de Decisão no problema de **risco de crédito** (German Credit, UCI).

**Disciplina:** Aprendizado de Máquina 
**Aluno:** Leonardo Gemin Pereira 
**Vídeo:** ver [`link_video.txt`](link_video.txt)

## Estrutura

| Caminho | Conteúdo |
|---|---|
| [`notebooks/t1_interpretabilidade.ipynb`](notebooks/t1_interpretabilidade.ipynb) | **Artefato principal**, já executado. Código + justificativas + discussão de todas as etapas. |
| `data/german.data` | Dataset original (UCI Statlog German Credit, 1000 × 20 + alvo). |
| `figures/` | Figuras geradas pelo notebook (usadas no vídeo). |
| `link_video.txt` | Link do vídeo no YouTube (não listado). |
| `requirements.txt` | Dependências com versões. |

## Como executar

```bash
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
cd notebooks
jupyter nbconvert --to notebook --execute --inplace t1_interpretabilidade.ipynb   # ~3 min
```

Tudo usa `random_state=42`; os resultados são reprodutíveis.

---

## Etapa 1 — Dataset

**Statlog (German Credit Data)** — Hofmann (1994), UCI ML Repository.
Um banco decide se concede crédito; cada cliente é rotulado **bom pagador (700)** ou **mau pagador (300)**.

| Requisito | Atendido |
|---|---|
| ≥ 5 features | 20 |
| Alvo categórico | bom / mau pagador |
| Numéricas e categóricas | 7 numéricas + 13 categóricas |
| ≥ 100 instâncias | 1000 |
| Não usado em aula | sim |

Por que este dataset: crédito é uma decisão **regulada** (LGPD art. 20 garante revisão de decisões
automatizadas). Além disso, o dataset tem **atributos sensíveis** (idade, sexo/estado civil, trabalhador
estrangeiro). Os códigos `A11`, `A34`… foram traduzidos para rótulos legíveis, para que as explicações
façam sentido para um analista de crédito.

## Etapa 2 — Pré-processamento e treino

- Sem valores ausentes nem duplicatas. Outliers mantidos (valores altos de crédito são legítimos).
- **Divisão 80/20 estratificada.** Os 200 clientes de teste não são usados em nenhuma decisão de modelagem.
- Pré-processamento **por modelo**, sempre dentro de `Pipeline` (sem vazamento de dados):

| Modelo | Numéricas | Categóricas | Por quê |
|---|---|---|---|
| KNN | StandardScaler | One-Hot | distância exige escala comum; one-hot não cria ordem falsa |
| Naïve Bayes (`CategoricalNB`) | KBinsDiscretizer (quantis) | OrdinalEncoder | gera tabelas P(valor\|classe) legíveis; as numéricas são assimétricas (não-gaussianas) |
| Árvore | — | One-Hot | invariante à escala; o one-hot gera regras do tipo “tem X?” |

- `GridSearchCV` com CV estratificada de 5 folds, otimizando o **F1 da classe “mau”**. Otimizar AUC levava
  o KNN a k = 93, com recall de 3% no limiar 0,5. A árvore foi limitada a profundidade ≤ 5 para continuar legível.
- Hiperparâmetros escolhidos: KNN k=5, distância Manhattan, pesos por distância · NB α=1, 3 faixas ·
  Árvore profundidade 3, `min_samples_leaf=20`, entropia, `class_weight="balanced"`.

### Resultados (teste, limiar 0,5, positivo = mau pagador)

| Modelo | Acurácia | Precisão | Recall | F1 | Acur. bal. | ROC-AUC | Custo* |
|---|---|---|---|---|---|---|---|
| Baseline (sempre bom) | 0,700 | 0,000 | 0,000 | 0,000 | 0,500 | 0,500 | 300 |
| KNN | 0,730 | 0,575 | 0,383 | 0,460 | 0,631 | 0,749 | 202 |
| Naïve Bayes | **0,755** | **0,604** | 0,533 | 0,566 | **0,692** | **0,780** | 161 |
| Árvore de Decisão | 0,625 | 0,436 | **0,850** | **0,576** | 0,689 | 0,704 | **111** |

\*Matriz de custo do dataset: aprovar um mau pagador custa 5; recusar um bom custa 1. Com o limiar ótimo
de custo (P(mau) > 1/6), os três modelos convergem para um custo de ≈ 110–120.

## Etapa 3 — Interpretabilidade

| | Árvore | Naïve Bayes | KNN |
|---|---|---|---|
| Intrínseca global | estrutura/regras, importância por impureza | tabelas P(valor\|classe), log-razão | — |
| Intrínseca local | caminho de decisão | decomposição aditiva **exata** do log-odds | vizinhos mais próximos |
| Agnóstica | permutação, SHAP | permutação, SHAP | permutação, SHAP, **LIME** |

Principais achados:
- **Árvore:** 8 regras; a raiz é `checking_status = sem conta corrente`. Usa só 7 das 20 features, e
  nenhuma delas é sensível.
- **Naïve Bayes:** saldo < 0 DM multiplica a chance de “mau” por ≈ 2,1. **Achado contraintuitivo:**
  `credit_history = conta crítica` *reduz* o risco e “todos pagos” o *aumenta*. A causa provável é viés de
  seleção ou erro de codificação do dataset (Grömping, 2019).
- **KNN:** não tem explicação nativa. SHAP e permutação mostram que **sexo/estado civil é a 3ª feature mais
  influente** (2ª na permutação). É um artefato do one-hot + distância. SHAP e LIME concordam nos fatores
  principais, mas divergem nos secundários; o LIME teve baixa fidelidade local (R² ≈ 0,19).

## Etapa 4 — Comparação

- `checking_status` é a feature nº 1 em **todas** as 8 medidas de importância × modelo.
- Fora do topo, a concordância é moderada (Spearman entre rankings SHAP: 0,51–0,61).
  `purpose` e `credit_history` importam para o NB e o KNN, mas não para a árvore.
- O mesmo cliente recebe **três justificativas diferentes**: o NB cita o histórico de crédito, a árvore cita
  saldo + poupança e o KNN cita o prazo. Isso é o efeito Rashomon.

A discussão completa (sentido de negócio, limitações de cada modelo, ferramentas) está nas §4–5 do notebook.

## Desafio final — resumo

Métricas dizem *se* o modelo acerta; a interpretabilidade diz *por quê*. Neste trabalho, ela:
1. confirmou que os modelos usam variáveis financeiras legítimas;
2. revelou uma regra provavelmente espúria aprendida dos dados (`credit_history`);
3. detectou o uso de atributo sensível pelo KNN, que nenhuma métrica mostraria;
4. mostrou que a explicação dada ao cliente depende do modelo escolhido.

Recomendação: **Naïve Bayes** (melhor AUC e explicação exata, após revisar `credit_history`). A
**árvore** é a melhor opção para comunicar regras a gestores. O **KNN** não deve ser usado como está.

## Referências
- Hofmann, H. (1994). *Statlog (German Credit Data)*. UCI ML Repository. https://doi.org/10.24432/C5NC77
- Grömping, U. (2019). *South German Credit Data: Correcting a Widely Used Data Set*. Beuth Hochschule, Report 4/2019.
- Lundberg & Lee (2017). *A Unified Approach to Interpreting Model Predictions*. NeurIPS.
- Ribeiro, Singh & Guestrin (2016). *“Why Should I Trust You?”*. KDD.
- Molnar, C. (2022). *Interpretable Machine Learning*.
