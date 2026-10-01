# Handoff completo — Trabalho de Séries Temporais

> Estado consolidado da sessão de correção, auditoria, bônus, relatório e apresentação.
>
> Diretório do projeto: `tb_2_fz/trabalho_series_temporais_diferente/`

## 1. Resumo executivo

O pipeline compara cinco bases temporais com quatro modelos oficiais — SARIMAX, Holt-Winters, Random Forest e PLS — e mantém Kalman e Fourier+ACF como análises bônus. A execução integral original levou cerca de 170 minutos. Por isso, as correções posteriores foram desenhadas para reutilizar hiperparâmetros, previsões e checkpoints persistidos, sem repetir a busca completa.

O estado final validado tem:

- 5 bases;
- 20 combinações oficiais;
- 31 configurações no benchmark completo, incluindo bônus válidos;
- 32.579 previsões consolidadas;
- zero falhas e zero duplicatas nas previsões;
- Delhi e Sales apenas com o período sazonal padrão após a regra de ganho mínimo;
- relatório técnico Markdown, HTML e PDF;
- Ljung-Box oficial com 20 linhas;
- apresentação interativa separada do relatório técnico;
- Kalman e Fourier+ACF fora do placar oficial.

## 2. Objetivo original e evolução do escopo

### Objetivo inicial

Adicionar ao notebook atualizado, sem remover etapas existentes:

1. regressão dinâmica com filtro de Kalman;
2. investigação sazonal por Fourier+ACF;
3. cenários padrão e bônus nos benchmarks;
4. checkpoints para evitar perda de execuções longas.

### Escopo adicional após a primeira execução

A auditoria da execução encontrou problemas que exigiram correções:

- explosão do Kalman em Delhi por outlier de pressão;
- SARIMAX Delhi também degradado pela mesma exógena;
- intervalos Kalman degenerados ou subcalibrados;
- cache SARIMAX sem identidade de cenário/período;
- cenário Fourier+ACF aceito com ganho sazonal irrelevante;
- benchmarks e placares desatualizados;
- relatório incompatível com a rubrica e com o macOS.

## 3. Bases e configuração temporal

| Base | Alvo | Horizonte | Período padrão final | Observação |
|---|---|---:|---:|---|
| `brasil_vitorias` | `victories` | 1 | 4 | série anual e curta |
| `delhi_temperatura` | `meantemp` | 7 | 30 | temperatura diária; pressão defasada |
| `microsoft_open` | `target_open` | 1 | 30 | sessões observadas de bolsa |
| `pilgrims_close` | `Close` | 5 | 1 | sem período sazonal oficial útil |
| `sales_profit` | `Profit` | 7 | 12 | cauda pesada e quebra de regime |

Fontes/URLs originais e algumas unidades monetárias não estavam documentadas localmente. Os relatórios sinalizam isso explicitamente em vez de inventar metadados.

## 4. Modelos

### Modelos oficiais da avaliação

- SARIMAX;
- Holt-Winters;
- Random Forest;
- PLS.

A avaliação oficial sempre deve conter exatamente:

```text
5 bases × 4 modelos = 20 combinações
```

### Modelos e cenários bônus

- regressão dinâmica com filtro de Kalman;
- período sazonal alternativo identificado por Fourier+ACF;
- combinação Kalman + Fourier+ACF quando elegível.

Os bônus não entram no ranking nem no número de vitórias oficial.

## 5. Protocolo de avaliação

- Validação e teste respeitam ordem temporal.
- O teste usa walk-forward com múltiplas origens.
- Cada origem prevê o horizonte completo da base.
- Os quatro modelos oficiais usam as mesmas origens por base.
- Hiperparâmetros são escolhidos na validação e congelados no teste.
- Lags e janelas respeitam `lag >= horizonte`.
- Exógenas desconhecidas na origem entram defasadas.
- O benchmark usa MAE como métrica principal.
- RMSE, sMAPE, viés, quantis de erro e skill contra baseline são diagnósticos complementares.

## 6. Resultados oficiais finais

### Vencedores por base

| Base | Vencedor oficial | MAE | RMSE | sMAPE | Skill MAE vs. ingênuo |
|---|---|---:|---:|---:|---:|
| Brasil | Holt-Winters | 2,6445 | 3,5346 | 26,73% | 30,74% |
| Delhi | SARIMAX | 1,8252 | 2,3642 | 7,87% | 6,12% |
| Microsoft | SARIMAX | 1,5026 | 2,3951 | 0,84% | 40,88% |
| Pilgrim’s | Holt-Winters | 0,6161 | 0,8862 | 2,64% | aproximadamente 0% |
| Sales | Random Forest | 5.479,24 | 8.133,69 | 35,36% | 25,20% |

### Placar oficial

| Modelo | Vitórias | Posição média |
|---|---:|---:|
| Holt-Winters | 2 | 2,0 |
| SARIMAX | 2 | 2,0 |
| Random Forest | 1 | 3,0 |
| PLS | 0 | 3,0 |

Os MAEs não devem ser somados entre bases: as escalas e unidades são diferentes.

## 7. Foco do grupo: PLS

### Como foi aplicado

1. construção de uma matriz causal compartilhada com o Random Forest;
2. lags do alvo, médias/desvios móveis, delta e calendário;
3. externas conhecidas ou defasadas conforme disponibilidade;
4. imputação/remoção inicial causada por lags;
5. winsorização causal;
6. padronização;
7. escolha de `n_components` na validação;
8. teste walk-forward com componentes congelados.

### Resultado por base

| Base | MAE PLS | Vencedor | Diferença relativa para o vencedor | Skill vs. ingênuo |
|---|---:|---|---:|---:|
| Brasil | 2,6774 | Holt-Winters | +1,24% | 29,88% |
| Delhi | 1,9813 | SARIMAX | +8,55% | -1,91% |
| Microsoft | 2,4950 | SARIMAX | +66,04% | 1,84% |
| Pilgrim’s | 0,8292 | Holt-Winters | +34,59% | -34,59% |
| Sales | 5.974,19 | Random Forest | +9,03% | 18,45% |

### Interpretação

PLS foi competitivo em Brasil e Sales, mas não venceu nenhuma base. Sua utilidade principal foi comprimir conjuntos de features colineares em componentes latentes e fornecer coeficientes/VIP interpretáveis. Não há justificativa para declarar superioridade geral.

Features PLS relevantes encontradas:

- Brasil: dispersão e médias móveis;
- Delhi: médias móveis, `lag_7` e componentes semanais;
- Microsoft: `close_lag_1`, `lag_1` e lags curtos;
- Pilgrim’s: `lag_5` e `Low_lag_5`;
- Sales: `transaction_count_lag_7`, médias móveis e `lag_7`.

Importância não implica causalidade.

## 8. Bônus Fourier+ACF

### Ideia

O bônus combina três evidências calculadas apenas em treino+validação:

- picos do espectro de Fourier;
- autocorrelação em lags candidatos;
- força sazonal obtida por STL.

### Regra conservadora final

```text
ganho_forca = força_candidata - força_padrão

se ganho_forca < 0,05:
    usar período padrão
senão:
    aceitar período candidato, respeitando viabilidade do modelo
```

### Decisões finais

| Base | m padrão | candidato | ganho de força | m efetivo | Decisão |
|---|---:|---:|---:|---:|---|
| Brasil | 4 | 4 | 0,000 | 4 | mesmo período |
| Delhi | 30 | 36 | 0,039 | 30 | rejeitado |
| Microsoft | 30 | 176 | 0,430 | 176 | aprovado |
| Pilgrim’s | 1 | 2275 | 0,572 | 2275 | aprovado sob viabilidade |
| Sales | 12 | 11 | 0,033 | 12 | rejeitado |

### Impacto observado

- Microsoft:
  - Kalman melhorou MAE em aproximadamente `0,126`;
  - PLS melhorou em aproximadamente `0,080`;
  - Random Forest piorou aproximadamente `0,020`.
- Pilgrim’s:
  - Random Forest melhorou aproximadamente `0,0128`;
  - PLS e Kalman pioraram levemente.

Conclusão: identificar sazonalidade mais forte não garante ganho preditivo. A variante deve ser validada por modelo.

## 9. Bônus Kalman

### Estrutura

O modelo implementado é uma regressão dinâmica:

```text
y_t = x_t' β_t + ε_t
β_t = β_(t-1) + η_t
```

- `β_t`: coeficientes que mudam no tempo;
- `Q`: variância do estado;
- `R`: variância da observação;
- `P`: incerteza do estado;
- atualização de Joseph para estabilidade numérica;
- intervalos de 80% e 95%;
- atualização somente quando a observação se torna disponível.

### Correções de robustez

- winsorização causal apenas de features derivadas de externas;
- quantis 0,5%/99,5% combinados com cerca IQR;
- lags e janelas do alvo preservados;
- piso robusto da variância residual;
- proteção de `P0`, `Q` e `R`;
- diagnósticos de variância bruta, piso e variância efetiva.

### Caso Delhi

O dado `meanpressure=7679,33` contaminava `meanpressure_lag_7` e fazia o Kalman prever aproximadamente `1598,60` para uma temperatura real de `32,81`.

Após clipping causal:

```text
7679,33 → aproximadamente 1015,9
```

O erro máximo do Kalman caiu para `11,91` e o MAE para `2,0202`.

### Resultados pontuais

- Kalman não venceu oficialmente nenhuma base.
- Foi competitivo em Delhi e Microsoft.
- Teve desempenho fraco em Sales e Pilgrim’s.
- Fourier+ACF melhorou o Kalman em Microsoft, mas piorou Pilgrim’s.

### Intervalos probabilísticos

| Base/cenário | Cobertura 80% | Cobertura 95% | Veredito |
|---|---:|---:|---|
| Brasil padrão | 0,0% | 12,1% | inaceitável |
| Delhi padrão | 66,7% | 83,2% | subcobertura |
| Microsoft padrão | 37,7% | 49,7% | inaceitável |
| Microsoft Fourier | 21,6% | 31,1% | piorou |
| Pilgrim’s padrão | 78,8% | 91,6% | próximo, ainda abaixo |
| Pilgrim’s Fourier | 72,4% | 87,7% | subcobertura |
| Sales padrão | 78,7% | 90,1% | razoável, abaixo do nominal |

Os intervalos não devem ser usados operacionalmente em Brasil/Microsoft sem calibração conformal na validação.

## 10. Causas-raiz encontradas e correções

### 10.1 Explosão Delhi

**Causa:** outlier de pressão propagado para lag exógeno.

**Correção:** clipping causal de exógenas por origem, com quantis + IQR.

### 10.2 Cache SARIMAX incorreto

**Causa:** cache nomeado apenas por base, permitindo compartilhar configuração entre cenários e períodos diferentes.

**Correção:** chave de cache v3 com base, cenário, `m`, `d`, `D` e hash das exógenas; rejeição explícita de `seasonal_order` incompatível.

### 10.3 Cenários sazonais sem ganho material

**Causa:** aceitar `m` alternativo com diferença pequena na força STL.

**Correção:** ganho absoluto mínimo de `0,05`. Delhi `m=36` e Sales `m=11` foram rejeitados.

### 10.4 Intervalos Kalman degenerados

**Causa:** variâncias residuais próximas de zero em algumas origens.

**Correção:** escala robusta das inovações e piso positivo para variância/P/Q/R.

**Estado:** degeneração numérica removida; subcalibração probabilística ainda permanece.

### 10.5 Sales com MAE alto

**Diagnóstico:** não era explosão numérica. A série tem escala alta, zeros, cauda pesada e quebra real de regime em julho de 2016.

- média aproximada dos 30 dias anteriores: `46.288`;
- média aproximada dos 30 dias seguintes: `8.181`;
- melhor modelo melhora o ingênuo em aproximadamente `25,2%`.

### 10.6 Relatórios desatualizados

**Causa:** geração acoplada ao `reportlab`, fonte Windows e DataFrames em memória.

**Correção:** última célula autocontida, lendo CSVs persistidos e gerando Markdown/HTML/PDF pelo Chrome local.

## 11. Arquivos principais alterados nesta sequência

### `pipeline.ipynb`

Estado final:

- 103 células;
- 57 células de código;
- 46 células Markdown;
- 31 células com outputs preservados;
- última célula com gerador de relatório autocontido;
- sem célula temporária de correção;
- todas as células compilam.

Mudanças funcionais:

- bônus Fourier+ACF;
- filtro de Kalman;
- regra de ganho sazonal `0,05`;
- clipping causal robusto;
- cache SARIMAX v3;
- piso de variância Kalman;
- checkpoints por base/modelo/cenário;
- benchmark com identificação de cenário;
- relatório final alinhado à rubrica.

### `resultados/ljung_box.csv`

Regenerado com exatamente as 20 combinações oficiais. Substitui a versão antiga com 39 linhas e cenários rejeitados.

### Relatórios regenerados

- `relatorio/RELATORIO.md`;
- `relatorio/relatorio.html`;
- `relatorio/relatorio.pdf`;
- `relatorio/checklist_rubrica.csv`;
- `relatorio/figuras/*.png`.

### Novos entregáveis desta solicitação

- `relatorio/apresentacao_interativa.html`;
- `HANDOFF_SESSAO.md`.

## 12. Artefatos de resultados

### Previsões e configuração

- `resultados/previsoes.csv`: previsões consolidadas;
- `resultados/hiperparametros.csv`: configurações escolhidas;
- `resultados/busca.csv`: candidatos avaliados;
- `resultados/decisoes_analise.csv`: decisões temporais por base;
- `resultados/disponibilidade_exogenas.csv`: disponibilidade e uso de externas.

### Benchmarks

- `benchmark_detalhado.csv`;
- `benchmark_por_horizonte.csv`;
- `benchmark_sem_fallback.csv`;
- `mae_por_base.csv`;
- `placar_modelos.csv`.

`benchmark_detalhado.csv` inclui bônus. Para o ranking oficial, filtrar:

```python
cenario_sazonal == "padrao"
modelo_base in {"SARIMAX", "Holt-Winters", "Random Forest", "PLS"}
```

### Bônus

- `benchmark_delta_sazonal.csv`;
- `benchmark_kalman_probabilistico.csv`;
- `bonus_fourier_acf.csv`;
- `bonus_kalman_diagnosticos.csv`;
- `resultados/figuras/bonus_*.png`.

### Interpretação

- `importancia_features.csv`;
- `coeficientes_sarimax.csv`;
- `holtwinters_estado.csv`;
- `holtwinters_suavizacao.csv`;
- `diagnostico_pls.csv`;
- `busca_pls.csv`;
- `ljung_box.csv`;
- `ljung_box_pls.csv`.

### Sales

- `diagnostico_sales_distribuicao.csv`;
- `diagnostico_sales_metricas.csv`;
- `diagnostico_sales_extremos.csv`;
- `diagnostico_sales_maiores_erros.csv`.

## 13. Checkpoints e backups

### Backup do notebook antes dos bônus

```text
pipeline.pre_bonus_backup.ipynb
```

### Checkpoints da primeira rodada bônus

```text
resultados/checkpoints_bonus_v2/
```

Contém também cenários que depois foram rejeitados, como Delhi `m=36` e Sales `m=11`. Não usar como resultado final sem aplicar os filtros atuais.

### Checkpoints corrigidos

```text
resultados/checkpoints_correcao_incremental_v3/
```

Contém os Kalman corrigidos e SARIMAX Delhi corrigido.

### Backup dos benchmarks anteriores à consolidação

```text
resultados/backup_benchmarks_pre_consolidacao_20260929_171224/
```

É histórico; não usar como resultado final.

## 14. Relatório final e rubrica

A última célula de `pipeline.ipynb`:

1. lê artefatos persistidos;
2. não treina nem otimiza modelos;
3. valida as 20 configurações oficiais;
4. separa bônus;
5. regenera figuras e Ljung-Box;
6. gera Markdown e HTML;
7. imprime o mesmo HTML em PDF via Google Chrome headless;
8. promove arquivos de forma transacional;
9. gera checklist da rubrica.

Saídas:

```text
relatorio/RELATORIO.md
relatorio/relatorio.html
relatorio/relatorio.pdf
relatorio/checklist_rubrica.csv
relatorio/figuras/
```

Validação final:

- 23 checks automáticos aprovados;
- PDF com 46 páginas;
- 33 figuras;
- QA visual aprovado;
- uma página visualmente vazia por quebra deliberada de seção, sem perda de conteúdo.

## 15. Como reproduzir sem repetir os 170 minutos

### Atualizar somente os relatórios

1. Abrir `pipeline.ipynb`.
2. Não executar as células de modelagem.
3. Executar somente a última célula.
4. Verificar a mensagem `VALIDAÇÕES DO RELATÓRIO`.
5. Abrir `relatorio/relatorio.html` e `relatorio/relatorio.pdf`.

Pré-requisito usado para PDF no macOS:

```text
/Applications/Google Chrome.app/Contents/MacOS/Google Chrome
```

Markdown e HTML são preservados mesmo se a impressão do PDF falhar.

### Abrir a apresentação

```bash
open relatorio/apresentacao_interativa.html
```

### Validar notebook sem executar

```bash
python3 - <<'PY'
import json
from pathlib import Path
nb = json.loads(Path('pipeline.ipynb').read_text())
for i, cell in enumerate(nb['cells']):
    if cell.get('cell_type') == 'code':
        compile(''.join(cell.get('source', [])), f'cell_{i}', 'exec')
print('Notebook compilado:', len(nb['cells']), 'células')
PY
```

## 16. Arquivos modificados no Git que não podem ser atribuídos integralmente a esta sessão

O `git status` já continha alterações amplas no projeto. Foram observados como modificados:

- `DOCUMENTACAO_PLS.md`;
- `LICENSE`;
- `PLANO.md`;
- arquivos preparados e raw em `bases/`;
- `prepare_bases.ipynb`;
- vários caches e CSVs de resultados.

Essas alterações não foram revertidas. Não assumir que todas foram criadas nesta sessão. Antes de commit, revisar o diff completo e separar o que pertence à entrega.

Não houve commit, push, merge ou publicação externa nesta sessão.

## 17. Arquivos que não devem ser usados como fonte final

- `resultados/checkpoints_bonus_v2/` para cenários posteriormente rejeitados;
- `resultados/backup_benchmarks_pre_consolidacao_20260929_171224/`;
- caches SARIMAX antigos nomeados apenas por base;
- outputs antigos incorporados em células anteriores do notebook quando divergirem dos CSVs consolidados;
- qualquer placar que inclua Kalman/Fourier+ACF como oficial.

Fontes finais preferenciais:

1. `resultados/previsoes.csv`;
2. `resultados/benchmark_detalhado.csv` com filtro oficial;
3. `resultados/hiperparametros.csv`;
4. `resultados/ljung_box.csv` para os 20 pares oficiais;
5. `relatorio/RELATORIO.md` e PDF final.

## 18. Pendências conhecidas

### Metadados

Confirmar manualmente:

- URL original das cinco bases;
- moeda de Microsoft e Pilgrim’s;
- unidade monetária de Sales;
- evidências individuais marcadas como “evidência não localizada”.

### Kalman probabilístico

Implementar calibração conformal na validação antes de qualquer uso operacional dos intervalos, principalmente em Brasil e Microsoft.

### Resíduos

Muitos modelos rejeitam a hipótese de ruído branco no Ljung-Box. Isso indica sinal residual, mas não invalida automaticamente o ranking por MAE. Deve aparecer como limitação.

### Sales

A quebra de regime de julho de 2016 exige abordagem específica se houver objetivo operacional:

- detecção de mudança;
- ponderação temporal;
- regressoras de evento/regime;
- recalibração mais frequente.

### Entrega

Antes do commit:

- revisar `git diff` completo;
- confirmar que fontes/unidades foram preenchidas;
- decidir se backups/checkpoints entram no pacote;
- confirmar nomes e evidências dos integrantes;
- abrir a apresentação no computador que será usado.

## 19. Checksums dos principais entregáveis no momento do handoff

```text
pipeline.ipynb
0f67626f33b8e0597fe5aa25eaf9f095fccb07d18f3a223d764ba1cbf1b177c1

relatorio/RELATORIO.md
e1975159a5d3280490f6b44a9262205268651b487cb661f040ae1d275073d520

relatorio/relatorio.html
cb2cd5e7ba620cdeec11203107c6a9451c9bbe65740c60df595a4064cd1a3de0

relatorio/relatorio.pdf
e46f133b9488e22848a262059274dceca4f2a89f11cb196999b2f378b25c8d3c

resultados/previsoes.csv
aaf54bc6a611421d1bdba8b57b356f2f40a2b90790dbf131efc264a4186e74d3

resultados/benchmark_detalhado.csv
65740b5efe22000001581258a0dd186fd417fd90e21ba0e1951198c132c75b1d

relatorio/apresentacao_interativa.html
d7a630ee3231c35a48c064487896d6ae3bbc8fb33257d9f5e3865bb86d304b21
```

Os checksums mudam se a última célula for executada novamente ou se arquivos forem editados.

## 20. Próximo passo recomendado

1. confirmar fontes, unidades e evidências dos integrantes;
2. revisar a apresentação interativa em tela cheia;
3. ensaiar uma narrativa curta: método → PLS → vencedores → bônus → limitações;
4. executar somente a última célula se algum CSV consolidado mudar;
5. revisar o diff e preparar o commit apenas após aprovação do grupo.
