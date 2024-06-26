# Projeto de Engenharia de dados - Estimativa de custos AWS - WindFarm

## Introdução

Ainda sobre o projeto WindFarm do professor Fernando Amaral, surgiram no meu último post duas indagações que me inspiraram a escrever sobre e dar um passo além do projeto sugerido. Nesse ponto, preciso afirmar que tive dificuldades em pensar em como poderia dar passos além do que foi proposto. Porém, o intuito de se escrever por aqui é sempre receber essas perguntas e utilizá-las para o aprendizado.

A primeira questão foi sobre os custos da AWS. Em um projeto de engenharia de dados, essa é uma etapa crucial, é o que definirá se o projeto vai ou não para frente. Muitos dos projetos falham nessa etapa essencialmente por não se ter em mente o valor que será gasto no projeto. Tendo isso em vista, vamos criar um ambiente fictício baseado no projeto já descrito. Utilizando os dados, a quantidade de dados produzidos, as ferramentas AWS e a calculadora da AWS para nos ajudar nessa estimativa.

O segundo ponto é a definição de Kinesis Firehose e Kinesis Stream. Quais as atribuições dessas ferramentas e, no projeto, o que impactaria se pudéssemos fazer suas substituições.

Acredito que, em alguns aspectos, os cálculos que utilizarei não serão muito precisos, então peço àqueles que tiverem mais experiência, se puderem contribuir para melhorar ainda mais esse conteúdo, sintam-se à vontade.

## Descrição do ambiente utilizado

Uma turbina eólica, de maneira bem grotesca, é um estrutura que transformam a energia cinética do vento em energia elétrica. Então o problema de negócio é o seguinte. Precisamos coletar dados de uma turbina específica diáriamente, com o intuíto de observar em tempo real e analisar o desempenho ao longo do tempo. Para este caso considerarei o período de um ano e apenas dados como temperatura, pressão e potencia.

As ferramentas utilizadas para o projeto dessa pipeline foram:
- AWS Lambda
- Um bucket S3 para armazenar os dados em streaming
- AWS Kinesis Stream
- AWS Kinesis Firehouse
- AWS Crawler
- AWS Glue
- AWS bucket S3 Para armazenar os dados processados
- AWS Athena

Os arquivos geradores fornecem os dados na seguintes estruturas e tamanhos:

- Temperatura

    - Id (int) - 4bts
    - Data (date) - 8bts
    - Type (int) - 4bts
    - Timestamp (date) - 8bts

- Pressão hidráulica

    - Id(int) - 4bts
    - Data(date) - 8bts
    - Type(int) - 4bts
    - Timestamp(date) - 8bts

- Potencia

    - Id(int) - 4bts
    - Data - (date) - 8bts
    - Type - (int) - 4bts
    - Timestamp - (date) - 8bts

Cada arquivo produtor gera um total de 24bts, como são 3 produtores temos 72bts. Para este caso, os dados serão produzidos a cada segundo então temos um total de 3 registros por segundo gerando 72bts por segundo. O que por hora será 253,125Kbt. 

Os dados serão produzidos 24hs por dia todos os dias e alimentará um endpoint para visualizar as informações geradas. A partir disso temos que serão geradas um mês temos 178mb e 7,776 milhões de registros. No momento iremos considerar que esses dados são armazenados no S3 por cerca de um ano. 

Os custos serão estimados para cada ferramenta por mês, no final faremos uma estimativa por milhão de registro, que no caso quando precisarmos converter para tempo temos para produzir um milhão de registros o é aproximadamente 3,86 dias.

## Estimativa de custos para cada ferramenta
### AWS Lambda

Para o Lambda utilizamos os seguintes parâmtros:

- Architecture: x86
- Number of requests: 3
- Duration of each request (in ms): 100

Amount of memory allocated
- Value: 128mb

Amount of ephemeral storage allocated
- Value: 512mb

Considerando o free tier temos um consumo mensal de 1,38USD

### Bucket S3

Considerando as configurações:

- S3 Standard storage: 0,173828125GB por mês

Considerando que serão inseridos novos dados a cada segundo temos então 7.776.000 de registros por mês

Neste caso por mês teremos o valor mensal de 38.88USD


### AWS Kinesis Stream
Temos dois tipos de modos

Utilizando o Provisioned mode

Com o modo de provisionamento, você especifica o numero de shards necessários para sua aplicação baseado na taxa de escrita e leitura. Um shard é uma unidade de capacidade de 1MB/s de escrita e 2MB/s de leitura.

Utilizando o on-Demand mod mode

Com modo sob demanda, você paga por quantidade de GB de dados escritos e lidos pelo seu data stream. Você não precisaespecificar o quanto de leitura e escrita é esperado pela sua aplicação para performar. O Kinesis data Streams define isso automaticamente baseado na suas carga de trabalho seja para cima ou para baixo. Esse modo tem a desvantagem de não se ter uma precisão de valores gastos.

para o nosso caso utilizaremos o modo provisionado, pois se trata de uma aplicação simples e nela temos o controle do que é produzido.

Considerando as seguintes configurações:

- Baseline number of records: 3 por segundo
- Peak number of records: 3 por segundo
- Average record size: 0,0703125KB
- Buffer for growth and to absorb un-expected peaks: 20%
- Number of Consumer Applications: 1
- Duration of data retention: 1 dia
- Number of Enhanced fan-out consumers: 1

O total estimado de consumo por mês é de  43.81 USD


### AWS Kinesis Firehouse

O kinesis Firehouse cobra pelo volume de dados ingetados no Kinesis Data Firehouse. Utilizando outros serviços  associadas, estes serão cobrados separadamente.

- Source Type: Direct PUT or Data Stream
- Data records units: exact number
- Number of records for data ingestion: 3 por segundo
- Record size: 0,0703125KB

O total estimado de consumo por mês é de  1.09 USD

### AWS Glue

Os serviços estimados serão 

AWS Glue ETL jobs and interactive sessions
AWS Glue Crawlers

Apache Spark ETL jobs

- Number of DPUs for Apache Spark job: 10DPUs
- Number of DPUs for Apache Spark job: 1 hora

Python Shell ETL jobs

- Number of DPUs for Python Shell job: 1DPU
- Duration for which Python Shell job ETL runs: 1 hora

Interactive sessions

- Number of DPUs for each provisioned interactive session: 2
- Duration for provisioned interactive sessions: 1 hora

Crawlers

- Number of crawlers: 3
- Duration for each crawler: 1 minuto


O total estimado de consumo por mês é de  5.94 USD

### AWS bucket S3 Para armazenar os dados processados

Como a quantidade de registro serão as mesmas que as produzidas o valor de armazenamento é o mesmo do que o orçado anteriormente. Neste caso por mês teremos o valor mensal de 38.88USD

### AWS Athena

Para o negócio, pensando que temos um dataviz que é atualizado de hora em hora. Então podemos utilizar os seguintes parâmetros:


SQL queries with per query pricing
- Total number of queries: 8 querys por dia
- Amount of data scanned per query: 0.178GB

SQL queries with capacity based pricing
- Number of DPUs: 24
- Length of time (hours) capacity is active: 1 Hora por mês

O total estimado de consumo por mês é de  7.41 USD

## Resumo:

O total de valor mensal estimado para cada serviço ficou em:

- AWS Lambda: 1,38USD
- Um bucket S3 para armazenar os dados em streaming: 38.88USD
- AWS Kinesis Stream: 43.81 USD
- AWS Kinesis Firehouse: 1.09 USD
- AWS Glue: 5.94 USD
- AWS bucket S3 Para armazenar os dados processados: 38.88USD
- AWS Athena: 7.41 USD

Dando um total de 137,39 USD o que representa 17,67 USD para cada milhão de registros. Claro que podem haver cobranças que não foram inseridas neste cálculo, creio que entre 10 a 20% do valor seria uma estimativa ideal para evitar possíveis cobranças inesperadas.


## Conclusão

O presente artigo, tenta apresentar os custos baseados em um problema em que as ferramentas apresentadas já foram desenhados. Não significa que para este problema as ferramentas utilizadas sejam as mais adequadas. Para este caso, se estivéssimos falando de um objeto real de telemetria, existem outros tipos de ferramentas mais eficazes e talvez até mais baratos. Uma das sugestões que recebi seria utilizar o DBT para o processamento e as consultas em um banco de dados como aurora ou PostgreSQL.

Outro ponto importante, é que no processo é considerado apenas um processo de interação. Na vida real certamente teríamos diversos outros consumidores. Eu gostaria de estimar mais especificamente esta parte, mas nesse caso achei complexo dizer que temos apenas tantos consumidores sem um embasamento real. Porém a partir de um consumidor podemos estrapolar para as demais quantidades.

Certamente há outros pontos importantes deixados de fora, mas para este projeto acredito que os essenciais foram atendidos. Caso tenham alguma dúvida ou sugestão me procurem.

