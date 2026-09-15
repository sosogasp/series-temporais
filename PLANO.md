# Trabalho de Séries Temporais — Plano de Execução

**Grupo 5 — modelo de especialização: PLS Regression**
5 bases × 4 modelos (SARIMAX, Holt-Winters, Random Forest, PLS) = 20 combinações.

O plano está montado para **trabalho paralelo**: as frentes avançam ao mesmo tempo e só
se encontram nos pontos de sincronização marcados.

---

## 1. Estrutura da pasta

```
Trabalho_03/
├── PLANO.md
├── pipeline.ipynb      <- notebook narrativo: importa de src/, roda e comenta
├── src/
│   ├── dados.py        <- carregamento e limpeza das 5 bases
│   ├── features.py     <- construção das features
│   ├── validacao.py    <- walk-forward e métricas
│   └── modelos.py      <- os 4 modelos com interface comum
├── dados/              <- bases congeladas (não se edita o arquivo bruto)
├── resultados/         <- previsões, MAE, resíduos, tabelas
└── relatorio/          <- HTML paginado + PDF
```

O código mora em `src/`, não no notebook. Esse é o detalhe que permite 5 pessoas
trabalharem juntas: cada frente edita o seu arquivo `.py` e ninguém dá conflito no
`.ipynb`. O notebook importa, executa e narra.

---

## 2. Sincronização 0 — o contrato (todo mundo junto, antes de tudo)

Nada começa antes disso. Uma sessão do grupo inteiro para fixar:

| Decisão | Onde vive |
|---|---|
| Quais são as 5 bases e de onde vêm | `dados/` + seção 1 do notebook |
| Frequência e período sazonal `m` de cada base | dicionário `BASES` |
| Horizonte de previsão `h` | constante global |
| Nº de origens de previsão e início do teste | constantes globais |
| Disponibilidade de cada variável externa na data da previsão | dicionário das externas |

Três invariantes que valem para os 4 modelos, do começo ao fim: mesmas origens, mesmo
horizonte, mesmo conjunto de teste; nenhuma feature usa informação posterior a `t - h`;
o conjunto de teste não entra na escolha de hiperparâmetro.

**Saída desta etapa:** as constantes preenchidas no topo do notebook e o dicionário
`BASES` fechado. A partir daqui as 5 frentes destravam.

---

## 3. As frentes

| Frente | Responsabilidade |
|---|---|
| **F1 — Infra** | `dados.py` e `validacao.py`: carregar as bases, walk-forward, MAE, formato de saída dos resultados |
| **F2 — Exploração e STL** | Qualidade dos dados, gráficos, decomposição STL e força da sazonalidade das 5 bases |
| **F3 — Features** | `features.py`: lags, janelas, calendário, cíclicas, externas, tratamento de NaN |
| **F4 — Estatísticos** | SARIMAX e Holt-Winters: espaço de busca, otimização e ajuste |
| **F5 — ML e PLS** | Random Forest e PLS + a seção aprofundada do PLS no relatório |

---

## 4. Ondas de execução

### Onda 1 — cinco frentes ao mesmo tempo

Todas partem do contrato e nenhuma depende da outra ainda.

- **F1** escreve `carregar_base()` e o esqueleto do `walk_forward()`. É a frente mais
  urgente: metade do grupo fica bloqueada até ela entregar.
- **F2** roda a exploração e a STL direto nos arquivos brutos. É a frente mais
  independente de todas — pode ir do início ao fim sem esperar ninguém.
- **F3** começa o `features.py` usando uma base qualquer carregada na mão, e troca pelo
  `carregar_base()` quando F1 entregar.
- **F4** estuda e define os espaços de busca de SARIMAX e Holt-Winters. Holt-Winters é
  univariado, então F4 não depende de features em momento nenhum.
- **F5** escreve a seção teórica do PLS — como funciona, intuição, hipóteses, papel do
  `n_components`, por que ele lida bem com features correlacionadas. Vale 20% junto com
  a importância das features e não depende de código nenhum. Começar no dia 1.

### Sincronização 1 — as interfaces

Ponto de encontro curto. F1 e F3 publicam as assinaturas e todo mundo passa a programar
contra elas:

```
carregar_base(nome)            -> DataFrame indexado por data, coluna 'y' + exógenas
construir_features(df, h)      -> X, y alinhados, sem vazamento
walk_forward(base, modelo, h)  -> DataFrame [origem, data, y_real, y_pred]
```

Resultados sempre salvos como `resultados/previsoes_{base}_{modelo}.csv` com essas
colunas. Fixado isso, as 4 frentes de modelo rodam sem conversar entre si.

### Onda 2 — otimização, quatro frentes em paralelo

Um modelo por responsável, os 5 bases cada. Ninguém toca no conjunto de teste.

- SARIMAX: ordens `(p,d,q)(P,D,Q,m)`, AIC/BIC, quais exógenas entram — **F4**
- Holt-Winters: tendência, sazonalidade, amortecimento, suavização — **F4**
- Random Forest: `n_estimators`, `max_depth`, `min_samples_*`, `max_features` — **F5**
- PLS: `n_components`, com as features padronizadas — **F5**

F4 tem dois modelos e F5 também, então F1 e F3 assumem uma parte da otimização assim
que fecharem suas entregas — provavelmente Random Forest, que é a busca mais cara.

Cada frente registra espaço de busca, método, valores escolhidos e justificativa
enquanto otimiza, não depois.

### Sincronização 2 — congelamento

Hiperparâmetros escolhidos viram constantes. Daqui pra frente ninguém mais mexe neles.

### Onda 3 — rodada final

As 20 combinações com os parâmetros congelados. Pode ser dividida por modelo (mesma
divisão da Onda 2) ou rodada de uma vez por quem tiver a máquina mais rápida. Cada
execução salva previsões, resíduos, parâmetros e tempo em `resultados/`.

### Onda 4 — três análises em paralelo

Só dependem dos arquivos em `resultados/`, então rodam simultaneamente:

- **MAE**: tabela por base, ranking dentro de cada base, vitórias e posição média de
  cada modelo. MAE não é somado nem promediado entre bases — escalas diferentes.
  Fecha com a discussão de em que tipo de base cada modelo foi bem ou mal.
- **Resíduos**: resíduos no tempo, ACF e Ljung-Box para cada combinação. A tabela
  consolidada do Ljung-Box vai no corpo do relatório, os gráficos extras no apêndice.
- **Importância das features**: Random Forest com importância nativa e/ou permutation;
  PLS com coeficientes padronizados e/ou VIP scores. Destacar as variáveis externas e
  retomar a disponibilidade delas.

Em paralelo, F5 completa a seção do PLS com o desempenho real nas 5 bases e as
hipóteses para as diferenças.

### Onda 5 — relatório

Cada frente escreve a seção da etapa que tocou — o texto sai em paralelo. Uma pessoa
faz a montagem final: HTML paginado autocontido + PDF com o mesmo conteúdo, nas 14
seções do enunciado. Fecha o `.zip` com PDF, HTML, notebook, códigos, bases (ou fontes),
MAE consolidado e o registro de demandas.

---

## 5. Caminho crítico

```
contrato ──► F1 (walk-forward) ──► otimização ──► rodada final ──► análises ──► relatório
                    │
                    └──► F3 (features) ──┘

F2 (STL)  ──────────────────────────────────────────────────────► relatório
F5 (teoria PLS) ────────────────────────────────────────────────► relatório
```

O gargalo é **F1**. Se o walk-forward atrasar, tudo atrasa. Se sobrar gente na Onda 1,
o lugar de colocar é ali.

F2 e a teoria do PLS são as duas frentes que nunca bloqueiam ninguém — servem de
"trabalho de fundo" para quem estiver esperando alguma entrega.

---

## 6. Registro de demandas

Uma linha por pessoa por dia, preenchida no dia. Vale nota e é usado para ajuste
individual.
