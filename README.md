# Análise de Backtest — regressão sobre operações de scalping

Investiga **quais condições de mercado precedem uma operação vencedora** num backtest de scalping de
mini-índice (WIN, M5), cruzando as entradas do backtest com os candles do período.

> ⚠️ **Este repositório foi reproduzido em 31/10/2026 com os dados originais e o resultado de 2023
> NÃO se sustenta.** Os números abaixo são os medidos agora, não os originais. A seção
> "Correção metodológica" explica o que mudou e por quê.

## Dados

| arquivo | conteúdo |
|---|---|
| `teste.xlsx` | diário do backtest (1.662 linhas — entradas, saídas, lucro, saldo) |
| `WIN_M5.xlsx` | candles M5 do mini-índice (100.321 linhas, 2020–2023) |
| `Analysis.ipynb` | tratamento, cruzamento e modelo |

## Resultado medido (31/10/2026)

**312 operações** cruzadas entre o diário e os candles.

| regime de validação | R² |
|---|---|
| **Split aleatório 80/20** (metodologia original de 2023) | **0,126** |
| **Split temporal** (treino 2020-01 → 2022-11 · teste 2022-11 → 2023-09) | **−0,206** |
| Baseline (prever a média do treino) | −0,282 |

**Leitura:** com split temporal o R² out-of-sample é **negativo** — o modelo erra mais que simplesmente
prever a média. Ele **não prevê** resultado de operação.

O valor originalmente publicado (R² 0,546) **não se reproduziu** nem no split aleatório. A diferença
provavelmente vem do critério de cruzamento: a versão de 2023 filtrava aberturas *e* fechamentos de posição,
gerando um conjunto diferente de linhas.

## O achado que sobrevive

Correlação de cada preditor com o resultado da operação:

| variável | correlação com o resultado |
|---|---|
| **RSI defasado (`rsi_t1`)** | **−0,423** |
| Variação de preço (`var_t1`) | −0,147 |
| Variação de volume (`var_vol_t1`) | −0,114 |
| Detecção de doji (`cdl_doji`) | +0,059 |

O **RSI defasado** tem a única correlação relevante, e é **negativa**: RSI alto antes da entrada está
associado a operações **perdedoras**. Isso é consistente com a leitura clássica de sobrecompra — mas também
pode ser artefato da própria regra do backtest, que tende a entrar justamente em rompimento.

Testar essa hipótese exigiria separar a regra de entrada do resultado, o que este dataset não permite.

## Correção metodológica — por que o resultado mudou

O notebook original usava `train_test_split(..., random_state=42)` sobre dados de série temporal.
**Isso embaralha o tempo**: o modelo treina com operações de 2023 e é avaliado com operações de 2020,
tendo "visto o futuro" indiretamente pelas condições de mercado.

A correção é **particionar por tempo**: treinar no passado, testar no futuro. É a mesma disciplina de
divisão sem vazamento usada depois no projeto de detecção de queda por Wi-Fi CSI.

Com split temporal, o desempenho real aparece: **negativo**.

## O que este repositório demonstra

Não que existe um modelo preditivo. Mas que **split aleatório em série temporal infla resultado** — e que
o jeito de saber disso é refazer a validação, não confiar no primeiro R².

Um resultado negativo publicado vale mais que um positivo não reproduzível.

## Metodologia do pipeline

1. **Tratamento do backtest:** remove colunas irrelevantes (Tipo, Volume, Ordem, Oferta, Comissão, Swap,
   Saldo), renomeia (`Horário→datetime`, `Preço→price`, `Lucro→profit`) e trunca o horário para casar com
   o timeframe de 5 minutos.
2. **Tratamento dos candles:** junta `<DATE>` + `<TIME>`, renomeia OHLCV, calcula **RSI(14)** (EMA de
   ganhos/perdas) e detecção de **doji** (corpo < 10% do range).
3. **Cruzamento:** só os candles cujo horário coincide com uma abertura de posição entram no dataset.
4. **Validação:** OLS em `statsmodels`, nos dois regimes (aleatório e temporal).

## Stack

`Python` · `pandas` · `numpy` · `statsmodels` · `scikit-learn` · `openpyxl`

## Reproduzir

```bash
pip install pandas numpy statsmodels scikit-learn openpyxl
jupyter notebook Analysis.ipynb
```

Os dois arquivos `.xlsx` estão no repositório, então o notebook roda ponta a ponta.
Nota: `pandas_ta` (usado em 2023) quebrou em versões recentes do Python — RSI e doji foram
reimplementados com pandas puro para permitir a reprodução.

## Contexto

Estudo de 2023 sobre operações de scalping em mini-índice, revisado e corrigido em 2026.
