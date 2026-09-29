# PLS Regression — o modelo de especialização do nosso grupo

## 1. Escopo

O modelo de especialização do nosso grupo, o Grupo 5, é o **Partial Least Squares Regression (PLS Regression)**. Nós o avaliamos nas mesmas cinco bases, origens de previsão, horizontes e conjuntos de teste que usamos para os modelos comuns. O Random Forest e o PLS recebem a mesma tabela de features; só o pré-processamento que o estimador exige é específico do PLS.

Implementamos o PLS no [`pipeline.ipynb`](pipeline.ipynb), nas seções **3.8 — PLS**, **4.5 — Importância das features** e **4.6 — PLS: busca, diagnóstico e resíduos por passo**. Um resumo desta documentação também entra na seção 11 do nosso relatório automático (`relatorio/`).

Os artefatos da execução ficam em [`resultados/`](resultados/) (seção 14).

## 2. O que é PLS Regression

PLS é um método supervisionado de regressão por variáveis latentes. Em vez de ajustar diretamente uma regressão sobre todas as colunas de `X`, ele constrói componentes que resumem os preditores e, ao mesmo tempo, são úteis para explicar `y`.

Sejam:

- `X ∈ R^(n×p)`: matriz com `n` observações e `p` features;
- `y ∈ R^n`: alvo univariado;
- `A`: número de componentes escolhido.

Em cada etapa `a`, o PLS procura um vetor de pesos `w_a` e forma o score latente:

```text
t_a = X_(a-1) w_a
```

A direção é escolhida para produzir alta covariância entre `t_a` e o alvo residual. De forma simplificada:

```text
max ||Cov(t_a, y_(a-1))||², sujeito a ||w_a|| = 1
```

Depois, a informação explicada pelo componente é retirada de `X` e de `y` por deflação. Ao final, a regressão pode ser representada por:

```text
y_estimado = intercepto + X_padronizado B
```

Como temos apenas um alvo em cada base, o nosso caso é o chamado **PLS1**.

## 3. Diferença para métodos relacionados

| Método | Como escolhe as direções | Usa `y` ao criar componentes? | Consequência |
|---|---|---:|---|
| Regressão linear | Não reduz dimensões | Sim, diretamente no ajuste | Pode apresentar coeficientes instáveis com forte multicolinearidade |
| PCA/PCR | Maximiza a variância de `X` | Não | Um componente com muita variância pode ter pouca utilidade preditiva |
| PLS | Favorece covariância entre projeções de `X` e `y` | Sim | Componentes são supervisionados e orientados à previsão |
| Random Forest | Divide recursivamente o espaço das features | Sim | Captura não linearidades, mas não produz componentes lineares latentes |

O PLS é adequado ao nosso trabalho porque os lags, as médias móveis e os desvios móveis que criamos a partir da mesma série são naturalmente correlacionados.

## 4. Hipóteses, requisitos e limitações

### 4.1 Hipóteses e requisitos práticos

- A relação preditiva é aproximadamente linear no espaço latente.
- As linhas utilizadas no ajuste precisam estar alinhadas temporalmente com o alvo.
- Features devem estar em escalas comparáveis.
- Transformações aprendidas devem ser ajustadas somente com dados disponíveis na origem.
- O número de componentes não pode exceder o número de features nem o posto permitido pelas amostras da janela.
- Ausências produzidas por lags e janelas precisam ser removidas antes do ajuste.

O PLS não exige independência entre as features e é justamente útil sob multicolinearidade. Não ignoramos a dependência temporal entre observações: nós a representamos com features defasadas e a avaliamos com validação walk-forward, sem embaralhamento aleatório.

### 4.2 Vantagens

- Trata conjuntos de preditores correlacionados.
- Realiza regressão e redução supervisionada de dimensão conjuntamente.
- Permite previsão com mais features do que seria confortável para uma regressão linear sem regularização.
- Oferece interpretação por coeficientes, pesos, componentes e VIP Scores.
- Tem custo computacional relativamente baixo em comparação com grades SARIMAX e florestas extensas.

### 4.3 Limitações

- Continua sendo linear no espaço latente.
- Pode não capturar interações, limiares, mudanças de regime ou não linearidades.
- É sensível à escala e a valores extremos.
- Muitos componentes podem reincorporar ruído e aproximar o modelo da regressão linear completa.
- Componentes além do posto de `X` são numericamente instáveis; features exatamente redundantes precisam ser removidas ou a grade limitada ao posto.
- Coeficientes e VIP indicam associação preditiva, não causalidade.
- Bom MAE não garante resíduos sem autocorrelação.

## 5. Features recebidas pelo PLS

A função `criar_features(base)` produz a mesma tabela que usamos no Random Forest. Para uma data prevista `t`, lags, janelas e externas observacionais desconhecidas usam somente informações de `t-h` ou anteriores. Calendário e externas comprovadamente disponíveis na origem podem representar a própria data `t` sem constituir vazamento.

### 5.1 Lags do alvo

Os candidatos que usamos são:

```text
h, h+1, h+2, h+3, m, m+h
```

Mantemos somente os lags maiores ou iguais a `h`. Isso evita que uma previsão de vários passos use valores observados depois da origem.

### 5.2 Janelas do alvo

Deslocamos a série em `h` antes dos cálculos e criamos:

- média móvel de 4, 8 e 12 períodos;
- desvio-padrão móvel de 4, 8 e 12 períodos;
- diferença entre as médias de 4 e 12 períodos (`delta_nivel`).

Duas dessas colunas são redundantes por construção: `media_4` é exatamente a média de `lag_h … lag_h+3`, e `delta_nivel` é exatamente `media_4 − media_12`. O Random Forest não é afetado por isso, mas o PLS é (seção 6.1).

### 5.3 Calendário

Usamos:

- mês;
- dia da semana;
- semana do ano;
- seno e cosseno cíclicos dessas variáveis.

Removemos automaticamente as features que ficam constantes em uma base.

### 5.4 Variáveis externas

- **Conhecidas na origem:** entram com o valor associado à data prevista.
- **Não conhecidas antecipadamente:** entram com deslocamento de `h` períodos.

A configuração final que adotamos após a análise de VIF foi:

| Base | Externas usadas pelo PLS |
|---|---|
| Delhi | `humidity_lag_7`, `wind_speed_lag_7`, `meanpressure_lag_7` |
| Pilgrim's Pride | `Low_lag_5`, `Volume_lag_5` |
| Microsoft | `close_lag_1`, `volume_lag_1` — já disponíveis na origem |
| Sales | `transaction_count_lag_7` |
| Brasil | `matches_played_lag_1`, `friendly_matches_lag_1` |

As decisões completas estão em [`resultados/decisoes_analise.csv`](resultados/decisoes_analise.csv).

## 6. Pré-processamento específico

No PLS, aplicamos três etapas próprias sobre a tabela de `criar_features`:

```text
colunas_pls (uma vez, no treino inicial)
→ WinsorizerQuantil só nas externas → StandardScaler → PLSRegression
```

Reajustamos o winsorizador, o scaler e o PLS em cada origem do walk-forward, apenas com o treino disponível.

### 6.1 Remoção de redundâncias exatas (`colunas_pls`)

Com `media_4` e `delta_nivel`, a matriz de features tem posto `p − 2` em todas as bases; no Brasil, `semana_cos` também é função exata de `semana_sin` no treino (série anual), e o posto cai para `p − 3`. O número de condição da matriz padronizada fica em torno de `10^16`.

Isso importa porque o PLS extrai no máximo tantos componentes úteis quanto o posto de `X`. Componentes além do posto são direções de erro de arredondamento, e os coeficientes associados podem ser enormes. Na nossa versão anterior, a grade ia até o número de colunas, e a validação escolheu 22 componentes em Pilgrim's, cujo posto é 20, e 23 em Microsoft, cujo posto é 22.

`colunas_pls` percorre as colunas na ordem da tabela e mantém cada uma apenas se ela **não** for reproduzida pelas já mantidas. O critério é a fração de variância residual de uma regressão da coluna padronizada sobre as mantidas, que precisa ser maior que `PLS_TOL_REDUNDANCIA = 1e-10`. Fazemos essa seleção somente no treino inicial e a guardamos na base, então validação e teste não participam. O resultado foi:

| Base | Features na tabela | Features no PLS | Removidas |
|---|---:|---:|---|
| Delhi | 25 | 23 | `media_4`, `delta_nivel` |
| Pilgrim's Pride | 22 | 20 | `media_4`, `delta_nivel` |
| Microsoft | 24 | 22 | `media_4`, `delta_nivel` |
| Sales | 23 | 21 | `media_4`, `delta_nivel` |
| Brasil | 20 | 17 | `media_4`, `delta_nivel`, `semana_cos` |

Não perdemos nenhuma informação, porque as colunas removidas são combinações lineares exatas das restantes. A tolerância `1e-10` separa redundância exata de colinearidade forte: lags consecutivos de uma série de preço são muito correlacionados, mas continuam acima do limiar e são mantidos. É justamente esse tipo de colinearidade que o PLS trata bem.

### 6.2 Winsorização das variáveis externas

A base de Delhi tem medições de pressão impossíveis, como `7679,33`, `310,44` e `-3,04`. Para limitar esses erros de medição sem alterar as bases congeladas, usamos o `WinsorizerQuantil`, que aprende em cada janela de treino os quantis

```text
quantil inferior = 0,005
quantil superior = 0,995
```

e limita **somente as variáveis externas** a esse intervalo. Deixamos os lags e as janelas do alvo intactos, e o calendário já é limitado por construção. Fizemos assim porque limitar os lags do alvo pelos quantis do treino impede o modelo de acompanhar níveis novos de uma série com tendência: se o preço atinge uma nova máxima, o lag é cortado e a previsão fica sistematicamente abaixo.

Os quantis são sempre aprendidos no treino da janela, e os de validação e teste nunca são consultados.

### 6.3 Histórico das decisões (transparência sobre o uso do teste)

Nem todas as decisões desta seção foram tomadas **antes** de vermos o teste. O nosso histórico foi este:

1. **Versão 1** — nossa primeira versão, sem winsorização e com grade até 15 componentes. Em Delhi, uma pressão impossível entrou como `meanpressure_lag_7` e gerou uma previsão de `1.252,94 °C` para `2016-04-04` (origem `2016-04-01`). Essa data fica **dentro do teste**.
2. **Versão 2** — winsorização de **todas** as features e grade até o número de colunas. Numa revisão posterior, fizemos uma ablação com os mesmos componentes e sem o clipping, que mostrou MAE de teste de 16.035 em Pilgrim's e 1.033 em Microsoft, com previsões de milhões e de centenas de milhares. Ou seja, o clipping estava mascarando a instabilidade numérica da seção 6.1. Na mesma revisão, também identificamos viés positivo nas séries de preço, causado pelo corte dos lags do alvo.
3. **Versão 3 (atual)** — removemos as redundâncias exatas, limitamos a grade ao posto e passamos a winsorizar só as externas.

As correções da versão 3 têm justificativa independente do resultado. Um número de componentes acima do posto é matematicamente mal definido, e cortar lags de uma série em tendência introduz viés por construção. Na **validação**, restringir o clipping às externas (mantida a remoção de redundâncias) mudou o MAE assim:

| Base | Clipping em todas | Clipping só nas externas |
|---|---:|---:|
| Delhi | 1,915 | 1,916 |
| Pilgrim's Pride | 0,538 | 0,497 |
| Microsoft | 1,349 | 1,195 |
| Sales | 4.765,294 | 4.791,286 |
| Brasil | 2,899 | 2,900 |

Fizemos essa comparação antes de integrar o PLS à versão final do notebook, quando o Brasil usava `m = 14`; hoje ele usa `m = 4`. Nas outras quatro bases, as decisões de análise são as mesmas, e os resultados finais coincidem com os daquela execução.

A melhora não é uniforme: Sales piora levemente e Delhi e Brasil ficam iguais. Mesmo assim, como fizemos todas essas correções depois de observar o teste, **os MAEs de teste do PLS não são estimativas totalmente intocadas**. Não temos dados novos para um holdout independente.

Para evitar um novo ciclo de ajuste guiado pelo teste, congelamos a versão 3. Não usamos os diagnósticos abaixo para escolher nada. Todos usam a versão 3 **sem nenhuma winsorização**:

- **Delhi:** a previsão volta a explodir (`1.264 °C` em `2016-04-04`), então a winsorização ainda é necessária por causa da pressão.
- **Microsoft:** validação de 0,715, contra 1,195 com clipping. `close_lag_1` também é um nível de preço em tendência, e cortá-lo custa precisão.

**Próximo passo que recomendamos:** tratar as pressões impossíveis de Delhi na preparação da base, com uma regra definida antes da modelagem e aplicada aos quatro modelos. Com isso, podemos remover a winsorização do PLS.

### 6.4 Padronização

Depois da winsorização, ajustamos o `StandardScaler` na mesma janela de treino. Passamos `scale=False` ao PLS porque a escala já foi aplicada antes. Isso evita padronização duplicada e mantém o pré-processamento dentro do objeto de pipeline.

### 6.5 Ausência de vazamento

- lags, janelas e externas observacionais desconhecidas usam deslocamento de pelo menos `h`;
- calendário e externas conhecidas podem representar `t` porque já estão disponíveis na origem;
- definimos o conjunto de exógenas, as colunas constantes e as colunas redundantes do PLS somente no treino inicial;
- ajustamos a winsorização e a escala em cada janela;
- reservamos a validação para a escolha de `n_components`;
- o teste não participa da escolha de exógenas, features ou componentes;
- exceção: tomamos as decisões de pré-processamento das seções 6.1 e 6.2 depois de observar resultados no teste (seção 6.3). Os parâmetros aprendidos, como quantis, escala e colunas mantidas, nunca usam validação ou teste.

## 7. Hiperparâmetros e convergência

### 7.1 `n_components`

É o hiperparâmetro preditivo que otimizamos. A grade que usamos é:

```text
1 até min(PLS_MAX_COMPONENTS, número de features não redundantes, amostras da menor janela - 1)
```

Na configuração atual:

```text
PLS_MAX_COMPONENTS = 30
```

Como o número de features não redundantes é igual ao posto de `X`, a grade cobre todos os componentes admissíveis sem ultrapassar o posto (seção 6.1).

**Papel.** `n_components` define quantas direções latentes o modelo extrai antes de fazer a regressão. Cada componente novo é construído sobre o que os anteriores não explicaram. Os primeiros capturam a covariância mais forte entre `X` e `y`; os últimos, relações cada vez mais fracas e, portanto, mais sujeitas a ruído amostral.

**Efeito.** É o controle de viés-variância do modelo:

- **poucos componentes:** o modelo é muito regularizado. As previsões ficam próximas de uma combinação suave das features mais correlacionadas com o alvo, com risco de subajuste;
- **muitos componentes:** o modelo se aproxima da regressão linear por mínimos quadrados sobre todas as features. Ganha flexibilidade, mas os coeficientes ficam mais instáveis sob multicolinearidade;
- **no limite, com `n_components` igual ao posto de `X`,** o PLS reproduz exatamente a regressão linear comum; acima do posto, o ajuste fica mal definido (seção 6.1).

**Evidência empírica: as curvas de validação** (MAE de validação por número de componentes, em [`busca_pls.csv`](resultados/busca_pls.csv)) mostram os três comportamentos:

| Base | k = 1 | Mínimo | k máximo | Formato da curva |
|---|---:|---:|---:|---|
| Delhi | 2,270 | 1,916 (k = 19–20) | 1,919 (k = 23) | Queda rápida até ~9, depois platô |
| Pilgrim's Pride | 1,261 | 0,497 (k = 19) | 0,498 (k = 20) | Queda forte em k = 2, platô a partir de ~14 |
| Microsoft | 3,149 | 1,195 (k = 13) | 1,300 (k = 22) | Forma de U: melhora até 13 e piora depois |
| Sales | 5.005,2 | 4.791,3 (k = 5) | 4.882,1 (k = 21) | Mínimo em 5, irregular depois |
| Brasil | 2,837 | 2,837 (k = 1) | 3,692 (k = 17) | Piora já em k = 2 e não volta ao nível de k = 1 |

- **Microsoft** tem o formato clássico: subajuste com poucos componentes e sobreajuste com muitos.
- **Brasil** está no extremo da variância. Com cerca de 50–60 observações de treino, cada componente extra estima mais parâmetros do que os dados sustentam.
- **Delhi e Pilgrim's** têm muitas observações, e a curva estabiliza: componentes extras quase não mudam o erro, porque o ajuste já está perto da regressão linear completa.

### 7.2 Outros parâmetros e escolhas de configuração

- `scale=False`: a padronização é feita pelo `StandardScaler` do pipeline. Se o PLS padronizasse de novo, o efeito seria nulo, mas a escala ficaria duplicada e mais difícil de auditar. Padronizar em algum lugar é obrigatório: o PLS maximiza covariância, e sem escala comum a feature de maior variância numérica dominaria os componentes (por exemplo, `Volume`, na casa dos milhões, contra preços na casa das dezenas).
- Quantis de winsorização (`0,005` e `0,995`): fixamos esses valores, não os otimizamos, e os aplicamos só às externas (seção 6.2).
- `PLS_TOL_REDUNDANCIA = 1e-10`: fixo; serve só para detectar combinações lineares exatas (seção 6.1).

### 7.3 Parâmetros numéricos fixos

```text
max_iter = 500
tol = 1e-6
```

`max_iter` e `tol` controlam o NIPALS, mas não fazem diferença aqui. Com um único alvo (PLS1), o vetor de pesos sai em forma fechada e o algoritmo para na primeira iteração de cada componente. `ajustar_pls` confere `n_iter_` e rejeita qualquer ajuste que atinja `max_iter`. Em todos os refits persistidos, `iteracoes_max = 1`.

### 7.4 Método de otimização: busca em grade exaustiva

Usamos uma **busca em grade exaustiva**: testamos todos os valores inteiros admissíveis de `n_components`. Escolhemos esse método porque o espaço de busca tem uma única dimensão, é discreto e pequeno (17 a 23 candidatos por base), e cada candidato é barato de avaliar. Busca aleatória ou bayesiana só compensam em espaços grandes ou contínuos. No nosso caso, testar tudo custa pouco e elimina o risco de perder o ótimo. Rodamos os candidatos em paralelo com `joblib`.

Avaliamos cada candidato pelo MAE walk-forward na validação. Configurações com falha recebem valor infinito e são marcadas como inválidas; se todas falharem, a execução é interrompida. Congelamos o candidato finito de menor MAE antes do teste.

A busca detalhada está em [`resultados/busca_pls.csv`](resultados/busca_pls.csv).

## 8. Protocolo walk-forward

Seguimos o mesmo protocolo em todos os modelos. Em cada base:

1. os 30% finais formam o teste;
2. dos 70% anteriores, os 30% finais formam a validação;
3. a validação usa no máximo 30 origens distribuídas pela janela;
4. o teste usa todas as origens completas;
5. as origens avançam em blocos não sobrepostos de tamanho `h`;
6. a janela de treino é expansiva;
7. o modelo, winsorizador e scaler são reajustados em cada origem.

Horizontes:

| Base | `h` | Unidade |
|---|---:|---|
| Delhi | 7 | dias |
| Pilgrim's Pride | 5 | sessões observadas |
| Microsoft | 1 | sessão observada |
| Sales | 7 | dias |
| Brasil | 1 | ano |

## 9. Resultados do PLS

Os MAEs de bases diferentes não devem ser comparados entre si, pois os alvos têm unidades e escalas distintas.

| Base | Features PLS | Componentes | MAE validação | MAE teste | Posição na base (de 4) | Tempo (s) |
|---|---:|---:|---:|---:|---:|---:|
| Delhi | 23 | 20 | 1,916 | 1,981 | 2º | 21,5 |
| Pilgrim's Pride | 20 | 19 | 0,497 | 0,829 | 3º | 70,5 |
| Microsoft | 22 | 13 | 1,195 | 2,495 | 2º | 28,8 |
| Sales | 21 | 5 | 4.791,286 | 5.974,187 | 4º | 20,5 |
| Brasil | 17 | 1 | 2,837 | 2,677 | 2º | 8,7 |

Os tempos são da execução registrada e podem variar conforme máquina e carga do sistema.

Com a grade limitada ao posto, as escolhas variam bastante entre as bases:

- **Sales:** 5 de 21 componentes. É uma redução dimensional forte, e o MAE de validação piora com mais componentes.
- **Microsoft:** 13 de 22. A curva tem mínimo claro em 13 e sobe depois.
- **Delhi** (20 de 23) e **Pilgrim's** (19 de 20): a curva de validação fica praticamente plana a partir de cerca de 13 componentes, com diferenças na terceira casa decimal. O número exato escolhido pouco importa, e a redução dimensional tem pouco efeito sobre o erro nessas bases.
- **Brasil:** 1 de 17. Na série anual curta, uma única direção latente foi a melhor.

Na comparação com os outros modelos, o nosso PLS não venceu em nenhuma base, mas ficou em 2º em três delas, com posição média de 2,6, empatado com o Random Forest ([`placar_modelos.csv`](resultados/placar_modelos.csv)). Em Pilgrim's e Microsoft, os MAEs de SARIMAX e Holt-Winters coincidem com o da previsão ingênua. Nessas bases, as posições deles refletem o fallback do walk-forward para origens com falha, e não o próprio modelo.

O MAE de teste fica acima do de validação em Pilgrim's, Microsoft e Sales. Nas duas séries de preço, o teste cobre níveis mais altos e mais voláteis do que a validação.

### 9.1 Comparação sem escala e possíveis explicações

Como o MAE bruto não é comparável entre bases, usamos duas medidas relativas. Calculamos ambas depois do teste, **apenas como diagnóstico**, e não as usamos em nenhuma escolha:

- **MAE / ingênuo:** razão entre o MAE do PLS e o de uma previsão ingênua que repete o último valor observado na origem para todo o horizonte, nas mesmas origens de teste. Abaixo de 1, o PLS supera o ingênuo;
- **MAE / nível:** MAE dividido pela média do valor absoluto do alvo no teste.

| Base | `h` | MAE / ingênuo | MAE / nível | Autocorrelação do alvo no lag `h` (treino) | Média do alvo: validação → teste |
|---|---:|---:|---:|---:|---|
| Delhi | 7 | 1,02 | 7,6% | 0,925 | 26,4 → 26,0 |
| Pilgrim's Pride | 5 | **1,35** | 3,5% | 0,996 | 9,0 → 23,9 |
| Microsoft | 1 | 0,98 | 1,4% | 0,998 | 108,0 → 182,5 |
| Sales | 7 | **0,82** | 25,1% | 0,557 | 20.877 → 23.798 |
| Brasil | 1 | **0,70** | 26,7% | 0,533 | 7,1 → 10,0 |

O padrão geral é claro: **o PLS acrescenta valor onde o alvo é ruidoso e tem pouca persistência, e acrescenta pouco ou nada onde a série é quase um passeio aleatório.**

- **Microsoft (quase passeio aleatório).** A autocorrelação no lag 1 é de 0,998, então a melhor informação disponível é o último preço, e o PLS praticamente empata com o ingênuo (0,98). O erro relativo é baixo (1,4%) porque o preço de um dia para o outro muda pouco. O MAE dobra da validação para o teste porque o teste ocorre em outro regime: média de 182 contra 108 e desvio-padrão de 35 contra 10. A autocorrelação residual e o viés positivo (seção 11) indicam que o modelo atrasa em relação à alta. A winsorização de `close_lag_1` é uma causa plausível (seção 6.3).
- **Pilgrim's Pride (quase passeio aleatório, `h = 5`).** É o único caso em que o PLS perde para o ingênuo (1,35). Nossas hipóteses:
  1. os coeficientes foram aprendidos em um período de preços baixos (média de 9 na validação) e aplicados a um teste em nível muito mais alto (24). O modelo combina vários lags com pesos de sinais opostos (`lag_5` +5,6, `lag_6` −0,8, `lag_8` −0,5 na escala padronizada), e essas diferenças entre lags amplificam o ruído quando o nível e a volatilidade absoluta crescem;
  2. `Low_lag_5` também é um nível de preço e é winsorizado pelos quantis do treino, o mesmo problema de Microsoft.

  Modelar a variação em relação à origem (`y_t − y_origem`) em vez do nível seria a correção natural, mas mudaria o alvo e o protocolo comum, então deixamos como recomendação.
- **Delhi (sazonal e suave, `h = 7`).** O PLS empata com o ingênuo (1,02). Com horizonte de 7 dias, as features do alvo só vão até `t − 7`, e a temperatura muda devagar, então o último valor já é uma boa previsão. O ciclo anual não aparece nos lags disponíveis (o maior é `lag_37`); ele entra só pelo calendário, o que explica o VIP alto de `semana_cos`. O erro relativo é moderado (7,6%), e validação e teste têm níveis parecidos, então o MAE quase não muda.
- **Sales (lucro diário ruidoso).** A autocorrelação no lag 7 é de apenas 0,557, e o último valor é um mau previsor. As médias móveis suavizam o ruído e puxam a previsão para o nível médio recente, por isso o PLS supera o ingênuo em 18%. Isso também explica os poucos componentes (5): o sinal útil é essencialmente "nível médio recente + volume de transações", e componentes adicionais só ajustam ruído. O erro relativo alto (25%) reflete a variabilidade intrínseca do lucro diário, não uma falha específica do modelo.
- **Brasil (anual, curta).** Há cerca de 60 observações de treino e autocorrelação de 0,533. Com um único componente, o PLS vira essencialmente uma média ponderada do nível recente (médias e desvios móveis lideram o VIP), o que atua como encolhimento e supera o ingênuo em 30%. Mais componentes pioram a validação porque há poucos dados para estimá-los (seção 7.1). O nível do teste é mais alto (10 contra 7), o que explica o viés positivo de 1,04.

Resultados detalhados:

- [`resultados/hiperparametros.csv`](resultados/hiperparametros.csv)
- [`resultados/mae_por_base.csv`](resultados/mae_por_base.csv)
- [`resultados/previsoes.csv`](resultados/previsoes.csv)
- [`resultados/diagnostico_pls.csv`](resultados/diagnostico_pls.csv)

## 10. Interpretação por coeficientes e VIP

### 10.1 Coeficientes padronizados

Como as features são padronizadas (e, no caso das externas, winsorizadas) antes do PLS, os coeficientes representam alterações na previsão associadas a uma unidade padronizada da feature tratada, mantendo as demais relações latentes do modelo.

- sinal positivo: associação com aumento da previsão;
- sinal negativo: associação com redução;
- valor absoluto maior: efeito linear maior na escala padronizada.

O sinal não deve ser lido isoladamente em presença de features altamente correlacionadas e não implica causalidade.

### 10.2 VIP Score

Para a feature `j`, o VIP combina sua participação nos componentes com a quantidade de variação do alvo explicada por cada componente:

```text
VIP_j = sqrt(
  p × [Σ_a SSY_a × (w_ja² / ||w_a||²)] / Σ_a SSY_a
)
```

Nesta implementação:

```text
SSY_a = ||t_a||² × ||q_a||²
```

A referência usada é:

- `VIP >= 1`: feature relevante;
- `VIP < 1`: evidência menor de contribuição global.

O limiar é heurístico e deve ser combinado com coeficiente, contexto e disponibilidade temporal.

### 10.3 Principais resultados

Seguimos o método de interpretação comum do notebook (seção 4.5). Reajustamos o PLS com o número de componentes escolhido, só com a janela de treino (até o início da validação). A **permutation importance** mede quanto o MAE piora na validação quando cada coluna é embaralhada. Não usamos o teste. `media_4` e `delta_nivel` não aparecem porque as removemos como redundâncias exatas.

VIP e permutation respondem perguntas diferentes: o VIP mede quanto a feature pesa na construção dos componentes; a permutation, quanto a previsão depende dela fora da amostra. Features muito correlacionadas dividem o VIP, mas têm permutation baixa, porque a informação continua disponível na "irmã".

- **Delhi:** médias móveis, lags de 7–10 dias e `semana_cos` lideram o VIP (todos perto de 1,42). Na permutation, `semana_cos` (1,20) e `media_12` (1,00) dominam, e os lags individuais ficam perto de zero, porque carregam informação redundante entre si. `meanpressure_lag_7` tem VIP 1,346 e permutation 0,38.
- **Pilgrim's Pride:** `lag_5` e `Low_lag_5` têm o maior VIP (1,506). Na permutation, `lag_5` domina com folga (5,88, contra 0,90 de `Low_lag_5`).
- **Microsoft:** `close_lag_1` tem o maior VIP (1,532), mas a permutation dele é baixa (0,19) comparada à de `lag_1` (1,63). Ou seja, o fechamento anterior é quase redundante com a abertura anterior.
- **Sales:** `transaction_count_lag_7` lidera nas duas medidas (VIP 1,629; permutation 3.876), à frente de `media_12` e `media_8`. É o caso mais claro de externa com contribuição preditiva própria.
- **Brasil:** desvios e médias móveis lideram o VIP. Partidas (VIP 0,884) e amistosos (0,788) ficam abaixo de 1 e têm permutation ligeiramente negativa: embaralhá-las não piora a validação.

Tabela completa: [`resultados/importancia_features.csv`](resultados/importancia_features.csv).

### 10.4 Variáveis externas e disponibilidade temporal

Um VIP alto só tem valor prático se a variável estiver disponível no momento real da previsão. Para cada externa que usamos:

| Base | Externa | VIP | Disponível na origem? | Como entra |
|---|---|---:|---|---|
| Delhi | `meanpressure_lag_7` | 1,346 | Pressão futura não é conhecida | Valor de `t − 7`, já observado na origem |
| Delhi | `humidity_lag_7` | 0,857 | Idem | Valor de `t − 7` |
| Delhi | `wind_speed_lag_7` | 0,543 | Idem | Valor de `t − 7` |
| Pilgrim's | `Low_lag_5` | 1,506 | Mínima futura não é conhecida | Valor de `t − 5` |
| Pilgrim's | `Volume_lag_5` | 1,036 | Volume futuro não é conhecido | Valor de `t − 5` |
| Microsoft | `close_lag_1` | 1,532 | Sim: é o fechamento do pregão anterior, conhecido antes da abertura prevista | Conhecida, já defasada na base |
| Microsoft | `volume_lag_1` | 0,408 | Sim: volume do pregão anterior | Conhecida, já defasada na base |
| Sales | `transaction_count_lag_7` | 1,629 | Transações futuras não são conhecidas | Valor de `t − 7` |
| Brasil | `friendly_matches_lag_1` | 0,788 | Calendário do ano seguinte é só parcialmente conhecido | Valor do ano anterior (opção conservadora) |
| Brasil | `matches_played_lag_1` | 0,884 | Idem | Valor do ano anterior |

Nenhuma externa usa valor observado depois da origem, e todos os VIPs acima valem para previsões que poderiam ser feitas na prática.

Fazemos duas leituras cuidadosas:

- `Low_lag_5` e `close_lag_1` têm VIP alto em grande parte porque são **quase o próprio alvo defasado**: a mínima de Pilgrim's e o fechamento anterior de Microsoft acompanham o nível do preço. O VIP deles é praticamente igual ao de `lag_5` e `lag_1`, mas a permutation é bem menor. Isso indica que carregam informação de nível redundante com os lags do alvo, não um sinal externo independente.
- Em Delhi, `meanpressure_lag_7` fica acima de 1 mesmo com as medições impossíveis. Como a winsorização limita esses valores, o VIP reflete a variação plausível da pressão, que tem sazonalidade anual e, por isso, pode estar funcionando parcialmente como marcador de estação.

Tabela completa: [`resultados/importancia_features.csv`](resultados/importancia_features.csv).

## 11. Resíduos e Ljung-Box

Aplicamos o teste separadamente a cada passo de previsão (`1` a `h`). Juntar os passos em uma única sequência misturaria horizontes com viés e variância diferentes, e isso poderia parecer autocorrelação.

| Base | `h` | Passos rejeitados a 5% | Faixa de p-valor | Leitura |
|---|---:|---:|---|---|
| Brasil | 1 | 0 de 1 | 0,256 | Sem evidência de autocorrelação |
| Delhi | 7 | 1 de 7 | 0,014 – 0,949 | Evidência fraca e isolada (passo 4) |
| Microsoft | 1 | 1 de 1 | < 0,001 | Autocorrelação residual |
| Pilgrim's Pride | 5 | 0 de 5 | 0,055 – 0,689 | Sem evidência de autocorrelação |
| Sales | 7 | 0 de 7 | 0,203 – 0,785 | Sem evidência de autocorrelação |

Com sete testes em Delhi, uma única rejeição a 5% é compatível com acaso e não deve ser lida como falha sistemática.

A evidência clara de dinâmica não capturada fica apenas em Microsoft. O viés é de 0,97, ou seja, o modelo subestima a abertura em média, num período de alta. Uma causa plausível é a winsorização de `close_lag_1`, que também é um nível de preço em tendência (seção 6.3). A outra é a limitação estrutural do PLS: ele é linear no espaço latente e não representa mudanças de regime.

Em Pilgrim's, a versão anterior rejeitava ruído branco nos 5 passos. A rejeição desapareceu e o viés caiu para cerca de 0,04 quando os lags do alvo deixaram de ser limitados. Isso indica que a autocorrelação vinha do pré-processamento, não do PLS em si.

Em Sales, o viés por passo varia de −106 a 1.172, pequeno perto do desvio dos resíduos (cerca de 7.300 a 10.400). A dispersão pesa mais do que o viés.

O Ljung-Box agregado da seção 4.4 do notebook (`ljung_box.csv`) junta todos os passos. Para `h > 1`, ele pode apontar autocorrelação que é só efeito de misturar horizontes. Para o PLS, a leitura que adotamos é esta, por passo.

Tabela completa: [`resultados/ljung_box_pls.csv`](resultados/ljung_box_pls.csv).

## 12. Nossas conclusões sobre o PLS

1. A seleção de exógenas, a remoção de colunas constantes e redundantes e a escolha de `n_components` não consultam o teste. As exceções são as decisões de pré-processamento da seção 6.3, que tomamos depois de observar resultados no teste.
2. Com features redundantes por construção, `n_components` não pode ultrapassar o posto de `X`. Acima dele, o PLS fica numericamente instável, e um clipping agressivo pode esconder o problema.
3. A winsorização ainda é necessária em Delhi por causa das pressões impossíveis. O tratamento correto desses valores é na preparação da base.
4. A validação favoreceu forte redução dimensional em Sales (5 componentes) e no Brasil (1). Em Delhi e Pilgrim's, a curva é plana a partir de cerca de 13 componentes.
5. Em Delhi, Pilgrim's Pride, Microsoft e Sales, ao menos uma externa ficou com VIP acima de 1; no Brasil, as duas ficaram abaixo. Pela permutation, só `transaction_count_lag_7` (Sales) tem contribuição própria clara. `close_lag_1` e `Low_lag_5` são, em grande parte, redundantes com os lags do alvo.
6. Há autocorrelação residual clara só em Microsoft. Delhi tem uma rejeição isolada, e Pilgrim's, Sales e Brasil não mostram evidência de autocorrelação.
7. Na comparação com os outros modelos, o nosso PLS não venceu nenhuma base, ficou em 2º em Delhi, Microsoft e Brasil e teve posição média de 2,6. Frente à previsão ingênua, ganha onde o alvo é ruidoso (Sales e Brasil), empata em Delhi e Microsoft, onde o último valor já é um bom previsor, e perde em Pilgrim's (seção 9.1).

## 13. Reproduzir somente a parte do PLS

O notebook guarda cada combinação base × modelo em `resultados/checkpoints/` e reaproveita o que já existe. Para refazer só o PLS, basta apagar os checkpoints dele e executar o notebook inteiro. SARIMAX, Holt-Winters e Random Forest são carregados dos checkpoints, e o relatório é regenerado no final.

No PowerShell, a partir da raiz do repositório:

```powershell
Remove-Item resultados\checkpoints\*__PLS.pkl
python -m nbconvert --to notebook --execute --inplace "pipeline.ipynb" --ExecutePreprocessor.timeout=-1
```

Dependências principais do PLS:

- Python;
- NumPy;
- pandas;
- matplotlib;
- joblib;
- scikit-learn.

## 14. Dicionário dos artefatos

Resultados comuns aos quatro modelos (filtrar `modelo == "PLS"`):

| Arquivo | Conteúdo |
|---|---|
| [`previsoes.csv`](resultados/previsoes.csv) | Uma linha por data prevista: base, modelo, origem, data, passo, valor real, previsto, resíduo e se a origem falhou |
| [`hiperparametros.csv`](resultados/hiperparametros.csv) | Parâmetros escolhidos, MAE de validação e tempo por base e modelo |
| [`busca.csv`](resultados/busca.csv) | Todas as configurações testadas |
| [`mae_por_base.csv`](resultados/mae_por_base.csv) | MAE de teste e posição dentro da base |
| [`ljung_box.csv`](resultados/ljung_box.csv) | Ljung-Box com os passos agregados |
| [`importancia_features.csv`](resultados/importancia_features.csv) | Coeficiente padronizado, VIP e permutation importance, com externas marcadas |

Específicos do PLS:

| Arquivo | Conteúdo |
|---|---|
| [`busca_pls.csv`](resultados/busca_pls.csv) | Curva de validação: uma linha por base e `n_components` testado |
| [`diagnostico_pls.csv`](resultados/diagnostico_pls.csv) | Componentes, features na tabela e no PLS, features removidas por redundância, externas winsorizadas, iterações, convergência, quantis, corte do ajuste, MAE de validação e de teste, MAE ingênuo e razão MAE / ingênuo |
| [`ljung_box_pls.csv`](resultados/ljung_box_pls.csv) | Ljung-Box por base e passo de previsão: n, lags, viés, desvio, estatística, p-valor e conclusão |

## 15. Checklist de validação

Validamos a execução atual com os seguintes critérios:

- execução completa do notebook sem erros, com relatório regenerado;
- 4.489 previsões PLS, nenhuma origem com falha;
- nenhuma duplicidade por base, origem e data;
- previsões, resíduos, MAEs, coeficientes e VIPs finitos;
- parâmetros finais iguais ao menor MAE da respectiva busca;
- grade de `n_components` limitada ao número de features não redundantes (posto de `X`);
- nenhum ajuste rejeitado por convergência; `iteracoes_max = 1` em todos os refits;
- colunas redundantes do PLS definidas só no treino inicial;
- winsorização aplicada apenas às externas;
- Ljung-Box do PLS calculado por passo de previsão;
- nenhuma previsão fisicamente extrema em Delhi.

## 16. Referências

- [scikit-learn — PLSRegression](https://scikit-learn.org/stable/modules/generated/sklearn.cross_decomposition.PLSRegression.html).
- [scikit-learn — Cross decomposition](https://scikit-learn.org/stable/modules/cross_decomposition.html).
- Geladi, P.; Kowalski, B. R. *Partial least-squares regression: a tutorial*. Analytica Chimica Acta, 185, 1–17, 1986. [DOI](https://doi.org/10.1016/0003-2670%2886%2980028-9).
- Wold, S.; Sjöström, M.; Eriksson, L. *PLS-regression: a basic tool of chemometrics*. Chemometrics and Intelligent Laboratory Systems, 58, 109–130, 2001. [DOI](https://doi.org/10.1016/S0169-7439%2801%2900155-1).