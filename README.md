# Análise de Backtest — regressão sobre operações de scalping

Investiga **quais condições de mercado precedem uma operação vencedora** num backtest de scalping de
mini-índice (WIN, M5), cruzando as entradas do backtest com os candles do período.

## Resultado

**R² (uncentered) = 0,546 · R² ajustado = 0,540** · n = **607 operações** · F = 102,9 · p < 0,001

Sete variáveis preditoras testadas, entre elas:
- **RSI defasado** (`RSI t-1`)
- **variação de preço defasada** (`Var. t-1`)
- **variação de volume defasada** (`Var. Volume t-1`)
- indicadores de candle (`cdl_doji` via `pandas_ta`)

Gráficos de dispersão (`y_test` × cada variável) mostram a relação individual de cada preditor com o resultado.

## Metodologia

1. **Tratamento do backtest:** remove colunas irrelevantes (Tipo, Volume, Ordem, Oferta, Comissão, Swap,
   Saldo), renomeia (`Horário→datetime`, `Preço→price`, `Lucro→profit`) e **arredonda o horário** para
   casar com o timeframe do candle.
2. **Tratamento da base:** junta `<DATE>` + `<TIME>` num datetime único, renomeia OHLCV, calcula **RSI** e
   **detecção de doji**.
3. **Cruzamento:** para cada candle, verifica se o horário coincide com uma **abertura de posição** do
   backtest — só essas linhas entram no dataset.
4. **Modelo:** OLS com `train_test_split` (80/20, `random_state=42`).

## ⚠️ Ressalva metodológica

Usar R² **não-centrado** e `train_test_split` **aleatório** em série temporal é problemático: embaralhar
os dados deixa o modelo "ver o futuro". Um resultado de 0,546 num split aleatório de dados temporais
**superestima** o desempenho.

A versão correta seria um **split temporal** (treinar em período anterior, testar no posterior) — que é a
mesma disciplina de "divisão sem vazamento" usada depois no projeto de detecção de queda por CSI.

## Stack

`Python` · `pandas` · `numpy` · `statsmodels` · `scikit-learn` · `pandas_ta` · `matplotlib`

## Contexto

Estudo de 2023 sobre operações de scalping em mini-índice, cruzando o diário de um backtest com os dados
de mercado do mesmo período.
