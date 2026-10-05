# Séries temporais com variáveis externas — Grupo 5 (PLS Regression)

Comparação de SARIMAX, Holt-Winters, Random Forest e PLS Regression (modelo de especialização do grupo) em cinco bases, com validação walk-forward e MAE como métrica. Kalman e Fourier+ACF entram como bônus, fora do placar oficial.

Integrantes: Fernando Paiva, Giovanna Pelati, João Vargas, Matheus Cury e Sophia Gasparetto.

## Entregáveis

| Item do enunciado | Arquivo |
|---|---|
| Relatório final em PDF | `relatorio/relatorio.pdf` |
| Relatório HTML paginado e autocontido | `relatorio/relatorio.html` |
| Arquivo-fonte do relatório | `pipeline.ipynb` (última célula) |
| Códigos da análise | `prepare_bases.ipynb` e `pipeline.ipynb` |
| Bases e fontes | `bases/raw/` (versões congeladas), `bases/*_prepared.csv`; links na seção 3 do relatório |
| Resultados consolidados por MAE | `resultados/mae_por_base.csv` e `resultados/placar_modelos.csv` |
| Registro diário de demandas | `relatorio/registro_demandas.csv` (também na seção 14 do relatório) |

Material complementar: `DOCUMENTACAO_PLS.md` (estudo detalhado do PLS), `relatorio/item_5_1_documentacao_exploracao.md` (exploração das bases), `relatorio/apresentacao_interativa.html` e `n2_series_temporais.pdf` (enunciado).

## Como reproduzir

```bash
python -m pip install -r requirements.txt
```

1. `prepare_bases.ipynb` — lê `bases/raw/` e gera as cinco bases preparadas e os artefatos do item 5.1 em `resultados/item_5_1/`. Leva menos de 2 minutos e reproduz as bases byte a byte.
2. `pipeline.ipynb` — executar do início ao fim (Restart & Run All). Leva cerca de 20 minutos:
   - a seção 2 (análise prévia e bônus Fourier+ACF) é recalculada sempre e é a parte mais lenta (cerca de 11 minutos, quase todo no Fourier+ACF de Pilgrim's);
   - a modelagem (seção 3.10) carrega os 31 checkpoints de `resultados/checkpoints_final/` em vez de reotimizar;
   - a última célula gera `relatorio/RELATORIO.md`, `relatorio.html` e `relatorio.pdf` (o PDF exige Google Chrome ou Microsoft Edge).

Para executar sem abrir o Jupyter:

```bash
python -c "import nbformat; from nbclient import NotebookClient; nb = nbformat.read('pipeline.ipynb', 4); NotebookClient(nb, timeout=None, kernel_name='python3').execute(); nbformat.write(nb, 'pipeline.ipynb')"
```

Para refazer toda a modelagem do zero, trocar `rodar(bases)` por `rodar(bases, refazer=True)` na seção 3.10. A rodada original levou cerca de 3 horas, e Random Forest e Holt-Winters podem variar levemente se as versões das bibliotecas forem diferentes das de `requirements.txt`. Um checkpoint só é aceito se tiver a versão `VERSAO_CHECKPOINT` definida na seção 3.10; qualquer outro é refeito.

## Estrutura

```
bases/raw/                 bases originais congeladas
bases/*_prepared.csv       bases preparadas (prepare_bases.ipynb)
prepare_bases.ipynb        preparação e exploração (item 5.1)
pipeline.ipynb             análise prévia, modelos, comparação e gerador do relatório
resultados/                previsões, buscas, métricas, diagnósticos e checkpoints
resultados/item_5_1/       tabelas e painéis da exploração das bases
resultados/figuras/        figuras do bônus Fourier+ACF
relatorio/                 relatório (MD, HTML, PDF), figuras, registro de demandas e apresentação
```

Principais arquivos de `resultados/`: `previsoes.csv` (todas as previsões fora da amostra), `hiperparametros.csv` (configuração escolhida e tempo), `busca.csv` (candidatos avaliados), `mae_por_base.csv` e `placar_modelos.csv` (20 combinações oficiais), `ljung_box.csv` e `ljung_box_por_passo.csv`, `importancia_features.csv`, `diagnostico_pls.csv`, e os arquivos `_bonus` e `benchmark_*` com as configurações bônus.
