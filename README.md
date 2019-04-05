# Solução simples de importação e geração indicadores de grande volumes de dados no Google Cloud
### Repositório criado com objetivo de compartilhar solução desde as motivações da arquitetura até a implementação. 
#### Motivação: Cliente possui site que a cada 1 minuto gera um arquivo de log no formato CSV de 400 MB. 
Esse arquivo possui todo ciclo de navegação de cada cliente desde o acesso inicial ao site até o pagamento. 
#### Desafio: Desenhar solução escalável que leia esses arquivos e que se consiga extrair informações relevantes para o clientes, como: 
KPIs importantes como taxa de conversão, taxa de abandono de carrinho de compras entre outros. 


### O escopo desse documento se divide nos seguintes tópicos.

1. Arquitetura Proposta
2. Motivação da Arquitetura
3. Motivação Pessoal para Arquitetura
4. Implementação
5. Proposta de Evolução


### 1. Arquitetura Proposta

![alt text](https://raw.githubusercontent.com/samueljb/my-tests2/master/csv_file_to_bigquery_pt.png "Imagem Arquitetura")

> Basicamente o Google DataFlow é o orquestrador, job em Python, que pega os arquivos CSVs adicionados no bucket do Google Storage, transforma no formato reconhecido e adiciona na tabela do BigQuery. 

> Com os dados na tabela do BigQuery podemos criar transformações utilizando comandos SQLs, criando views e nossos KPIs(indicadores) que serão compartilhadas no google DataStudio para visualização dos Gestores da empresa.

#### 1.1 O Google Dataflow 
##### Será o orquestrador de toda execução, será criado um job em python que fará a captura do arquivo bucket do Google Storage, traformar o mesmo e inserir na Tabela do BigQuery responsável pela: 
Gestão da pipeline dos dados, sendo possivel executar poderosas tranformações dos dados, com poucas linhas de códigos.
Gerenciamento de Recursos, instanciando CPUs, conforme a necessidade, conforme a google diz "Recursos praticamente ilimitados"


#### 1. Arquitetura Proposta


#### 1. Motivação Pessoal para Arquitetura

##### 1.1 Porque desenhar toda solução na núvem e não localmente ?
Como meu computador não é muito, experimentar qualquer solução localmente seria lastimável. 
##### 1.2 Porque utilizando o Google Cloud ?
Escolhi o google por já ter bastante contato toda solução Cloud do [Firebase](https://firebase.google.com) para um projeto pessoal com IONIC. Também porque eu ganhei um voucher por 6 meses para utilizar todos o serviços do Google Cloud gratuitamente.
##### 1.2 Porque utilizar Python e não Java ?
Porque Java eu já tenho bastante experiência e Python para esse tipo de projeto eu teria um código mais limpo.

de tratame para ingestão de dados utilizando o [Google Fataflow](https://cloud.google.com/dataflow/).
CSV > Google Storage > Google DataFlow > Google BigQuery
