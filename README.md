# Solução simples de importação e geração indicadores de grande volumes de dados no Google Cloud
### Repositório criado com objetivo de compartilhar solução desde as motivações da arquitetura até a implementação. 
#### Motivação: Cliente possui site que a cada 1 minuto gera um arquivo de log no formato CSV de 400 MB. 
Esse arquivo possui todo ciclo de navegação de cada cliente desde o acesso inicial ao site até o pagamento. 
#### Desafio: Desenhar solução escalável que leia esses arquivos e que se consiga extrair informações relevantes para o clientes, como: 
KPIs importantes como taxa de conversão, taxa de abandono de carrinho de compras entre outros. 


### O escopo desse documento se divide nos seguintes tópicos.

1. Arquitetura Proposta
2. Motivação da Arquitetura
3. Fluxo Principal
4. Motivação Pessoal para Arquitetura
5. Implementação
6. Proposta de Evolução


### 1. Arquitetura Proposta

![alt text](https://raw.githubusercontent.com/samueljb/my-tests2/master/csv_file_to_bigquery_pt.png "Imagem Arquitetura")

#### Abaixo uma breve descrição de cada step e seu devido marketing.

+ [Google DataFlow](https://cloud.google.com/dataflow/?hl=pt-br), nosso orquestrador responsável pela gestão da pipeline dos dados.
> Processamento simplificado de dados de stream e em lote, com a mesma confiabilidade e expressividade
+ [Google Storage](https://cloud.google.com/storage/?hl=pt-Br), nosso storage aonde colocaremos nossos arquivos CSV.
> Armazenamento unificado de objetos para desenvolvedores e empresas
+ [BigQuery](https://cloud.google.com/bigquery/?hl=pt-br), nosso banco de dados.
> Um armazenamento de dados na nuvem rápido, altamente dimensionável, econômico e totalmente gerenciado para análise com machine learning.
+ [DataStudio](https://datastudio.google.com/), nossa ferramenta para criação de relatórios. 
> Desbloqueie o poder de seus dados com painéis interativos e relatórios bonitos que inspiram decisões de negócios mais inteligentes.

### 2. Motivação da Proposta
Para essa essa arquitetura priorizamos não obrigar instalar nenhuma biblioteca ou ferramenta localmente, tudo será instalado na núvem.  
Cobre requisitos do cliente e outros como escalabilidade sob demanda, modelo de programação simplificado e opensource, controle de custo, gerenciamento automático de recursos, que conforme a google diz:
> "Recursos praticamente ilimitados".

### 3. Fluxo Principal

> Basicamente o [Google DataFlow](https://cloud.google.com/dataflow/?hl=pt-br) é o orquestrador, nesse exemplo é um job em Python, que pega os arquivos CSVs adicionados no bucket do [Google Storage](https://cloud.google.com/storage/?hl=pt-Br), transforma no formato reconhecido e adiciona na tabela do [BigQuery](https://cloud.google.com/bigquery/?hl=pt-br). 

> Com os dados na tabela do [BigQuery](https://cloud.google.com/bigquery/?hl=pt-br) podemos criar transformações utilizando comandos SQLs, criando views e nossos KPIs(indicadores) que serão compartilhadas no google [DataStudio](https://datastudio.google.com/) com uma estética de Relatório e Dashboards para visualização dos Gestores da empresa.

### 4. Motivação Pessoal para Arquitetura

##### 3.1 Porque desenhar toda solução na núvem e não localmente ?
Como meu computador não é muito, experimentar qualquer solução localmente seria lastimável. 
##### 3.2 Porque utilizando o Google Cloud ?
Escolhi o google por já ter bastante contato toda solução Cloud do [Firebase](https://firebase.google.com) para um projeto pessoal com IONIC. Também porque eu ganhei um voucher por 6 meses para utilizar todos o serviços do Google Cloud gratuitamente.
##### 3.3 Porque utilizar Python e não Java ?
Porque Java eu já tenho bastante experiência e Python para esse tipo de projeto eu teria um código mais limpo.
##### 3.4 Porque utilizar Google DataStudio ?
Utilizei aqui para simplificar o nosso exemplo, o customização dos relatórios é muito limitada, indicaria soluções como Tableau, SAP BO, SAS entre outros, dependeria do orçamento.

de tratame para ingestão de dados utilizando o [Google Fataflow](https://cloud.google.com/dataflow/).
CSV > Google Storage > Google DataFlow > Google BigQuery


### 5. Implementação

#### 5.1 Dataflow

#### Para implementação após todo setup do google listados no ponto 5.1, temos os seguintes arquivos:

##### Exemplo com csv que deve ser importado no bucket do seu google storage:

> [dados_navigacionais_100.csv](dados_navigacionais_100.csv)

#####  Arquivo python com a implementação do JOB para rodar no Google Dataflow:

> [storage-to-dataflow-to-bigquery.py](storage-to-dataflow-to-bigquery.py)

Com os seguintes pontos importantes, para customização do script:

Linha 121: A separação dos campos no CSV e os tipos de cada um, para criação da tabela do banco.
```python
schema='load_timestamp:STRING,ip:STRING,visit_id:STRING,device_type:STRING,url_location:STRING,page_type:STRING,search_query:STRING,product_id:STRING,site_department_id:STRING,product_unit_price:STRING,freight_delivery_time:STRING,freight_value:STRING,cart_qty:STRING,cart_total_value:STRING',
```
Os tipos são os mesmos do SCHEMA das tabelas do BigQuery.

Linha 38: Pega as linhas do CSV e transforma no formato entendível pelo BigQuery, informando a ordem dos campos no CSV.
```python
    row = dict( zip(('load_timestamp', 'ip', 'visit_id', 'device_type', 'url_location', 'page_type', 'search_query', 'product_id', 'site_department_id', 'product_unit_price', 'freight_delivery_time', 'freight_value', 'cart_qty', 'cart_total_value'),
                values))
```

Linha 104:  CREATE_IF_NEEDED cria a tabela caso não exista.
Linha 106:  WRITE_TRUNCATE apaga a tabela caso exista e insira, esta assim a título de exemplo mas deveria ser: WRITE_APPEND
```python
            create_disposition=beam.io.BigQueryDisposition.CREATE_IF_NEEDED,
            # Deletes all data in the BigQuery table before writing.
            write_disposition=beam.io.BigQueryDisposition.WRITE_TRUNCATE)))
```
Aqui você pode ver mais detalhes desse parâmetros:
https://beam.apache.org/documentation/io/built-in/google-bigquery/


#### 5.2 BigQuery

Foia criada seguinte view b2w-americanas-teste:dataNavigationDataSet.SalesKPI, com os indicadores mais relevantes:
```sql
    SELECT  'Abandono de Carrinho de compras' as Description, basket.losing as q1, thankyou.buy as q2, 100-ROUND((thankyou.buy / basket.losing)*100, 2) as rate  FROM
(SELECT count(distinct visit_id) as losing FROM `b2w-americanas-teste.dataNavigationDataSet.RAW_DATA_NAVIGATION` 
where page_type = 'basket') basket
CROSS JOIN
(SELECT count(distinct visit_id) as buy FROM `b2w-americanas-teste.dataNavigationDataSet.RAW_DATA_NAVIGATION` 
where page_type = 'thankyou') thankyou
union all
SELECT  'Taxa de conversão vindo da home', home.visit, thankyou.buy, ROUND((thankyou.buy / home.visit)*100, 2) as conversion_rate_home  FROM
(SELECT count(distinct visit_id) as visit FROM `b2w-americanas-teste.dataNavigationDataSet.RAW_DATA_NAVIGATION` 
where page_type = 'home') home
CROSS JOIN
(SELECT count(distinct visit_id) as buy FROM `b2w-americanas-teste.dataNavigationDataSet.RAW_DATA_NAVIGATION` 
where page_type = 'thankyou') thankyou
union all
SELECT  'Taxa de conversão vindo da Busca', home.visit, thankyou.buy, ROUND((thankyou.buy / home.visit)*100, 2) as conversion_rate_home  FROM
(SELECT count(distinct visit_id) as visit FROM `b2w-americanas-teste.dataNavigationDataSet.RAW_DATA_NAVIGATION` 
where page_type = 'search') home
CROSS JOIN
(SELECT count(distinct visit_id) as buy FROM `b2w-americanas-teste.dataNavigationDataSet.RAW_DATA_NAVIGATION` 
where page_type = 'thankyou') thankyou
union all
SELECT  'Taxa de conversão vindo do Produto', home.visit, thankyou.buy, ROUND((thankyou.buy / home.visit)*100, 2) as conversion_rate_home  FROM
(SELECT count(distinct visit_id) as visit FROM `b2w-americanas-teste.dataNavigationDataSet.RAW_DATA_NAVIGATION` 
where page_type = 'product') home
CROSS JOIN
(SELECT count(distinct visit_id) as buy FROM `b2w-americanas-teste.dataNavigationDataSet.RAW_DATA_NAVIGATION` 
where page_type = 'thankyou') thankyou
union all
SELECT  'Não pagamento - Desistencia', home.visit, thankyou.buy, 100-ROUND((thankyou.buy / home.visit)*100, 2) as conversion_rate_home  FROM
(SELECT count(distinct visit_id) as visit FROM `b2w-americanas-teste.dataNavigationDataSet.RAW_DATA_NAVIGATION` 
where page_type = 'payment') home
CROSS JOIN
(SELECT count(distinct visit_id) as buy FROM `b2w-americanas-teste.dataNavigationDataSet.RAW_DATA_NAVIGATION` 
where page_type = 'thankyou') thankyou


```

### 6 Execução

### 6.1 Pre-Requisitos

+ 6.1.1 Criar conta no [Google Cloud](https://console.cloud.google.com/?_ga=2.266585353.-640181581.1548091189), a partir desse console você vai poder criar o seu projeto.
+ 6.1.2 Criar [o seu projeto no seu console](https://console.cloud.google.com/cloud-resource-manager?_ga=2.260162954.-640181581.1548091189), esse ponto é importante pois todos os itens deve ser criados dentro desse projeto.
+ 6.1.3 Ativar no projeto e criar [um bucket no Google Storage](https://console.cloud.google.com/storage/browser?_ga=2.25283386.-640181581.1548091189) e adicionar o CSV, isso tudo pode ser feito visualmente.
+ 6.1.4 Ativar no projeto criar apenas o [dataset no Google BigQuery](https://bigquery.cloud.google.com/?hl=pt-br) , pois em nosso script a tabela é criada caso não exista.
+ 6.1.5 Acessar [o terminal remoto do seu projeto](https://cloud.google.com/shell/)
+ 6.1.6 Acessar [ativar o Google Dataflow](https://console.cloud.google.com/dataflow?_ga=2.22120508.-640181581.1548091189) para monitorar a execução da pipeline.

### 6.2 Execução

##### Fazer upload do [storage-to-dataflow-to-bigquery.py](storage-to-dataflow-to-bigquery.py) via interface no terminal remoto.

##### Acessar a pasta do arquivo e executar o comando abaixo no terminal, substituindo as seguintes variáveis, por:
> NOME_SEU_DATA_PROJETO, nome do projeto criado no ponto 6.1.2
> NOME_SEU_BUCKET, nome do bucket criado no ponto 6.1.3.
> NOME_SEU_DATA_SET, nome do dataset criado no ponto 6.1.4.

```terminal
python storage-to-dataflow-to-bigquery.py --input gs://NOME_SEU_BUCKET/dados_navegacionais* --output NOME_SEU_DATA_PROJETO:NOME_SEU_DATA_SET.RAW_DATA_NAVIGATION --runner DataflowRunner --project NOME_SEU_DATA_PROJETO --job_name job-name-001 --temp_location gs://NOME_SEU_BUCKET/tmp/
```
