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

### 5.1 Pre-Requisitos

+ Criar conta no [Google Cloud](https://console.cloud.google.com/?_ga=2.266585353.-640181581.1548091189), a partir desse console você vai poder criar o seu projeto.
+ Criar [o seu projeto no seu console](https://console.cloud.google.com/cloud-resource-manager?_ga=2.260162954.-640181581.1548091189), esse ponto é importante pois todos os itens deve ser criados dentro desse projeto.
+ Criar [um bucket no Google Storage](https://console.cloud.google.com/storage/browser?_ga=2.25283386.-640181581.1548091189) e adicionar o CSV, isso tudo pode ser feito visualmente.
+ Criar apenas o [dataset no Google BigQuery](https://bigquery.cloud.google.com/?hl=pt-br) , pois em nosso script a tabela é criada caso não exista.
+ Acessar [o terminal remoto do seu projeto](https://cloud.google.com/shell/)

