# Modelagem comparativa de séries temporais — Grupo 5 (PLS)

Gerado de artefatos persistidos em 2026-09-29T17:31:47-03:00. Cenário oficial: **padrao**, 5 bases × 4 modelos = 20 combinações.

## 1. Resumo executivo

O conjunto oficial é fechado em cinco bases × quatro modelos (20 combinações), cenário padrao. SARIMAX venceu 2 base(s). MAEs só são comparáveis dentro de cada base; somá-los entre escalas seria metodologicamente errado.

O relatório lê previsões, parâmetros e diagnósticos persistidos. Nenhum ajuste ou otimização ocorre nesta etapa. Kalman e Fourier/ACF aparecem apenas como bônus.

| base | modelo | MAE |
| --- | --- | --- |
| brasil_vitorias | Holt-Winters | 2.644506355581009 |
| delhi_temperatura | SARIMAX | 1.8252365644733708 |
| microsoft_open | SARIMAX | 1.5025858420089313 |
| pilgrims_close | Holt-Winters | 0.6161023035115002 |
| sales_profit | Random Forest | 5479.243924551781 |

| modelo | vitórias | posição_média |
| --- | --- | --- |
| Holt-Winters | 2 | 2.0 |
| SARIMAX | 2 | 2.0 |
| Random Forest | 1 | 3.0 |
| PLS | 0 | 3.0 |

## 2. Identificação dos integrantes e divisão de responsabilidades

A divisão abaixo reproduz demandas localmente registradas. Ausência de arquivo comprobatório é marcada como “evidência não localizada”, sem transformar relato em prova.

| integrante | responsabilidades | evidências |
| --- | --- | --- |
| Fernando Paiva | Organização geral do projeto; Definição das especificações e bases; Adição de elementos na pipeline padrão | evidência não localizada; evidência não localizada; pipeline.ipynb |
| Giovanna Pelati | Organização geral do projeto; Revisão da pipeline e implementação do modelo; Estudo e documentação do PLS | evidência não localizada; pipeline.ipynb; DOCUMENTACAO_PLS.md |
| João Vargas | Organização geral do projeto; Estudo e documentação do projeto; Base do relatório e entendimento da pipeline | evidência não localizada; RELATORIO.md; RELATORIO.md; relatorio.html; relatorio.pdf |
| Matheus Cury | Organização geral do projeto; Arquivos e base do projeto; Resíduos, análise inicial e feature importance | evidência não localizada; pipeline.ipynb; pipeline.ipynb |
| Sophia Gasparetto | Organização geral e pesquisa da base; Limpeza inicial da base; Pipeline de preparação das bases | evidência não localizada; prepare_bases.ipynb; prepare_bases.ipynb |

## 3. Documentação das cinco bases e das variáveis externas

As fontes são identificadas pelos nomes dos arquivos raw fornecidos. Como nenhuma URL de origem foi documentada localmente, o relatório não inventa links. Unidades seguem apenas o que é justificável pelo nome/schema: Delhi em °C; ações em moeda não documentada; Sales em unidade monetária não documentada; Brasil em contagem anual.

| base | alvo | unidade | frequência | período | observações |
| --- | --- | --- | --- | --- | --- |
| delhi_temperatura | meantemp | °C | diária | 2013-01-01 a 2017-04-24 | 1575 |
| pilgrims_close | Close | moeda não documentada localmente | sessões observadas | 1987-12-30 a 2026-09-16 | 9751 |
| microsoft_open | target_open | moeda não documentada localmente | sessões observadas | 2015-04-02 a 2021-03-31 | 1510 |
| sales_profit | Profit | unidade monetária não documentada localmente | diária | 2011-01-01 a 2016-07-31 | 2039 |
| brasil_vitorias | victories | contagem anual de vitórias | anual | 1914-01-01 a 2022-01-01 | 109 |

| base | arquivo preparado | arquivo(s) raw | URL | SHA256 |
| --- | --- | --- | --- | --- |
| delhi_temperatura | daily_delhi_climate_prepared.csv | DailyDelhiClimateTrain.csv; DailyDelhiClimateTest.csv | não documentada localmente | 41f38ab6840f7db11709d070a01f8ea593568225cdb1ce6ac0d34f1a8239549a |
| pilgrims_close | pilgrims_pride_prepared.csv | Pilgrim_s_Pride_Corporation_1987-12-30_2026-09-16.csv | não documentada localmente | b517e7246915d1548d026155efab93e94f44b71cd4446e8068280b3f893784ab |
| microsoft_open | microsoft_stock_prepared.csv | Microsoft_Stock.csv | não documentada localmente | efeee129496b40c1450e532ae7a1017e64353d53c4f9dc95e2673a375362d587 |
| sales_profit | sales_prepared.csv | Sales.csv | não documentada localmente | 507b42cb5c1d4564971f0d09f601f0611ee59fc31f082ee051fc36d1894269dc |
| brasil_vitorias | brazil_prepared.csv | brazil.csv | não documentada localmente | e7d2d3293b13b929f1af8acda01462db4e56f06a7a268cf5745c1ac710b5bd45 |

| base | externas | disponibilidade |
| --- | --- | --- |
| delhi_temperatura | humidity, wind_speed, meanpressure | observadas; entram defasadas em h |
| pilgrims_close | Open, High, Low, Volume | mesma sessão; entram defasadas em h |
| microsoft_open | close_lag_1, volume_lag_1 | sessão anterior; conhecidas na origem para h=1 |
| sales_profit | order_quantity, transaction_count | observadas após o dia; entram defasadas em h |
| brasil_vitorias | matches_played, friendly_matches | apuradas após o ano; entram defasadas em h |

![series_delhi_temperatura](figuras/series_delhi_temperatura.png)

![series_pilgrims_close](figuras/series_pilgrims_close.png)

![series_microsoft_open](figuras/series_microsoft_open.png)

![series_sales_profit](figuras/series_sales_profit.png)

![series_brasil_vitorias](figuras/series_brasil_vitorias.png)

## 4. Limpeza, preparação e Feature Engineering

prepare_bases.ipynb regulariza Delhi e Sales, preserva sessões de bolsa, agrega Brasil por ano e cria lags da Microsoft. RF e PLS compartilham a mesma matriz compatível; externas futuras observadas não são usadas. Outliers foram caracterizados, não apagados automaticamente.

Regras explícitas: lags do alvo em {h, h+1, h+2, h+3, m, m+h}, deduplicados e filtrados para lag ≥ h; média e desvio móvel nas janelas 4, 8 e 12 aplicados depois de shift(h); delta de nível causal; calendário e codificações cíclicas; externas conhecidas entram na data prevista e externas desconhecidas entram defasadas em h; linhas iniciais incompletas são removidas por dropna; no PLS, winsorização causal usa quantis estimados somente na janela de treino e aplicados às externas.

| base | exogena | papel | disponivel_na_origem | como_entra | r | p_valor | significativa | VIF | decisao |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| brasil_vitorias | friendly_matches_lag_1 | defasada | nao | defasada em h | 0.3927 | 0.00045 | True | 2.73 | mantida |
| brasil_vitorias | matches_played_lag_1 | defasada | nao | defasada em h | 0.3751 | 0.000842 | True | 2.73 | mantida |
| delhi_temperatura | humidity_lag_7 | defasada | nao | defasada em h | -0.5328 | 0.0 | True | 1.27 | mantida |
| delhi_temperatura | meanpressure_lag_7 | defasada | nao | defasada em h | -0.8436 | 0.0 | True | 1.2 | mantida |
| delhi_temperatura | wind_speed_lag_7 | defasada | nao | defasada em h | 0.3325 | 0.0 | True | 1.2 | mantida |
| microsoft_open | close_lag_1 | conhecida | sim | valor da propria data | 0.9996 | 0.0 | True | 1.01 | mantida |
| microsoft_open | volume_lag_1 | conhecida | sim | valor da propria data | -0.1106 | 0.000314 | True | 1.01 | mantida |
| pilgrims_close | Low_lag_5 | defasada | nao | defasada em h | 0.995 | 0.0 | True | 1.09 | mantida |
| pilgrims_close | Volume_lag_5 | defasada | nao | defasada em h | 0.2938 | 0.0 | True | 1.09 | mantida |
| pilgrims_close | High_lag_5 | defasada | nao | defasada em h | 0.9949 | 0.0 | True | 1134.6 | removida |
| pilgrims_close | Open_lag_5 | defasada | nao | defasada em h | 0.9946 | 0.0 | True | 2038.8 | removida |
| sales_profit | transaction_count_lag_7 | defasada | nao | defasada em h | 0.8486 | 0.0 | True | 1.0 | mantida |
| sales_profit | order_quantity_lag_7 | defasada | nao | defasada em h | 0.8366 | 0.0 | True | 108.9 | removida |

| base | missing total | duplicatas Date | irregularidade | outliers IQR do alvo | decisão de limpeza |
| --- | --- | --- | --- | --- | --- |
| delhi_temperatura | 0 | 0 | regular após preparação | 0 | ausentes tratados conforme prepare_bases.ipynb; duplicatas rejeitadas/removidas por regra explícita; outliers IQR preservados como observações reais |
| pilgrims_close | 0 | 0 | sessões observadas; intervalos de calendário variam por fins de semana/feriados, sem preencher preços | 161 | ausentes tratados conforme prepare_bases.ipynb; duplicatas rejeitadas/removidas por regra explícita; outliers IQR preservados como observações reais |
| microsoft_open | 0 | 0 | sessões observadas; intervalos de calendário variam por fins de semana/feriados, sem preencher preços | 0 | ausentes tratados conforme prepare_bases.ipynb; duplicatas rejeitadas/removidas por regra explícita; outliers IQR preservados como observações reais |
| sales_profit | 0 | 0 | regular após preparação | 9 | ausentes tratados conforme prepare_bases.ipynb; duplicatas rejeitadas/removidas por regra explícita; outliers IQR preservados como observações reais |
| brasil_vitorias | 0 | 0 | anos regulares após preparação (frequência anual completa) | 1 | ausentes tratados conforme prepare_bases.ipynb; duplicatas rejeitadas/removidas por regra explícita; outliers IQR preservados como observações reais |

## 5. STL, tendência e força da sazonalidade

A STL é calculada somente sobre o período anterior ao teste, usando m persistido (ou m_ideal apenas para visualização quando m=1). A força sazonal é 1−Var(resíduo)/Var(sazonal+resíduo). Fases automáticas apoiam a leitura, mas não provam causalidade nem mudanças estruturais.

| base | m | m_ideal | forca_sazonal |
| --- | --- | --- | --- |
| delhi_temperatura | 30 | 365 | 0.271 |
| pilgrims_close | 1 | 52 | -0.006 |
| microsoft_open | 30 | 252 | 0.201 |
| sales_profit | 12 | 12 | 0.156 |
| brasil_vitorias | 4 | 4 | 0.345 |

| base | fase | inicio | fim | pontos | variacao | variacao_pct |
| --- | --- | --- | --- | --- | --- | --- |
| delhi_temperatura | alta | 2013-01-24 | 2013-05-26 | 123 | 19.81 | 143.4 |
| delhi_temperatura | queda | 2013-06-06 | 2013-08-06 | 62 | -3.76 | -11.2 |
| delhi_temperatura | queda | 2013-09-06 | 2014-01-08 | 125 | -16.74 | -55.6 |
| delhi_temperatura | alta | 2014-01-19 | 2014-06-12 | 145 | 20.18 | 145.8 |
| delhi_temperatura | queda | 2014-06-23 | 2015-01-08 | 200 | -21.32 | -63.0 |
| delhi_temperatura | alta | 2015-01-13 | 2015-05-29 | 137 | 20.91 | 165.8 |
| delhi_temperatura | queda | 2015-06-09 | 2015-07-29 | 51 | -3.33 | -10.0 |
| delhi_temperatura | queda | 2015-09-07 | 2015-12-18 | 103 | -16.05 | -51.6 |
| pilgrims_close | estavel | 1990-05-23 | 1995-05-11 | 1257 | 0.05 | 1.7 |
| pilgrims_close | alta | 1996-06-25 | 1998-12-07 | 620 | 6.06 | 194.8 |
| pilgrims_close | queda | 1999-01-25 | 2000-04-24 | 316 | -4.14 | -47.0 |
| pilgrims_close | alta | 2000-08-09 | 2001-11-23 | 323 | 4.09 | 92.5 |
| pilgrims_close | alta | 2003-02-18 | 2005-05-11 | 563 | 17.81 | 362.9 |
| pilgrims_close | queda | 2007-06-22 | 2009-04-02 | 449 | -21.67 | -94.2 |
| pilgrims_close | alta | 2012-03-29 | 2014-07-15 | 576 | 13.38 | 306.6 |
| microsoft_open | queda | 2015-06-12 | 2015-08-28 | 55 | -1.65 | -3.6 |
| microsoft_open | alta | 2015-09-09 | 2015-12-10 | 66 | 10.45 | 23.6 |
| microsoft_open | alta | 2016-06-06 | 2018-09-21 | 580 | 59.33 | 116.0 |
| microsoft_open | queda | 2018-10-02 | 2019-01-02 | 63 | -4.63 | -4.2 |
| microsoft_open | alta | 2019-01-09 | 2019-05-15 | 88 | 20.07 | 19.0 |
| sales_profit | estavel | 2011-01-01 | 2011-03-14 | 73 | 1337.93 | 21.8 |
| sales_profit | estavel | 2013-01-29 | 2013-06-02 | 125 | 1730.59 | 37.2 |
| sales_profit | alta | 2013-06-03 | 2013-09-23 | 113 | 21001.02 | 329.6 |
| sales_profit | alta | 2014-03-20 | 2014-05-27 | 69 | 8752.21 | 33.2 |
| sales_profit | queda | 2014-05-30 | 2014-08-31 | 94 | -34823.59 | -99.9 |
| sales_profit | estavel | 2014-09-01 | 2014-11-27 | 88 | -21.41 | -99.9 |
| brasil_vitorias | alta | 1916-01-01 | 1921-01-01 | 6 | 0.84 | 82.9 |
| brasil_vitorias | queda | 1923-01-01 | 1927-01-01 | 5 | -1.27 | -76.7 |
| brasil_vitorias | alta | 1928-01-01 | 1930-01-01 | 3 | 0.54 | 110.7 |
| brasil_vitorias | alta | 1935-01-01 | 1939-01-01 | 5 | 1.22 | 158.9 |
| brasil_vitorias | alta | 1942-01-01 | 1950-01-01 | 9 | 1.73 | 80.7 |
| brasil_vitorias | alta | 1953-01-01 | 1962-01-01 | 10 | 6.41 | 171.4 |
| brasil_vitorias | queda | 1963-01-01 | 1967-01-01 | 5 | -3.24 | -34.2 |
| brasil_vitorias | queda | 1971-01-01 | 1974-01-01 | 4 | -0.5 | -7.5 |
| brasil_vitorias | alta | 1977-01-01 | 1980-01-01 | 4 | 1.42 | 22.7 |
| brasil_vitorias | queda | 1981-01-01 | 1984-01-01 | 4 | -1.72 | -22.5 |
| brasil_vitorias | alta | 1985-01-01 | 1988-01-01 | 4 | 2.4 | 39.6 |

![stl_delhi_temperatura](figuras/stl_delhi_temperatura.png)

![stl_pilgrims_close](figuras/stl_pilgrims_close.png)

![stl_microsoft_open](figuras/stl_microsoft_open.png)

![stl_sales_profit](figuras/stl_sales_profit.png)

![stl_brasil_vitorias](figuras/stl_brasil_vitorias.png)

## 6. Protocolo walk-forward

O corte é cronológico: 30% finais para teste e, dentro dos 70% anteriores, 30% para validação. O walk-forward usa as mesmas origens e horizonte por base; o teste permanece fora da seleção. Artefatos persistidos são a evidência reproduzível desta etapa. A cardinalidade abaixo valida a igualdade exata do conjunto de origens entre os quatro modelos de cada base.

| base | obs | h | treino | validacao | teste |
| --- | --- | --- | --- | --- | --- |
| delhi_temperatura | 1575 | 7 | 772 | 331 | 472 |
| pilgrims_close | 9751 | 5 | 4778 | 2048 | 2925 |
| microsoft_open | 1510 | 1 | 740 | 317 | 453 |
| sales_profit | 2039 | 7 | 999 | 428 | 612 |
| brasil_vitorias | 109 | 1 | 53 | 23 | 33 |

| base | modelo | origens | previsões | datas_previstas | horizonte_máximo | origens_paritárias_na_base |
| --- | --- | --- | --- | --- | --- | --- |
| brasil_vitorias | Holt-Winters | 33 | 33 | 33 | 1 | sim |
| brasil_vitorias | PLS | 33 | 33 | 33 | 1 | sim |
| brasil_vitorias | Random Forest | 33 | 33 | 33 | 1 | sim |
| brasil_vitorias | SARIMAX | 33 | 33 | 33 | 1 | sim |
| delhi_temperatura | Holt-Winters | 67 | 469 | 469 | 7 | sim |
| delhi_temperatura | PLS | 67 | 469 | 469 | 7 | sim |
| delhi_temperatura | Random Forest | 67 | 469 | 469 | 7 | sim |
| delhi_temperatura | SARIMAX | 67 | 469 | 469 | 7 | sim |
| microsoft_open | Holt-Winters | 453 | 453 | 453 | 1 | sim |
| microsoft_open | PLS | 453 | 453 | 453 | 1 | sim |
| microsoft_open | Random Forest | 453 | 453 | 453 | 1 | sim |
| microsoft_open | SARIMAX | 453 | 453 | 453 | 1 | sim |
| pilgrims_close | Holt-Winters | 585 | 2925 | 2925 | 5 | sim |
| pilgrims_close | PLS | 585 | 2925 | 2925 | 5 | sim |
| pilgrims_close | Random Forest | 585 | 2925 | 2925 | 5 | sim |
| pilgrims_close | SARIMAX | 585 | 2925 | 2925 | 5 | sim |
| sales_profit | Holt-Winters | 87 | 609 | 609 | 7 | sim |
| sales_profit | PLS | 87 | 609 | 609 | 7 | sim |
| sales_profit | Random Forest | 87 | 609 | 609 | 7 | sim |
| sales_profit | SARIMAX | 87 | 609 | 609 | 7 | sim |

![diagrama_walk_forward](figuras/diagrama_walk_forward.png)

## 7. Modelos, hiperparâmetros e otimização

SARIMAX considera ordens e sazonalidade; Holt-Winters é referência univariada; RF avalia profundidade, árvores e amostragem de features; PLS escolhe componentes latentes. A última célula não repete busca: apenas reporta configurações congeladas e critérios registrados antes do teste. A tabela de busca resume os domínios e quantos candidatos persistidos foram examinados. A grade integral permanece auditável em resultados/busca.csv; a tabela final registra os parâmetros vencedores.

| modelo | hiperparâmetros | domínio/procedimento | seleção |
| --- | --- | --- | --- |
| SARIMAX | p,d,q; P,D,Q; m; exógenas | ordens registradas em resultados/busca.csv | BIC + MAE de validação normalizados |
| Holt-Winters | tendência; sazonalidade; amortecimento; m | combinações aditiva/multiplicativa/ausente compatíveis | MAE de validação |
| Random Forest | n_estimators; max_depth; max_features; min_samples_split; min_samples_leaf | 16 configurações amostradas por base | MAE de validação |
| PLS | n_components | grade de 1 até o posto admissível da matriz | MAE de validação |

| base | modelo_base | candidatos | critério |
| --- | --- | --- | --- |
| brasil_vitorias | Holt-Winters | 9 | MAE validacao |
| brasil_vitorias | PLS | 17 | MAE validacao |
| brasil_vitorias | Random Forest | 16 | MAE validacao |
| brasil_vitorias | SARIMAX | 10 | BIC+MAE normalizados |
| delhi_temperatura | Holt-Winters | 9 | MAE validacao |
| delhi_temperatura | PLS | 23 | MAE validacao |
| delhi_temperatura | Random Forest | 16 | MAE validacao |
| delhi_temperatura | SARIMAX | 10 | BIC+MAE normalizados |
| microsoft_open | Holt-Winters | 9 | MAE validacao |
| microsoft_open | PLS | 22 | MAE validacao |
| microsoft_open | Random Forest | 16 | MAE validacao |
| microsoft_open | SARIMAX | 10 | BIC+MAE normalizados |
| pilgrims_close | Holt-Winters | 3 | MAE validacao |
| pilgrims_close | PLS | 20 | MAE validacao |
| pilgrims_close | Random Forest | 16 | MAE validacao |
| pilgrims_close | SARIMAX | 10 | BIC+MAE normalizados |
| sales_profit | Holt-Winters | 9 | MAE validacao |
| sales_profit | PLS | 21 | MAE validacao |
| sales_profit | Random Forest | 16 | MAE validacao |
| sales_profit | SARIMAX | 10 | BIC+MAE normalizados |

| base | modelo | params | criterio | valor |
| --- | --- | --- | --- | --- |
| delhi_temperatura | Holt-Winters | {'trend': 'add', 'seasonal': None, 'damped': True} | MAE validacao | 2.0103298669817256 |
| delhi_temperatura | Random Forest | {'n_estimators': 600, 'max_depth': None, 'max_features': 'sqrt', 'min_samples_split': 5, 'min_samples_leaf': 2} | MAE validacao | 1.6678284960577148 |
| delhi_temperatura | PLS | {'n_components': 20} | MAE validacao | 1.916147627761664 |
| pilgrims_close | SARIMAX | {'order': (1, 1, 0), 'seasonal_order': (0, 0, 0, 0)} | BIC+MAE normalizados | 1.032937140386145e-06 |
| pilgrims_close | Holt-Winters | {'trend': None, 'seasonal': None, 'damped': False} | MAE validacao | 0.3078068490971673 |
| pilgrims_close | Random Forest | {'n_estimators': 300, 'max_depth': None, 'max_features': 'sqrt', 'min_samples_split': 5, 'min_samples_leaf': 1} | MAE validacao | 0.4972813868123311 |
| pilgrims_close | PLS | {'n_components': 19} | MAE validacao | 0.4970976752801611 |
| microsoft_open | SARIMAX | {'order': (0, 0, 0), 'seasonal_order': (2, 0, 0, 30)} | BIC+MAE normalizados | 0.5142263616689173 |
| microsoft_open | Holt-Winters | {'trend': 'add', 'seasonal': 'mul', 'damped': False} | MAE validacao | 1.295293255822059 |
| microsoft_open | Random Forest | {'n_estimators': 300, 'max_depth': 12, 'max_features': 0.5, 'min_samples_split': 5, 'min_samples_leaf': 1} | MAE validacao | 1.2255158670742587 |
| microsoft_open | PLS | {'n_components': 13} | MAE validacao | 1.1953955086744317 |
| sales_profit | SARIMAX | {'order': (0, 1, 2), 'seasonal_order': (0, 0, 2, 12)} | BIC+MAE normalizados | 0.4607714034289983 |
| sales_profit | Holt-Winters | {'trend': None, 'seasonal': None, 'damped': False} | MAE validacao | 4081.372089788592 |
| sales_profit | Random Forest | {'n_estimators': 600, 'max_depth': 12, 'max_features': 'sqrt', 'min_samples_split': 5, 'min_samples_leaf': 1} | MAE validacao | 4079.1004465409474 |
| sales_profit | PLS | {'n_components': 5} | MAE validacao | 4791.285730283433 |
| brasil_vitorias | SARIMAX | {'order': (0, 1, 2), 'seasonal_order': (0, 0, 2, 4)} | BIC+MAE normalizados | 0.2161717906922945 |
| brasil_vitorias | Holt-Winters | {'trend': None, 'seasonal': 'add', 'damped': False} | MAE validacao | 2.661660187739533 |
| brasil_vitorias | Random Forest | {'n_estimators': 300, 'max_depth': 12, 'max_features': 'sqrt', 'min_samples_split': 2, 'min_samples_leaf': 1} | MAE validacao | 2.8749685990338163 |
| brasil_vitorias | PLS | {'n_components': 1} | MAE validacao | 2.8374431044232136 |
| delhi_temperatura | SARIMAX | {'order': (1, 1, 2), 'seasonal_order': (0, 0, 2, 30)} | BIC+MAE normalizados | 0.3265027116984424 |

## 8. Resultados comparativos por MAE

A tabela oficial tem exatamente 20 linhas. A posição resume desempenho dentro da base; a posição média evita misturar MAEs de unidades distintas. Um vencedor de MAE ainda pode deixar viés ou autocorrelação.

| base | modelo | MAE | n_previsoes | posição |
| --- | --- | --- | --- | --- |
| brasil_vitorias | Holt-Winters | 2.644506355581009 | 33 | 1 |
| brasil_vitorias | PLS | 2.677383307190214 | 33 | 2 |
| brasil_vitorias | SARIMAX | 2.85900802863595 | 33 | 3 |
| brasil_vitorias | Random Forest | 2.904827690407236 | 33 | 4 |
| delhi_temperatura | SARIMAX | 1.8252365644733708 | 469 | 1 |
| delhi_temperatura | Holt-Winters | 1.884667805405557 | 469 | 2 |
| delhi_temperatura | PLS | 1.9812694875036498 | 469 | 3 |
| delhi_temperatura | Random Forest | 2.021252597219686 | 469 | 4 |
| microsoft_open | SARIMAX | 1.5025858420089313 | 453 | 1 |
| microsoft_open | Random Forest | 2.321319901965061 | 453 | 2 |
| microsoft_open | PLS | 2.4949676731214074 | 453 | 3 |
| microsoft_open | Holt-Winters | 2.6158454361594643 | 453 | 4 |
| pilgrims_close | Holt-Winters | 0.6161023035115002 | 2925 | 1 |
| pilgrims_close | SARIMAX | 0.6173617993577347 | 2925 | 2 |
| pilgrims_close | PLS | 0.8292400830585303 | 2925 | 3 |
| pilgrims_close | Random Forest | 0.861541551156187 | 2925 | 4 |
| sales_profit | Random Forest | 5479.243924551781 | 609 | 1 |
| sales_profit | Holt-Winters | 5686.6980415623 | 609 | 2 |
| sales_profit | SARIMAX | 5720.227526740268 | 609 | 3 |
| sales_profit | PLS | 5974.186727838727 | 609 | 4 |

| modelo | vitórias | posição_média |
| --- | --- | --- |
| Holt-Winters | 2 | 2.0 |
| SARIMAX | 2 | 2.0 |
| Random Forest | 1 | 3.0 |
| PLS | 0 | 3.0 |

![comparacao_mae_oficial](figuras/comparacao_mae_oficial.png)

## 9. Análise dos resíduos e teste de Ljung-Box

Resíduo = observado − previsto. O Ljung–Box é calculado sobre erros fora da amostra; p>0,05 significa falta de evidência contra ruído branco nos lags avaliados, não prova independência. Todos os 20 pares oficiais têm série residual e ACF.

| base | modelo | n | lags | viés | desvio | LB | p-valor | ruído branco? |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| brasil_vitorias | Holt-Winters | 33 | 6 | 0.13323886538766286 | 3.5868081080883307 | 7.47922490932379 | 0.27879028459404975 | sim |
| brasil_vitorias | PLS | 33 | 6 | 1.035248104472567 | 3.504180393728382 | 7.763497866047472 | 0.2559471741641174 | sim |
| brasil_vitorias | Random Forest | 33 | 6 | 1.1652060373689161 | 3.8342413788237715 | 8.629425035231534 | 0.19551638301488203 | sim |
| brasil_vitorias | SARIMAX | 33 | 6 | 0.12240031223772908 | 3.594874296371424 | 15.803414266326875 | 0.014848932362475309 | não |
| delhi_temperatura | Holt-Winters | 469 | 10 | 0.001542635252117394 | 2.5029135137885814 | 293.9068387540955 | 3.0154334975622433e-57 | não |
| delhi_temperatura | PLS | 469 | 10 | 0.49493862324908594 | 2.469431138982987 | 519.4673723211632 | 3.045854813916962e-105 | não |
| delhi_temperatura | Random Forest | 469 | 10 | 0.9525375678286746 | 2.314896627447617 | 425.7713387222758 | 3.0584496885781417e-85 | não |
| delhi_temperatura | SARIMAX | 469 | 10 | 0.12570552506974253 | 2.36333520211678 | 286.67946321158183 | 1.0134540839477743e-55 | não |
| microsoft_open | Holt-Winters | 453 | 10 | 0.13483749162608008 | 3.6384341026087275 | 16.765213457619733 | 0.07972334750899243 | sim |
| microsoft_open | PLS | 453 | 10 | 0.9665937642922481 | 3.407287838431024 | 124.7492719214422 | 5.484696975664676e-22 | não |
| microsoft_open | Random Forest | 453 | 10 | 0.6077131936305474 | 3.3683593580898825 | 109.46384067652872 | 6.843722040367123e-19 | não |
| microsoft_open | SARIMAX | 453 | 10 | -0.014173667798438528 | 2.3977256557241975 | 58.55359978773339 | 6.798894559722284e-09 | não |
| pilgrims_close | Holt-Winters | 2925 | 10 | -0.008334695009287369 | 0.8862833369009177 | 1832.7135404768615 | 0.0 | não |
| pilgrims_close | PLS | 2925 | 10 | 0.03725020210606484 | 1.1696374418891538 | 3743.258991052972 | 0.0 | não |
| pilgrims_close | Random Forest | 2925 | 10 | 0.046098491772643774 | 1.1813556804974241 | 5584.144852109286 | 0.0 | não |
| pilgrims_close | SARIMAX | 2925 | 10 | -0.008374729004767628 | 0.8873360650407537 | 1843.3643560250628 | 0.0 | não |
| sales_profit | Holt-Winters | 609 | 10 | 171.07896545860126 | 8500.650159705108 | 137.5857167088176 | 1.315887962518846e-24 | não |
| sales_profit | PLS | 609 | 10 | 551.6021505272356 | 8742.84148460517 | 305.4447817779131 | 1.0973795799018404e-59 | não |
| sales_profit | Random Forest | 609 | 10 | 1689.252160623402 | 7962.879735349487 | 128.52628925999716 | 9.331590907415239e-23 | não |
| sales_profit | SARIMAX | 609 | 10 | 187.5816712628482 | 8527.976228366404 | 143.64920598795135 | 7.522681928121275e-26 | não |

![residuos_delhi_temperatura_SARIMAX](figuras/residuos_delhi_temperatura_SARIMAX.png)

![residuos_delhi_temperatura_Holt-Winters](figuras/residuos_delhi_temperatura_Holt-Winters.png)

![residuos_delhi_temperatura_Random_Forest](figuras/residuos_delhi_temperatura_Random_Forest.png)

![residuos_delhi_temperatura_PLS](figuras/residuos_delhi_temperatura_PLS.png)

![residuos_pilgrims_close_SARIMAX](figuras/residuos_pilgrims_close_SARIMAX.png)

![residuos_pilgrims_close_Holt-Winters](figuras/residuos_pilgrims_close_Holt-Winters.png)

![residuos_pilgrims_close_Random_Forest](figuras/residuos_pilgrims_close_Random_Forest.png)

![residuos_pilgrims_close_PLS](figuras/residuos_pilgrims_close_PLS.png)

![residuos_microsoft_open_SARIMAX](figuras/residuos_microsoft_open_SARIMAX.png)

![residuos_microsoft_open_Holt-Winters](figuras/residuos_microsoft_open_Holt-Winters.png)

![residuos_microsoft_open_Random_Forest](figuras/residuos_microsoft_open_Random_Forest.png)

![residuos_microsoft_open_PLS](figuras/residuos_microsoft_open_PLS.png)

![residuos_sales_profit_SARIMAX](figuras/residuos_sales_profit_SARIMAX.png)

![residuos_sales_profit_Holt-Winters](figuras/residuos_sales_profit_Holt-Winters.png)

![residuos_sales_profit_Random_Forest](figuras/residuos_sales_profit_Random_Forest.png)

![residuos_sales_profit_PLS](figuras/residuos_sales_profit_PLS.png)

![residuos_brasil_vitorias_SARIMAX](figuras/residuos_brasil_vitorias_SARIMAX.png)

![residuos_brasil_vitorias_Holt-Winters](figuras/residuos_brasil_vitorias_Holt-Winters.png)

![residuos_brasil_vitorias_Random_Forest](figuras/residuos_brasil_vitorias_Random_Forest.png)

![residuos_brasil_vitorias_PLS](figuras/residuos_brasil_vitorias_PLS.png)

## 10. Importância das features

RF é interpretado por importância nativa/permutação; PLS por permutation, coeficiente padronizado e VIP quando disponíveis. Importância não implica causalidade. Externas devem ser lidas junto de sua disponibilidade temporal. SARIMAX é interpretado pelos coeficientes, sinal e p-valor; Holt-Winters pelos estados finais de nível, tendência e amplitude sazonal.

| base | modelo | feature | permutation | tipo | disponibilidade | coef_padronizado | vip |
| --- | --- | --- | --- | --- | --- | --- | --- |
| brasil_vitorias | PLS | desvio_8 | 0.0410805559283257 | interna | - | 0.3365197896009145 | 1.511743408207843 |
| brasil_vitorias | PLS | lag_2 | -0.0337502117077554 | interna | - | 0.2200754720626079 | 0.988642137787586 |
| brasil_vitorias | PLS | matches_played_lag_1 | -0.0248686081187402 | externa | defasada | 0.1967086406933604 | 0.8836716297082352 |
| brasil_vitorias | PLS | lag_4 | 0.0222263527120826 | interna | - | 0.2737559192243837 | 1.2297901019019777 |
| brasil_vitorias | PLS | desvio_12 | 0.0199511007189401 | interna | - | 0.3364641615969911 | 1.5114935112601993 |
| brasil_vitorias | Random Forest | desvio_8 | 0.0615362318840583 | interna | - | evidência não localizada | evidência não localizada |
| brasil_vitorias | Random Forest | desvio_4 | 0.0587826086956524 | interna | - | evidência não localizada | evidência não localizada |
| brasil_vitorias | Random Forest | matches_played_lag_1 | 0.0415797101449278 | externa | defasada | evidência não localizada | evidência não localizada |
| brasil_vitorias | Random Forest | friendly_matches_lag_1 | 0.0334492753623192 | externa | defasada | evidência não localizada | evidência não localizada |
| brasil_vitorias | Random Forest | lag_5 | 0.0282608695652178 | interna | - | evidência não localizada | evidência não localizada |
| delhi_temperatura | PLS | semana_cos | 1.2013717012259175 | interna | - | -2.2756582272849135 | 1.418580410985817 |
| delhi_temperatura | PLS | media_12 | 0.9993859645958696 | interna | - | 2.4786305318607864 | 1.428018191203208 |
| delhi_temperatura | PLS | mes_cos | 0.5169713104119219 | interna | - | -1.4258351372708384 | 1.3520132991156315 |
| delhi_temperatura | PLS | semana_sin | 0.5130012562774839 | interna | - | -1.3350921094171446 | 0.4705576237665037 |
| delhi_temperatura | PLS | mes_sin | 0.4170123501141032 | interna | - | 1.1033956656557735 | 0.6159655185026094 |
| delhi_temperatura | Random Forest | semana_cos | 0.2072638837537436 | interna | - | evidência não localizada | evidência não localizada |
| delhi_temperatura | Random Forest | media_4 | 0.1581895293856317 | interna | - | evidência não localizada | evidência não localizada |
| delhi_temperatura | Random Forest | media_8 | 0.0494603572915428 | interna | - | evidência não localizada | evidência não localizada |
| delhi_temperatura | Random Forest | semana | 0.0390546118422976 | interna | - | evidência não localizada | evidência não localizada |
| delhi_temperatura | Random Forest | lag_10 | -0.0257365002550301 | interna | - | evidência não localizada | evidência não localizada |
| microsoft_open | PLS | lag_1 | 1.6301629086102778 | interna | - | 4.552602694050249 | 1.528816252324284 |
| microsoft_open | PLS | close_lag_1 | 0.1901525630974886 | externa | conhecida | 9.107911046924764 | 1.5323741700475637 |
| microsoft_open | PLS | lag_2 | 0.1627509523465637 | interna | - | 0.9135564489634592 | 1.525654689907333 |
| microsoft_open | PLS | semana | 0.0642221665663017 | interna | - | 0.6091789060494347 | 0.1729392602243652 |
| microsoft_open | PLS | lag_4 | -0.0596321358409543 | interna | - | -0.988694011553046 | 1.5216939879962514 |
| microsoft_open | Random Forest | close_lag_1 | 0.1943160799572224 | externa | conhecida | evidência não localizada | evidência não localizada |
| microsoft_open | Random Forest | lag_1 | 0.0266987641834379 | interna | - | evidência não localizada | evidência não localizada |
| microsoft_open | Random Forest | semana_sin | -0.0108455359381038 | interna | - | evidência não localizada | evidência não localizada |
| microsoft_open | Random Forest | desvio_8 | -0.0082369746738908 | interna | - | evidência não localizada | evidência não localizada |
| microsoft_open | Random Forest | lag_2 | 0.0050118925923559 | interna | - | evidência não localizada | evidência não localizada |
| pilgrims_close | PLS | lag_5 | 5.8782190607573925 | interna | - | 4.60483469592232 | 1.506196896865888 |
| pilgrims_close | PLS | Low_lag_5 | 0.8978017032231087 | externa | defasada | 0.9294085016259253 | 1.5058413697205804 |
| pilgrims_close | PLS | media_8 | 0.6412553705766884 | interna | - | 0.7089303158549264 | 1.5028601754402686 |
| pilgrims_close | PLS | lag_6 | 0.2753505345803039 | interna | - | -0.4205908324046366 | 1.5044740171750617 |
| pilgrims_close | PLS | media_12 | 0.0867218861501209 | interna | - | -0.219555350090594 | 1.5013739475374572 |
| pilgrims_close | Random Forest | lag_5 | 0.7187033979417542 | interna | - | evidência não localizada | evidência não localizada |
| pilgrims_close | Random Forest | Low_lag_5 | 0.6805042867132657 | externa | defasada | evidência não localizada | evidência não localizada |
| pilgrims_close | Random Forest | media_4 | 0.6107289477685465 | interna | - | evidência não localizada | evidência não localizada |
| pilgrims_close | Random Forest | media_12 | 0.5024590773476538 | interna | - | evidência não localizada | evidência não localizada |
| pilgrims_close | Random Forest | media_8 | 0.4695143527570609 | interna | - | evidência não localizada | evidência não localizada |
| sales_profit | PLS | transaction_count_lag_7 | 3875.626419214801 | externa | defasada | 2361.5421907750538 | 1.6287696639974911 |
| sales_profit | PLS | media_12 | 2024.8366781393063 | interna | - | 953.6308347521232 | 1.544488400868821 |
| sales_profit | PLS | media_8 | 1617.7573802288375 | interna | - | 802.8555029730161 | 1.542289558704195 |
| sales_profit | PLS | desvio_12 | 539.9462823474487 | interna | - | 612.8702604997347 | 1.067490175400196 |
| sales_profit | PLS | lag_8 | -275.6543146101435 | interna | - | -242.64263869943716 | 1.1752351034829165 |
| sales_profit | Random Forest | media_12 | 1597.0124553981898 | interna | - | evidência não localizada | evidência não localizada |
| sales_profit | Random Forest | media_8 | 1340.5505197261111 | interna | - | evidência não localizada | evidência não localizada |
| sales_profit | Random Forest | transaction_count_lag_7 | 988.156267140164 | externa | defasada | evidência não localizada | evidência não localizada |
| sales_profit | Random Forest | media_4 | 942.8109625029796 | interna | - | evidência não localizada | evidência não localizada |
| sales_profit | Random Forest | lag_10 | 475.39149892830267 | interna | - | evidência não localizada | evidência não localizada |

| base | exogena | disponibilidade | coeficiente | sinal | erro_padrao | p_valor | significativa |
| --- | --- | --- | --- | --- | --- | --- | --- |
| delhi_temperatura | humidity_lag_7 | defasada | 0.0018145493968393 | positivo | 0.0059335582113384 | 0.7597482926028278 | False |
| delhi_temperatura | wind_speed_lag_7 | defasada | 0.0060213921505695 | positivo | 0.0116696781294328 | 0.6058640908481057 | False |
| delhi_temperatura | meanpressure_lag_7 | defasada | 0.013582760673975 | positivo | 0.0265284597354105 | 0.6086460157822551 | False |
| pilgrims_close | Low_lag_5 | defasada | -0.0196529520184285 | negativo | 0.0087052841400759 | 0.0239713072553705 | True |
| pilgrims_close | Volume_lag_5 | defasada | -5.904657732877766e-09 | negativo | 4.4501061009505365e-09 | 0.1845557453613804 | False |
| microsoft_open | close_lag_1 | conhecida | 1.0006358238668138 | positivo | 0.0005027276337216 | 0.0 | True |
| microsoft_open | volume_lag_1 | conhecida | 6.398286006519111e-10 | positivo | 1.1211878532909278e-09 | 0.5682231597143859 | False |
| sales_profit | transaction_count_lag_7 | defasada | -26.78156071200365 | negativo | 6.74472720555504 | 7.16496217036535e-05 | True |
| brasil_vitorias | matches_played_lag_1 | defasada | -0.3918031368228022 | negativo | 0.1908557977933986 | 0.0400846814222587 | True |
| brasil_vitorias | friendly_matches_lag_1 | defasada | 0.1035928396525843 | positivo | 0.2125705840544252 | 0.6260217681039001 | False |

| base | nivel_final | tendencia_final | amplitude_sazonal | amplitude_sobre_nivel_pct | trend | seasonal | damped |
| --- | --- | --- | --- | --- | --- | --- | --- |
| delhi_temperatura | 15.873753075245576 | 2.2601890698038585e-07 | evidência não localizada | evidência não localizada | add | evidência não localizada | True |
| pilgrims_close | 21.277624166420875 | evidência não localizada | evidência não localizada | evidência não localizada | evidência não localizada | evidência não localizada | False |
| microsoft_open | 125.9671984882048 | 0.0824030684462889 | 0.0203855809977544 | 0.0 | add | mul | False |
| sales_profit | 2.4378989947962183e-08 | evidência não localizada | evidência não localizada | evidência não localizada | evidência não localizada | evidência não localizada | False |
| brasil_vitorias | 7.710429602807074 | evidência não localizada | 2.71842569217233 | 35.3 | evidência não localizada | add | False |

![importancia_rf_pls](figuras/importancia_rf_pls.png)

## 11. Explicação aprofundada do modelo escolhido

PLS projeta X em componentes supervisionados que maximizam covariância com y após padronização. É adequado sob multicolinearidade, mas assume relação representável em espaço latente linear. Poucos componentes subajustam; muitos elevam variância e aproximam regressão comum. A quantidade foi selecionada por MAE walk-forward na validação, nunca no teste.

VIP>1 é heurística exploratória. Coeficientes, permutation e estabilidade entre janelas devem ser avaliados em conjunto; projeções não lineares e quebras de regime são limitações materiais.

| base | n_components | n_features_tabela | n_features_pls | features_redundantes | features_winsorizadas | iteracoes_max | convergiu | quantil_clip_inferior | quantil_clip_superior | n_amostras_ajuste | corte_ajuste | mae_teste | mae_ingenuo | mae_sobre_ingenuo | mae_validacao | segundos |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| delhi_temperatura | 20 | 25 | 23 | media_4, delta_nivel | humidity_lag_7, wind_speed_lag_7, meanpressure_lag_7 | 1 | True | 0.005 | 0.995 | 1066 | 2016-01-08 | 1.9812694875036496 | 1.944135603642436 | 1.0191004597578697 | 1.916147627761664 | 3.2 |
| pilgrims_close | 19 | 22 | 20 | media_4, delta_nivel | Low_lag_5, Volume_lag_5 | 1 | True | 0.005 | 0.995 | 6810 | 2015-01-28 | 0.8292400830585303 | 0.6161023037250227 | 1.3459454347838873 | 0.4970976752801611 | 13.7 |
| microsoft_open | 13 | 24 | 22 | media_4, delta_nivel | close_lag_1, volume_lag_1 | 1 | True | 0.005 | 0.995 | 1026 | 2019-06-13 | 2.4949676731214083 | 2.5417439293598236 | 0.9815967864826586 | 1.1953955086744317 | 4.0 |
| sales_profit | 5 | 23 | 21 | media_4, delta_nivel | transaction_count_lag_7 | 1 | True | 0.005 | 0.995 | 1408 | 2014-11-27 | 5974.186727838727 | 7325.653530377668 | 0.8155158721423633 | 4791.285730283433 | 2.8 |
| brasil_vitorias | 1 | 20 | 17 | media_4, delta_nivel, semana_cos | matches_played_lag_1, friendly_matches_lag_1 | 1 | True | 0.005 | 0.995 | 64 | 1989-01-01 | 2.677383307190214 | 3.8181818181818183 | 0.701219437597437 | 2.8374431044232136 | 1.0 |

## 12. Conclusões, limitações e recomendações

Não há modelo universal. Em produção, selecionar por base e monitorar MAE por horizonte, viés, Ljung–Box, drift de entradas e cobertura de intervalos. Recalibração deve ser disparada por degradação sustentada, não por um ponto isolado.

Sales contém quebra de regime: treino, validação e teste exibem níveis/dispersões diferentes. Isso invalida a hipótese de distribuição estável e explica erros assimétricos; recomenda-se detector de mudança, janela adaptativa e avaliação pré/pós-regime.

Os intervalos de Kalman estão subcalibrados: coberturas observadas ficam abaixo dos níveis nominais em bases relevantes. Não usar os intervalos para decisão de risco sem recalibração conformal ou estimação mais robusta de Q/R.

| particao | n | media | mediana | desvio | zeros | outliers_iqr |
| --- | --- | --- | --- | --- | --- | --- |
| treino | 999 | 8729.783783783783 | 7253.0 | 6075.5255307047955 | 1 | 52 |
| validacao | 428 | 20877.418224299065 | 25224.0 | 15471.337450011428 | 119 | 0 |
| teste | 612 | 23798.220588235297 | 24834.0 | 16411.87745065606 | 35 | 0 |

| modelo_base | MAE |
| --- | --- |
| Holt-Winters | 5686.698041562301 |
| PLS | 5974.186727838727 |
| Random Forest | 5479.243924551782 |
| SARIMAX | 5720.227526740267 |

| base | cobertura_80_pct | cobertura_95_pct | largura_95 |
| --- | --- | --- | --- |
| brasil_vitorias | 0.0 | 12.12121212121212 | 1.190993776961924 |
| delhi_temperatura | 66.73773987206823 | 83.15565031982942 | 9.2565260790408 |
| microsoft_open | 37.74834437086093 | 49.668874172185426 | 3.2929807177893595 |
| pilgrims_close | 78.76923076923077 | 91.58974358974359 | 3.888760316877473 |
| sales_profit | 78.6535303776683 | 90.14778325123153 | 27727.028031017948 |

## 13. Referências

Fontes das bases: exclusivamente os nomes raw listados na seção 3; URL de origem não documentada localmente. Não se atribui procedência web sem evidência.

Hyndman, R. J.; Athanasopoulos, G. Forecasting: Principles and Practice, 3rd ed. https://otexts.com/fpp3/

statsmodels: STL, SARIMAX, Exponential Smoothing e Ljung–Box. https://www.statsmodels.org/stable/tsa.html e https://www.statsmodels.org/stable/generated/statsmodels.stats.diagnostic.acorr_ljungbox.html

scikit-learn: PLSRegression, RandomForestRegressor e permutation_importance. https://scikit-learn.org/stable/modules/generated/sklearn.cross_decomposition.PLSRegression.html ; https://scikit-learn.org/stable/modules/generated/sklearn.ensemble.RandomForestRegressor.html ; https://scikit-learn.org/stable/modules/permutation_importance.html

Ljung, G. M.; Box, G. E. P. (1978). On a Measure of Lack of Fit in Time Series Models. Biometrika, 65(2), 297–303. https://doi.org/10.1093/biomet/65.2.297

Enunciado local: conteudo_pdf.md. Preparação: prepare_bases.ipynb. Pipeline e artefatos: pipeline.ipynb e resultados/*.csv.

## 14. Apêndices técnicos e registro de demandas

Bônus — fora do placar oficial: Kalman adiciona estado probabilístico; Fourier/ACF investiga períodos alternativos. Nenhum resultado bônus altera as 20 comparações oficiais. O checklist abaixo cobre a rubrica dos tópicos 14–15, reprodutibilidade, conteúdo isoestrutural e artefatos. O registro de demandas preserva relatos sem fabricar evidência. O grupo declara uso de IA como apoio à implementação e redação; todos os resultados, código e interpretações exigem revisão crítica humana, compreensão e responsabilidade integral dos integrantes.

| base | cobertura_80_pct | cobertura_95_pct | largura_95 |
| --- | --- | --- | --- |
| brasil_vitorias | 0.0 | 12.12121212121212 | 1.190993776961924 |
| delhi_temperatura | 66.73773987206823 | 83.15565031982942 | 9.2565260790408 |
| microsoft_open | 37.74834437086093 | 49.668874172185426 | 3.2929807177893595 |
| pilgrims_close | 78.76923076923077 | 91.58974358974359 | 3.888760316877473 |
| sales_profit | 78.6535303776683 | 90.14778325123153 | 27727.028031017948 |

| base | m_padrao | m_fourier_acf_candidato | ganho_forca_sazonal | m_fourier_acf_efetivo | motivo_escolha_m |
| --- | --- | --- | --- | --- | --- |
| delhi_temperatura | 30 | 36 | 0.0390155292677668 | 30 | ganho_sazonal_0.039016_abaixo_de_0.05 |
| pilgrims_close | 1 | 2275 | 0.5721154000197541 | 2275 | ganho_sazonal_aprovado |
| microsoft_open | 30 | 176 | 0.4300839038218319 | 176 | ganho_sazonal_aprovado |
| sales_profit | 12 | 11 | 0.0329406957194219 | 12 | ganho_sazonal_0.032941_abaixo_de_0.05 |
| brasil_vitorias | 4 | 4 | 0.0 | 4 | mesmo_periodo_padrao |

| data | integrante | demanda | evidência |
| --- | --- | --- | --- |
| 10/09/2026 | Fernando Paiva | Organização geral do projeto | evidência não localizada |
| 15/09/2026 | Fernando Paiva | Definição das especificações e bases | evidência não localizada |
| 17/09/2026 | Fernando Paiva | Adição de elementos na pipeline padrão | pipeline.ipynb |
| 10/09/2026 | Giovanna Pelati | Organização geral do projeto | evidência não localizada |
| 22/09/2026 | Giovanna Pelati | Revisão da pipeline e implementação do modelo | pipeline.ipynb |
| 24/09/2026 | Giovanna Pelati | Estudo e documentação do PLS | DOCUMENTACAO_PLS.md |
| 10/09/2026 | João Vargas | Organização geral do projeto | evidência não localizada |
| 15/09/2026 | João Vargas | Estudo e documentação do projeto | RELATORIO.md |
| 17/09/2026 | João Vargas | Base do relatório e entendimento da pipeline | RELATORIO.md; relatorio.html; relatorio.pdf |
| 10/09/2026 | Matheus Cury | Organização geral do projeto | evidência não localizada |
| 15/09/2026 | Matheus Cury | Arquivos e base do projeto | pipeline.ipynb |
| 24/09/2026 | Matheus Cury | Resíduos, análise inicial e feature importance | pipeline.ipynb |
| 10/09/2026 | Sophia Gasparetto | Organização geral e pesquisa da base | evidência não localizada |
| 15/09/2026 | Sophia Gasparetto | Limpeza inicial da base | prepare_bases.ipynb |
| 17/09/2026 | Sophia Gasparetto | Pipeline de preparação das bases | prepare_bases.ipynb |
