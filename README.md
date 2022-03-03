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

