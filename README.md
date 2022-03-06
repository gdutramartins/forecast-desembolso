# Modelos para previsão de desembolso no BNDES



Nessa proposta de projeto vamos analisar modelos estatísticos e de inteligência artificial para tentar prever os desembolsos da área de operações indiretas e digitais do BNDES (ADIG). 

Uma operação é chamada de indireta no BNDES quando existe participação de um banco intermediador, ou seja, o BNDES provê o recurso, mas não opera com o beneficiário final, somente com o agente financeiro que será o responsável por intermediar a operação, analisar o risco e no caso de não pagamento da dívida, arcar com o prejuízo. 

Nessa modalidade de atuação existem diversos programas e linhas, para nosso estudo separamos os desembolsos para aquisição de ônibus e caminhões para empresas com os seguintes portes:

* Micro;
* Pequenas;
* Médias.



## Parte I - Carregamento de Séries



Os dados brutos foram fornecidos pela ADIG através de planilhas construídas através da extração de diversas fontes diferentes, tanto internas quanto externas. Abaixo apresento os arquivos utilizados e os dados relevantes por eles informados:

* 0.1 - Exo - IBC-Br.xls - Dados exógenos, sendo o mais importante para o modelo aquele que representa o Índice de Atividade Econômica do Banco Central.
* 0.3 - Exo - ETTJ - progress.xls - Dados exógenos, Estrutura Termo Taxa de Juros e respectivos parâmetros para seu cálculo. A planilha contém taxas aplicadas para remuneração do BNDES nas operações de crédito, além das taxas pré-fixadas utilizadas pelos bancos para obtenção de recursos.
* 1.1 - Endo - FINAME OeC.xls - Dados endógenos e exógenos diretamente relacionados com os desembolsos realizados pelo BNDES. Os principais dados utilizados são: Vendas de Onibus e Caminhões, Aprovação de Operações (aprovação é o estágio anterior a contratação). Essa planilha contém o dado que desejamos prever, o Desembolso.



O arquivo *carregamento-series.ipynb* realiza as seguintes atividades:

* Carregamento das séries do formato excel para estruturas em memória (pandas).
* Filtro das colunas que serão importantes para  a previsão de desembolso em séries Univariada e Multivariada.
* Filtro de registros que são muito antigos e não serão utilizados, já que não temos dados para análise do mercado.
* Filtro de registros futuros que são previsões dos modelos estatísticos.
* Exportação do arquivo CSV com os dados tratados para análise e construção dos modelos.





## Parte II - Analise e Visualização

Arquivo *visualizations.ipynb*



* Visualização dos dados em gráficos.
* Média móvel com janela de 3, 5 e 7 meses.
* Decomposição multiplicativa da série.
* Confronto entre  custo de captação BNDES e mercado.
* Verificação se a série de desembolso ou sua diferença são estacionárias.
* Análise de correlação entre as *features*, essa informação será útil para construir séries multivariadas.
* Análise de Autocorrelação e Autocorrelação Parcial (modelos autoregressivos).



A métrica utilizada para comparar a performance dos modelos foi R2.



## Parte III - Previsão com modelos Autoregressivos 

Arquivo *auto-regressive-pred.ipynb*



* Separação do dataset em treino/teste, sendo que para o treino não é necessário separar features e labels.
* Teste com os seguintes modelos autoregressivos:
  * ARIMA - 0.27 com ordem (0, 2, 11) (p, i, q)
  * SARIMA - 0.11 com ordem (0, 0, 1) (p, i, q) e ordem de sazonalidade (1, 0, 1)  (P, D, Q)
  * SARIMAX - tivemos problemas na execução e optamos por passar para os modelos de suavização e redes neurais.



## Parte IV - Previsão com Suavização Exponencial e Média Móvel

Arquivo *smooth_pred.ipynb*



* Separação do dataset em treino/teste, utilizando janelas de tamanho fixo. Diferente dos modelos autoregressivos que utilizam todo o passado das séries, nos modelos de suavização nós utilizamos janelas.
* Tentamos realizar as previsões tornando a série estacionária através da Diferença.
* R2 foi a métrica utilizada para comparação dos modelos, embora tenhamos medido também MSE e MAE.
* **Resultados**
  * Média Móvel  
    * Melhor janela 12 meses com R2 = 0.07216053724695337
    * Série da diferença com resultados piores.
  * Exponencial Único
    * Melhor janela 3 meses com R2 = 0.10391605340945498
    * Série da diferença com resultados piores.
  * Exponencial Duplo
    * Melhor janela 21 meses com R2=-0.09850542321069233
    * Série da diferença com resultados piores.
  * Exponencial Tripo
    * Melhor janela 38 meses com R2= 0.44334037347245714
    * Não foi possível aplicar a diferença porque os valores precisam ser positivos.



A previsão com Suavização por exponencial triplo superou todos os outros modelos.



## Parte V - Previsão com Redes Neurais Recorrentes (Univariadas)

Arquivo *rnn-pred.ipynb*

* Univariate, somente a feature de desembolso foi utilizada.
* Testamos somente com LSTM.
  * Poucos dados, o treino foi rápido.
  * GRU geralmente tem resultados piores e os resultados da LSTM não foram bons.
  * Os testes também foram realizados com redes bidirecionais.
* Separação do dataset em treino/validação/teste, utilizando janelas de tamanho fixo.
* Dados normalizados para treino, validação e teste.
* MSE foi a métrica utilizada para treinamento, mas R2 foi a métrica para comparação dos modelos.
* Treinamento com armazenamento do modelo com melhor performance nos dados de validação (save_best_only)



**Tabela comparativa que descreve a estrutura da rede e seus resultados** (arquivo *Comparativo-Performance-Modelos.xlsx*)

| Janela | % Teste | % Val | Epochs | Batch | Camada 1             | Camada 2             | Camada 3        | Camada 4        | Camada 5 | Camada 6 | Otimizador | Metrica Loss | R2      | MAE      | MSE         | Comentários                                    |
| ------ | ------- | ----- | ------ | ----- | -------------------- | -------------------- | --------------- | --------------- | -------- | -------- | ---------- | ------------ | ------- | -------- | ----------- | ---------------------------------------------- |
| 24     | 15      | 15    | 33     | 100   | LSTM(200)            | LSTM(200)            | -               | -               | -        | -        | Adam       | mse          | -0,1926 | 86,4408  | 13.646,7935 |                                                |
| 24     | 15      | 15    | 33     | 100   | LSTM(200)            | LSTM(200)            | -               | -               | -        | -        | SGD        | mse          | -2,3200 | 167,9298 | 37.991,4095 | Sempre uma linha reta com SGD                  |
| 24     | 15      | 15    | 88     | 20    | LSTM(250)            | LSTM(250)            | -               | -               | -        | -        | Adam       | mse          | -0,0621 | 88,1555  | 12.153,6093 |                                                |
| 24     | 15      | 15    | 83     | 20    | LSTM(200)            | LSTM(200)            | -               | -               | -        | -        | RMSProp    | mse          | -0,3373 | 100,7166 | 15.303,2783 |                                                |
| 24     | 15      | 15    | ?      | 20    | LSTM(350)            | LSTM(350)            | -               | -               | -        | -        | Adam       | mse          | -0,0518 | 88,4540  | 12.035,6523 |                                                |
| 16     | 15      | 15    | 114    | 20    | LSTM(350)            | LSTM(350)            | -               | -               | -        | -        | Adam       | mse          | -0,1007 | 90,4589  | 12.595,6271 |                                                |
| 24     | 15      | 15    | 115    | 20    | LSTM(350)            | LSTM(350)            | Dense(20, tanh) | -               | -        | -        | Adam       | mse          | -0,0206 | 87,6079  | 11.679,2760 |                                                |
| 24     | 15      | 15    | 115    | 20    | LSTM(350)            | LSTM(350)            | LSTM(350)       | Dense(50, tanh) | -        | -        | Adam       | mse          | -0,1200 |          |             |                                                |
| 24     | 15      | 15    | 115    | 20    | LSTM(350, rd=0.1)    | LSTM(350, rd=0.1)    | Dense(50, tanh) | -               | -        | -        | Adam       | mse          | -0,0675 | 88,2367  | 12.215,9194 | com recurrente dropout maior o resultado piora |
| 24     | 15      | 15    | 13     | 20    | LSTM(350)            | LSTM(350)            | GlobalAvPool1D  | Dense(20, tanh) | -        | -        | Adam       | mse          | -0,9857 | 118,2088 | 22.722,7331 |                                                |
| 24     | 15      | 15    | 83     | 20    | LSTM(350)            | Spatial Dropout(0.1) | LSTM(350)       | Dense(20, tanh) | -        | -        | Adam       | mse          | -0,1191 | 88,3325  | 12.806,3364 |                                                |
| 24     | 15      | 15    | 68     | 100   | Bi-LSTM(100)         | Bi-LSTM(50)          | -               | -               | -        | -        | Adam       | mse          | -0,0581 | 84,6726  | 12.301,6721 |                                                |
| 24     | 15      | 15    | 57     | 20    | Bi-LSTM(350)         | Bi-LSTM(350)         | Dense(20, tanh) | -               | -        | -        | Adam       | mse          | -0,1565 | 89,6787  | 13.233,5188 |                                                |
| 24     | 15      | 15    | 50     | 20    | Bi-LSTM(200)         | Bi-LSTM(100)         | -               | -               | -        | -        | Adam       | mse          | -0,1148 |          |             |                                                |
| 24     | 15      | 15    | 54     | 20    | Bi-LSTM(100, rd=0.1) | Bi-LSTM(50, rd=0.1)  | -               | -               | -        | -        | Adam       | mse          | -0,0755 | 82,7388  | 12.307,0377 |                                                |



## Parte V - Previsão com Redes Neurais Recorrentes (Multivariadas)

Arquivo *rnn-multivariate-pred.ipynb*

* Multivariate, selecionamos as features com maior correlação com desembolso.
* A normalização dos dados de entrada (features e labels) e saída (previsão) precisaram ser tratados de forma diferenciada.
* Resultados muito piores que os conseguidos na parte IV.
* Testamos com inúmeras variações de configuração, mas quase sempre o gráfico plotado era uma linha reta.
