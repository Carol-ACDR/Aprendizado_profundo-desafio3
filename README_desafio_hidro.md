# Previsão e Detecção de Anomalias em Sensores de Deformação

Este projeto explora dados históricos de strain gauges usados em monitoramento estrutural. O notebook analisa a qualidade das medições, identifica valores atípicos, estuda relações temporais entre sensores e compara modelos de previsão de deformação.

## Notebook

- `DesafioHidro.ipynb`

## Dataset

O notebook espera o seguinte arquivo no mesmo diretório:

```text
straingauge_raw.csv
```

Após filtrar observações a partir de 2008, a execução registrada contém 11.615 linhas e três colunas:

- `date_time_dimension_full_date`: data e hora da leitura;
- `sensor_strain_gauge_raw_engineer_vw`: valor de deformação;
- `sensor_strain_gauge_rawfksensor_id`: identificador do sensor.

Existem 24 sensores. Após a remoção de valores ausentes, permanecem 11.143 medições. A maior parte dos sensores possui aproximadamente 480 registros, enquanto o sensor 18 possui apenas 125.

> O CSV não está incluído neste repositório. Antes de redistribuí-lo, verifique sua origem, licença e condições de uso.

## Etapas do projeto

### Análise exploratória

O notebook verifica:

- quantidade de medições por sensor;
- valores ausentes e registros duplicados;
- distribuição das leituras;
- valores inconsistentes;
- outliers pelo intervalo interquartil;
- comportamento histórico individual dos 24 sensores.

Na execução salva foram encontrados 188 outliers pelo critério IQR e nenhuma duplicata de sensor e instante.

### Organização temporal

Os registros são transformados em uma tabela temporal com um sensor por coluna e agregados por média diária. O notebook compara MinMaxScaler, StandardScaler e RobustScaler, além de visualizar os períodos sem leitura.

### Detecção de anomalias

São comparadas três abordagens por sensor:

- intervalo interquartil (IQR);
- Z-score;
- Isolation Forest.

### Análise entre sensores

O projeto calcula autocorrelação e correlação de Spearman com deslocamentos temporais. Para os sensores 1 e 22, a execução registrada encontrou correlação máxima absoluta de aproximadamente `0,5946` com lag de `-20` dias.

### Previsão

Para o sensor 1, o fluxo principal utiliza:

- divisão cronológica de 80% para treino e 20% para teste;
- Isolation Forest para marcação de outliers;
- interpolação e preenchimento de lacunas;
- média móvel causal de sete dias;
- normalização Min-Max;
- janelas com os 60 dias anteriores;
- previsão da deformação sete dias à frente.

São comparados três modelos:

- LSTM;
- XGBoost;
- Random Forest.

O notebook também contém um experimento de transferência entre sensores, treinando uma LSTM com os demais sensores e avaliando no sensor 1.

## Resultados registrados

Os resultados abaixo foram extraídos da última execução salva e não foram recalculados durante a preparação deste repositório.

### Previsão temporal do sensor 1

As métricas abaixo estão na escala normalizada usada pelo primeiro experimento:

| Modelo | MAE | RMSE | R² |
|---|---:|---:|---:|
| LSTM | 0,0417 | 0,0517 | 0,9429 |
| XGBoost | 0,0662 | 0,0868 | 0,8391 |
| Random Forest | 0,0632 | 0,0812 | 0,8593 |

Nesse experimento, a LSTM apresentou o melhor resultado para a previsão sete dias à frente.

### Experimento entre sensores

O notebook também exibe os seguintes resultados na escala original:

| Modelo | MAE | RMSE | R² |
|---|---:|---:|---:|
| LSTM | 0,5090 | 0,6183 | 0,9686 |
| XGBoost | 0,2436 | 0,3363 | 0,9907 |
| Random Forest | 0,1692 | 0,2240 | 0,9959 |

Essas métricas finais devem ser interpretadas apenas como resultados exploratórios, pois a implementação atual mistura modelos e escalas de experimentos diferentes.

## Como executar

1. Instale o Python 3.10 ou superior.
2. Coloque `straingauge_raw.csv` no mesmo diretório do notebook.
3. Instale as dependências:

```bash
pip install jupyter pandas numpy matplotlib seaborn scipy scikit-learn statsmodels tensorflow xgboost
```

4. Abra o notebook:

```bash
jupyter notebook DesafioHidro.ipynb
```

## Limitações importantes

- A série diária contém 4.110 dias sem qualquer leitura, dentro de um total de 4.597 dias.
- A interpolação preenche lacunas muito longas entre campanhas de medição, criando uma grande quantidade de dados sintéticos e séries excessivamente suaves.
- As métricas elevadas podem refletir principalmente a interpolação e a média móvel, e não a capacidade de prever medições reais futuras.
- O experimento final reutiliza modelos clássicos treinados em outra etapa e aplica uma escala diferente, tornando sua comparação inconsistente.
- A versão atual referencia `df_recortado` e `mostrar_metricas` sem defini-los no próprio notebook. Portanto, pode não executar integralmente do início ao fim em uma sessão limpa.
- O processamento do sensor-alvo no experimento entre sensores ajusta transformações sobre toda a série avaliada, o que introduz informação do conjunto de teste.

Para uma avaliação mais confiável, recomenda-se trabalhar apenas com períodos de medição suficientemente densos, limitar o tamanho máximo das lacunas interpoladas, manter um teste temporal independente e encapsular cada modelo com seu próprio pré-processamento.

## Tecnologias

- Python;
- pandas e NumPy;
- Matplotlib e Seaborn;
- scikit-learn;
- statsmodels e SciPy;
- TensorFlow/Keras;
- XGBoost.

## Licença

Nenhuma licença foi definida para este projeto. Adicione um arquivo `LICENSE` antes de permitir reutilização ou redistribuição do código.
