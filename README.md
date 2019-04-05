# Solução simples de ingestão de grande volumes de dados no Google Cloud
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


#### 1. Motivação Pessoal para Arquitetura

##### 1.1 Porque desenhar toda solução na núvem e não localmente ?
Como meu computador não é muito, experimentar qualquer solução localmente seria lastimável. 
##### 1.2 Porque utilizando o Google Cloud ?
Escolhi o google por já ter bastante contato toda solução Cloud do [Firebase](https://firebase.google.com) para um projeto pessoal com IONIC.
##### 1.2 Porque utilizar Python e não Java ?
Porque Java eu já tenho bastante experiência e Python para esse tipo de projeto eu teria um código mais limpo.

de tratame para ingestão de dados utilizando o [Google Fataflow](https://cloud.google.com/dataflow/).
CSV > Google Storage > Google DataFlow > Google BigQuery
