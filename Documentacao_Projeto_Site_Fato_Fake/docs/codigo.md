# Codificação da Análise dos Dados Eleitorais

### 0 - Importação de Bibliotecas

``` py3
import warnings
import pandas as pd
import os
import seaborn as sns
import matplotlib.pyplot as plt
```

### 1 - Importação de Dados de Bens de Candidatos para o ano eleitoral 2022

``` py3
arquivoBensCandidatos = 'C:\\Temp\\Eleitoral\\BensCandidatos\\bem_candidato_2022_BRASIL.csv'

dfBens = pd.read_csv( arquivoBensCandidatos
                     , sep=';'
                     , engine='python'
                     , encoding='latin1',skiprows=1,header=None
                     , usecols=[2,4,6,8,11,12,13,14,15,16]
                    , names=['ANO_ELEICAO','NM_TIPO_ELEICAO','DS_ELEICAO','SG_UF','SQ_CANDIDATO','NR_ORDEM_CANDIDATO',
                            'CD_TIPO_BEM_CANDIDATO','DS_BEM_CANDIDATO','DS_TIPO_BEM_CANDIDATO','VR_BEM_CANDIDATO']
                    ,dtype={'ANO_ELEICAO':'str','NM_TIPO_ELEICAO':'str','DS_ELEICAO':'str','SG_UF':'str','SQ_CANDIDATO':'str',
                            'NR_ORDEM_CANDIDATO':'str','CD_TIPO_BEM_CANDIDATO':'str','DS_BEM_CANDIDATO':'str',
                            'DS_TIPO_BEM_CANDIDATO':'str','VR_BEM_CANDIDATO':'str'}
                    )

dfBens['VR_BEM_CANDIDATO'] = dfBens['VR_BEM_CANDIDATO'].str.replace(',', '.').astype(float)
```

### 2 - Importação de Dados de Candidatos para os anos eleitorais de 2014 à 2022

``` py3
diretorioCandidatos = 'C:\\Temp\\Eleitoral\\Candidatos\\'

dfCandidatos = pd.DataFrame()

for arquivos in os.listdir(diretorioCandidatos):

    arquivoCandidatos = diretorioCandidatos + arquivos 
    
    df = pd.read_csv(arquivoCandidatos
                               , sep=';'
                               , engine='python'
                               , encoding='latin1'
                               , header=0
                               , usecols=[2,4,7,10,14,15,17,18,39,46,48,50,52,54]
                               , names=['ANO_ELEICAO','NM_TIPO_ELEICAO','DS_ELEICAO','SG_UF','DS_CARGO','SQ_CANDIDATO'
                                        ,'NM_CANDIDATO','NM_URNA_CANDIDATO','SG_UF_NASCIMENTO','DS_GENERO','DS_GRAU_INSTRUCAO',
                                        'DS_ESTADO_CIVIL','DS_COR_RACA','DS_OCUPACAO'
                                        ]
                               , dtype={'ANO_ELEICAO':'str','NM_TIPO_ELEICAO':'str','DS_ELEICAO':'str',
                                        'SG_UF':'str','DS_CARGO':'str','SQ_CANDIDATO':'str','NM_CANDIDATO':'str',
                                        'NM_URNA_CANDIDATO':'str','SG_UF_NASCIMENTO':'str','DS_GENERO':'str',
                                        'DS_GRAU_INSTRUCAO':'str','DS_ESTADO_CIVIL':'str','DS_COR_RACA':'str',
                                        'DS_OCUPACAO':'str'
                                        }
                              )
    
    dfCandidatos = pd.concat([dfCandidatos , df] , ignore_index=True )
```

### 3 - Consolidação dos Bens declarados por Candidato e listagem dos 10 mais ricos

``` py3
resultBensAgregado = dfBens.groupby(['SQ_CANDIDATO'],as_index=False) \
                            .agg(VLR_TOTAL_BEM=('VR_BEM_CANDIDATO','sum')) \
                            .sort_values(by=['VLR_TOTAL_BEM'],ascending=False) \
                            .astype(str)

resultBensAgregado['VLR_TOTAL_BEM_C'] = resultBensAgregado['VLR_TOTAL_BEM'].astype(float)

resultFinalBensCandidatos = pd.merge(left=resultBensAgregado
, right=dfCandidatos
, left_on='SQ_CANDIDATO'
, right_on='SQ_CANDIDATO'
)

resultFinalBensCandidatos[['ANO_ELEICAO','SQ_CANDIDATO','NM_CANDIDATO','SG_UF','DS_CARGO','VLR_TOTAL_BEM']].head(10)
```

### 4 - Criação de De-Para e Cálculo do Percentual de Candidatos por Raça/Cor

``` py3
dfCandidatos.loc[dfCandidatos['DS_COR_RACA'].isin(['NÃO DIVULGÁVEL'
                                                   ,'NÃO INFORMADO'
                                                   ,'INDÍGENA'
                                                   ,'AMARELA'
                                                   ,'BRANCA'
                                                   ,'PARDA'
                                                   ,'PRETA']), 'DS_COR_RACA_TRATADA'] = 'COD_TOTAL'
dfCandidatos.loc[dfCandidatos['DS_COR_RACA'].isin(['PARDA','PRETA']), 'DS_COR_RACA_TRATADA'] = 'COR_NEGRA'
dfCandidatos.loc[dfCandidatos['DS_COR_RACA'].isin(['BRANCA']), 'DS_COR_RACA_TRATADA'] = 'COR_BRANCA'
dfCandidatos.loc[dfCandidatos['DS_COR_RACA'].isin(['AMARELA']), 'DS_COR_RACA_TRATADA'] = 'COR_AMARELA'
dfCandidatos.loc[dfCandidatos['DS_COR_RACA'].isin(['INDÍGENA']), 'DS_COR_RACA_TRATADA'] = 'COR_INDÍGENA'
dfCandidatos.loc[dfCandidatos['DS_COR_RACA'].isin(['NÃO DIVULGÁVEL','NÃO INFORMADO']), 
                 'DS_COR_RACA_TRATADA'] = 'COR_NAO_DIVULGAVEL'

dfCanidatoCorRaca = dfCandidatos.groupby(['ANO_ELEICAO'
                                          ,'DS_COR_RACA_TRATADA'],as_index=False).agg(QTD_CANDIDATO=('SQ_CANDIDATO','count'))

dfCandidatos.loc[dfCandidatos['DS_COR_RACA'].isin(['NÃO DIVULGÁVEL'
                                                   ,'NÃO INFORMADO'
                                                   ,'INDÍGENA','AMARELA'
                                                   ,'BRANCA'
                                                   ,'PARDA'
                                                   ,'PRETA']), 'DS_COR_RACA_TRATADA'] = 'TOTAL'

dfCanidatoCorRacaoTotal = dfCanidatoCorRaca.groupby(['ANO_ELEICAO'],as_index=False) \
                                                .agg(QTD_CANDIDATO_TOTAL=('QTD_CANDIDATO','sum'))

resultCandidatoCorRaca = pd.merge(left=dfCanidatoCorRaca
, right=dfCanidatoCorRacaoTotal
, left_on='ANO_ELEICAO'
, right_on='ANO_ELEICAO'
)

resultCandidatoCorRaca['PERCENTUAL'] = (resultCandidatoCorRaca['QTD_CANDIDATO'] /
                                        resultCandidatoCorRaca['QTD_CANDIDATO_TOTAL']) * 100

resultgrafico = resultCandidatoCorRaca[resultCandidatoCorRaca['DS_COR_RACA_TRATADA'] == 'COR_NEGRA'] \
                    [['ANO_ELEICAO'
                      ,'DS_COR_RACA_TRATADA'
                      ,'PERCENTUAL']]

```

### 4.1 - Gráfico - Criação de De-Para e Cálculo do Percentual de Candidatos por Raça/Cor

``` py3
#criando a fig e o ax no matplotlib
fig, ax = plt.subplots(figsize=(8,5))
#criando novamente o gráfico
sns.barplot(x='ANO_ELEICAO',y='PERCENTUAL',data=resultgrafico,ax=ax)
#modificação do fundo
ax.set_frame_on(False)
#adicionando um título
ax.set_title('Percentual Candidatos Negros',loc='center',pad=30,fontdict={'fontsize':20},color='#3f3f4e')
#retirando o eixo y
ax.get_yaxis().set_visible(False)
#retirnado os ticks do eixo x
ax.tick_params(axis='x',length=0,labelsize=12,colors='black')
#ajustando o título do gráfico
ax.set_xlabel('Anos de Eleição',labelpad=10,fontdict={'fontsize':10},color='black')
#colocando os rótulos
for retangulo in ax.patches:
    ax.text(retangulo.get_x() + retangulo.get_width() / 2,
          retangulo.get_height() + 2,
          '{:.1f}'.format(float(retangulo.get_height())).replace(',','.'),
          ha = 'center',
          fontsize=10,color='black')
#plotando o gráfico
plt.tight_layout();
```
### 5 - Análise de Bens Declarados para Candidatos Religiosos

``` py3
v_primeiroNome = dfCandidatos["NM_URNA_CANDIDATO"].str.split(" ", n = 1, expand = True)

dfCandidatos['PRIMEIRO_NOME']= v_primeiroNome[0] 

dfEvangelicos = dfCandidatos[(dfCandidatos['PRIMEIRO_NOME'].isin(['PASTOR'
                                                                 ,'PASTORA'
                                                                 ,'BISPO'
                                                                 ,'BISPA'
                                                                 ,'MISSIONARIO'
                                                                 ,'MISSIONARIA'
                                                                 ,'IRMAO'
                                                                 ,'IRMA'
                                                                 ,'APOSTOLO'
                                                                 ,'APOSTOLA'])) 
                                                                 & (dfCandidatos['ANO_ELEICAO']=='2022')]

dfEvangelicos.groupby(['PRIMEIRO_NOME','ANO_ELEICAO'],as_index=False) \
                                                .agg(QTD_TOTAL_CANDIDATOS=('SQ_CANDIDATO','count')) \
                                                .sort_values(by=(['PRIMEIRO_NOME','QTD_TOTAL_CANDIDATOS']),ascending=False)

resultFinalBensCandidatosEvangelicos = pd.merge(left=resultBensAgregado
, right=dfEvangelicos
, left_on='SQ_CANDIDATO'
, right_on='SQ_CANDIDATO'
)[['SQ_CANDIDATO','SG_UF','NM_CANDIDATO','PRIMEIRO_NOME','DS_CARGO','VLR_TOTAL_BEM','VLR_TOTAL_BEM_C']]

resultFinalBensCandidatosEvangelicos.sort_values(by='VLR_TOTAL_BEM_C',ascending=False)[['SQ_CANDIDATO'
                                                                                ,'SG_UF'
                                                                                ,'NM_CANDIDATO'
                                                                                ,'PRIMEIRO_NOME'
                                                                                ,'DS_CARGO'
                                                                                ,'VLR_TOTAL_BEM']].head(10)
```
### 6 - Candidatos a Governador que não são Nativos do Estado

``` py3
resultSetCandidatoGovernador = dfCandidatos[(dfCandidatos['ANO_ELEICAO']=='2022') 
             & (dfCandidatos['DS_CARGO']=='GOVERNADOR')][['SQ_CANDIDATO'
                                                          ,'NM_CANDIDATO'
                                                          ,'DS_CARGO'
                                                          ,'SG_UF'
                                                          ,'SG_UF_NASCIMENTO']]

resultSetCandidatoGovernador.loc[resultSetCandidatoGovernador['SG_UF'] == resultSetCandidatoGovernador['SG_UF_NASCIMENTO'], 
                                 'COMPARACAO'] = 'NASCIDO NO ESTADO'
resultSetCandidatoGovernador.loc[resultSetCandidatoGovernador['SG_UF'] != resultSetCandidatoGovernador['SG_UF_NASCIMENTO'], 
                                 'COMPARACAO'] = 'FORASTEIRO'

dfCandidatosGovernadorTotal = dfCandidatos[(dfCandidatos['ANO_ELEICAO']=='2022') & (dfCandidatos['DS_CARGO']=='GOVERNADOR')] \
                                .groupby(['ANO_ELEICAO','DS_CARGO'],as_index=False) \
                                .agg(QTD_CANDIDATO_GOVERNADOR=('SQ_CANDIDATO','count'))

resultSetCandidatoGovernador = resultSetCandidatoGovernador.groupby(['DS_CARGO','COMPARACAO'],as_index=False) \
                                .agg(QTDCANDIDATO=('COMPARACAO','count'))

resultFinalCandidatosGovernador = pd.merge(left=resultSetCandidatoGovernador
                                            , right=dfCandidatosGovernadorTotal
                                            , left_on='DS_CARGO'
                                            , right_on='DS_CARGO'
                                            )[['ANO_ELEICAO'
                                               ,'DS_CARGO'
                                               ,'COMPARACAO'
                                               ,'QTDCANDIDATO'
                                               ,'QTD_CANDIDATO_GOVERNADOR']]

resultFinalCandidatosGovernador['PERCENTUAL'] = (resultFinalCandidatosGovernador['QTDCANDIDATO'] 
                                                 / resultFinalCandidatosGovernador['QTD_CANDIDATO_GOVERNADOR']) * 100


```

### 6.1 - Gráfico - Candidatos a Governador que não são Nativos do Estado

``` py3
#criando a fig e o ax no matplotlib
fig, ax = plt.subplots(figsize=(8,5))
#criando novamente o gráfico
sns.barplot(x='COMPARACAO',y='PERCENTUAL',data=resultFinalCandidatosGovernador,ax=ax)
#modificação do fundo
ax.set_frame_on(False)
#adicionando um título
ax.set_title('Percentual Candidatos Governador Nascidos No Estado',loc='center',pad=30,fontdict={'fontsize':20},color='#3f3f4e')
#retirando o eixo y
ax.get_yaxis().set_visible(False)
#retirnado os ticks do eixo x
ax.tick_params(axis='x',length=0,labelsize=12,colors='black')
#ajustando o título do gráfico
ax.set_xlabel('Candidatos Governador Nascidos No Estado',labelpad=10,fontdict={'fontsize':10},color='black')
#colocando os rótulos
for retangulo in ax.patches:
    ax.text(retangulo.get_x() + retangulo.get_width() / 2,
          retangulo.get_height() + 2,
          '{:.1f}'.format(float(retangulo.get_height())).replace(',','.'),
          ha = 'center',
          fontsize=10,color='black')
#plotando o gráfico
plt.tight_layout();
```

### 7 - Análise de Candidatos que Declararam e Não Declararam Patrimônios

``` py3
resultFinalBensCandidatosDeclarado = pd.merge(left=dfCandidatos
                                            , right=resultBensAgregado
                                            , left_on='SQ_CANDIDATO'
                                            , right_on='SQ_CANDIDATO'
                                            , how='left'
                                            )[['ANO_ELEICAO'
                                               ,'SQ_CANDIDATO'
                                               ,'NM_CANDIDATO'
                                               ,'DS_CARGO'
                                               ,'VLR_TOTAL_BEM'
                                               ,'VLR_TOTAL_BEM_C']]

resultFinalBensCandidatosDeclarado['VLR_TOTAL_BEM_C'].fillna(0,inplace=True)
resultFinalBensCandidatosDeclarado['VLR_TOTAL_BEM_C'].fillna(0,inplace=True)

resultFinalBensCandidatosDeclarado = resultFinalBensCandidatosDeclarado[
                                    resultFinalBensCandidatosDeclarado['ANO_ELEICAO']=='2022']

resultFinalBensCandidatosDeclarado.loc[resultFinalBensCandidatosDeclarado['VLR_TOTAL_BEM_C'].astype(float) == 0
                                    , 'COMPARACAO'] = 'NÃO DECLAROU BEM'

resultFinalBensCandidatosDeclarado.loc[((resultFinalBensCandidatosDeclarado['VLR_TOTAL_BEM_C'].astype(float) >= float(1)) 
                                    & (resultFinalBensCandidatosDeclarado['VLR_TOTAL_BEM_C'].astype(float) < float(1000000)))
                                    ,'COMPARACAO'] = 'RICO'

resultFinalBensCandidatosDeclarado.loc[((resultFinalBensCandidatosDeclarado['VLR_TOTAL_BEM_C'].astype(float) 
                                         >= float(1000000)) 
                                    & (resultFinalBensCandidatosDeclarado['VLR_TOTAL_BEM_C'].astype(float) < float(1000000000)))
                                    , 'COMPARACAO'] = 'MILIONARIO'
resultFinalBensCandidatosDeclarado.loc[((resultFinalBensCandidatosDeclarado['VLR_TOTAL_BEM_C'].astype(float) 
                                         >= float(1000000000))                                       
                                & (resultFinalBensCandidatosDeclarado['VLR_TOTAL_BEM_C'].astype(float) < float(1000000000000)))
                                , 'COMPARACAO'] = 'BILIONARIO'

resultFinalBensCandidatosDeclaradoAgregado = resultFinalBensCandidatosDeclarado.groupby(['COMPARACAO','ANO_ELEICAO']
                                    ,as_index=False).agg(QTD_CANDIDATO=('SQ_CANDIDATO','count'),VLR_TOTAL_BENS=('VLR_TOTAL_BEM_C','sum'))

resultFinalBensCandidatosDeclaradoAgregado = pd.merge(left=resultFinalBensCandidatosDeclaradoAgregado
, right=dfCanidatoCorRacaoTotal
, left_on='ANO_ELEICAO'
, right_on='ANO_ELEICAO'
)[['ANO_ELEICAO','COMPARACAO','QTD_CANDIDATO','VLR_TOTAL_BENS','QTD_CANDIDATO_TOTAL']]

resultFinalBensCandidatosDeclaradoAgregado['PERCENTUAL'] = (resultFinalBensCandidatosDeclaradoAgregado['QTD_CANDIDATO'] 
                                            / resultFinalBensCandidatosDeclaradoAgregado['QTD_CANDIDATO_TOTAL']) * 100
```


### 7.1 - Gráfico Percentual - Análise de Candidatos que Declararam e Não Declararam Patrimônios

``` py3
#criando a fig e o ax no matplotlib
fig, ax = plt.subplots(figsize=(8,5))
#criando novamente o gráfico
sns.barplot(x='COMPARACAO',y='PERCENTUAL',data=resultFinalBensCandidatosDeclaradoAgregado,ax=ax)
#modificação do fundo
ax.set_frame_on(False)
#adicionando um título
ax.set_title('Percentual Candidatos Valores(R$) Vs Candidatos Que Não Declaram Bens',loc='center',pad=30,fontdict={'fontsize':20},color='#3f3f4e')
#retirando o eixo y
ax.get_yaxis().set_visible(False)
#retirnado os ticks do eixo x
ax.tick_params(axis='x',length=0,labelsize=12,colors='black')
#ajustando o título do gráfico
ax.set_xlabel('Categorização de Candidatos',labelpad=10,fontdict={'fontsize':10},color='black')
#colocando os rótulos
for retangulo in ax.patches:
    ax.text(retangulo.get_x() + retangulo.get_width() / 2,
          retangulo.get_height() + 2,
          '{:.1f}'.format(float(retangulo.get_height())).replace(',','.'),
          ha = 'center',
          fontsize=10,color='black')
#plotando o gráfico
plt.tight_layout();
```


### 7.2 - Gráfico Aberto Por Valor - Análise de Candidatos que Declararam e Não Declararam Patrimônios

``` py3
#criando a fig e o ax no matplotlib
fig, ax = plt.subplots(figsize=(8,5))
#criando novamente o gráfico
sns.barplot(x='COMPARACAO',y='VLR_TOTAL_BENS',data=resultFinalBensCandidatosDeclaradoAgregado,ax=ax)
#modificação do fundo
ax.set_frame_on(False)
#adicionando um título
ax.set_title('Percentual Candidatos Valores(R$) Vs Candidatos Que Não Declaram Bens',loc='center',pad=30,fontdict={'fontsize':20},color='#3f3f4e')
#retirando o eixo y
ax.get_yaxis().set_visible(False)
#retirnado os ticks do eixo x
ax.tick_params(axis='x',length=0,labelsize=12,colors='black')
#ajustando o título do gráfico
ax.set_xlabel('Categorização de Candidatos',labelpad=10,fontdict={'fontsize':10},color='black')
#colocando os rótulos
for retangulo in ax.patches:
    ax.text(retangulo.get_x() + retangulo.get_width() / 2,
          retangulo.get_height() + 2,
          '{:.1f}'.format(float(retangulo.get_height())).replace(',','.'),
          ha = 'center',
          fontsize=10,color='black')
#plotando o gráfico
plt.tight_layout();
```