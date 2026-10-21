# CP6 — Previsão de preço de criptomoeda com LSTM

**Integrante:** Rodrigo Leme — RM550266

Projeto que usa uma Rede Neural Recorrente (**LSTM**) para prever o preço do **Bitcoin** de hora em hora, com dados históricos da **API pública do CoinGecko**. O modelo é treinado com 83 dias de histórico e validado em **7 dias novos**, buscados novamente na API e que não foram usados no treino.

## Estrutura

```
cp2-lstm-cripto/
├── cp2_lstm_bitcoin.ipynb   # notebook comentado com todo o pipeline
├── requirements.txt         # bibliotecas necessárias
├── README.md
└── resultados/              # gerada ao rodar o notebook
    ├── 01_historico.png
    ├── 02_loss.png                       # curva loss / val_loss
    ├── 03_teste_previsto_vs_real.png     # previsão vs real no teste
    ├── 04_experimentos_val_loss.png
    ├── 05_experimentos_rmse.png
    ├── 06_predicao_dados_novos.png       # gráfico final com a predição
    ├── experimentos.csv
    └── modelo_lstm.keras                 # modelo treinado
```

## Pipeline

| Etapa | O que é feito |
|---|---|
| 1. Coleta | `get_historical_data("bitcoin", 90)` chama `/coins/bitcoin/market_chart` (90 dias, dados de hora em hora). Os últimos 7 dias são separados e não entram no treino. |
| 2. Pré-processamento | Resample para 1 hora, interpolação de valores faltantes, `MinMaxScaler` (ajustado só no treino) e janelas de 24 horas → próxima hora. |
| 3. Modelo | Divisão 70% treino / 15% validação / 15% teste em ordem temporal. LSTM(50) + Dense(1), perda MSE, otimizador Adam, 20 épocas. |
| 4. Avaliação | MSE, RMSE e MAE em dólar no conjunto de teste, comparados com o baseline de persistência (próximo = último). |
| 5. Experimentos | `window_size` de 12, 24 e 48 horas, comparando LSTM, média móvel e persistência. |
| 6. Dados novos | Nova chamada à API com `days=7`; previsão hora a hora nesses dados e projeção das próximas 24 horas. |

## Descrição do modelo

```
Entrada (24, 1)  →  LSTM (50 unidades)  →  Dense (1)  →  preço da próxima hora
```

- **Entrada:** preços normalizados das últimas 24 horas
- **Saída:** preço normalizado da próxima hora (convertido de volta para USD)
- **Perda:** MSE · **Otimizador:** Adam · **Épocas:** 20 · **Batch:** 32

## Como executar

**Google Colab (mais fácil):** abra o `cp2_lstm_bitcoin.ipynb` no Colab e clique em *Executar tudo*. As bibliotecas já vêm instaladas.

**Localmente:**

```bash
pip install -r requirements.txt
jupyter notebook cp2_lstm_bitcoin.ipynb
```

Execute as células em ordem. A pasta `resultados/` é criada automaticamente com os gráficos, a tabela de experimentos e o modelo salvo.

> A API pública do CoinGecko não exige chave, mas tem limite de requisições. O notebook faz apenas 2 chamadas (90 dias e 7 dias).

## Principais resultados

> Execução com dados de 23/07/2026 a 21/10/2026. Os números mudam a cada execução, porque a API sempre devolve os últimos 90 dias.

### Conjunto de teste (window_size = 24)

| Modelo | MSE | RMSE (USD) | MAE (USD) |
|---|---|---|---|
| LSTM | 1.263.466 | 1.124,04 | 1.033,59 |
| Persistência | 72.266 | 268,82 | 175,88 |

### Experimentos (RMSE no teste, USD)

| window_size | LSTM | Média móvel | Persistência |
|---|---|---|---|
| 12 | **864,43** | 541,60 | 268,82 |
| 24 | 1.124,04 | 722,17 | 268,82 |
| 48 | 1.041,59 | 936,37 | 268,82 |

A janela de **12 horas** teve o menor erro da LSTM.

### Dados novos (últimos 7 dias: 14/10 a 21/10/2026)

| Modelo | MSE | RMSE (USD) | MAE (USD) |
|---|---|---|---|
| LSTM | 607.412 | 779,37 | 663,03 |
| Persistência | 88.080 | 296,78 | 193,60 |

### Análise

- A LSTM acompanha a **tendência** do preço, mas as previsões ficam um pouco **abaixo** do valor real.
- O motivo principal é que no treino o preço variou entre US$ 62 mil e US$ 82 mil, e no teste chegou a US$ 87 mil. A LSTM tem dificuldade de prever valores fora da faixa que viu no treino.
- O `val_loss` para de cair depois das primeiras épocas, então treinar por mais épocas não melhora o modelo.
- O baseline de persistência teve o menor erro em todos os casos (veja as Observações).


<<<<<<< HEAD
**Histórico coletado da API (a linha vermelha separa os 7 dias novos)**

![historico](resultados/01_historico.png)

**Curva de aprendizado (loss / val_loss)**

![loss](resultados/02_loss.png)

**Previsão vs real no teste**

![teste](resultados/03_teste_previsto_vs_real.png)

**Experimentos com window_size**

![experimentos val_loss](resultados/04_experimentos_val_loss.png)

![experimentos](resultados/05_experimentos_rmse.png)

**Predição em dados novos + projeção das próximas 24 horas**

![predicao](resultados/06_predicao_dados_novos.png)
=======
>>>>>>> 1941d684505b1e572e17ea21c444320357421187
