﻿# Projeto de Engenharia de dados - Criando um Datalake - WindFarm

## Introdução

 O presente artigo é referente ao segundo projeto disponívei no curso do [Fernando Amaral](https://www.linkedin.com/in/fernando-amaral/) no curso [Formação Engenharia de Dados: Domine Big Data!](https://www.udemy.com/course/engenheiro-de-dados/learn/lecture/15289778?start=0) disponível na Udemy.

O que é um data lake?

O Data lake é uma estrutura de armazenamento de big data que tem como principal diferença dos armazenamentos tradicionais a sua possibilidade de se utilizar estruturas de dados diversos. Ele não é uma substituição dos modelos de datawrehouse, porém a sua aplicação pela quantidade de dados e da diferença de estruturas a sua aplicação tem ficado cada vez mais em evidência.

 Este projeto consiste na criação de aplicativos geradores de dados de temperatura, pressão e potencia em python simulando uma produção de energia eólica. Os dados serão consumidos pelo sistema de streaming da AWS o Kinesis, em seguida para o kinesis Firehouse. O Firehouse envia os dados para um bucket S3 e a partir dai utilizamos o Crwaler para catalogarmos os dados e em seguida enviado para o Glue, que é uma ferramenta de ETL em blocos. No final os dados são direcionados para um outro bucket S3 e no fim realizamos as consultas no Athena.

 O principal objetivo aqui é explorar as ferramentas descritas ao longo do curso. Você verá que o processo é bem simples, mas é a partir do domínio dos princicos das ferramentas é que podemos construir estruturas maiores.

 
![image](https://github.com/Jezandre/north_wind_db/assets/63671761/1f4e60e5-6822-400e-9c79-17656339e7cf)

 ## Materiais

 Para este processo utilizamos uma grande gama de ferramentas vamos descrever quais abaixo:

 - IDE: Utilizamos três notebooks no google colab para gerarmos os dados utilizando python
 - Biblioteca: Utilizamos a biblioteca Boto3 para que os dados gerados pudessem se conectar ao bucket S3 da AWS
 - Amazon AWS Kinesis Data Stremins: Serviço da AWS de stream de dados
 - Amazon AWS Kinesis Data Firehouse: Serviço da AWS de armazem de dados
 - Amazon S3: Serviço da amazon de armazenamentos de dados
 - Amazon Glue Crawler: Serviço da Amazon para catalogar dados
 - Amazon Glue Job: Serço de ETL da Microsoft
 - Amazon Athena: Serço para realizar consultas SQL
 - Amazon IAM: serviço de autenticação da Amazon


## Chave de conexão


O código utiliza o client da AWS através do boto3 para realizar a autenticação do usuário e gerar os dados que serão armazenados no bucket.

Os dados de acesso você pode criar seguindo o passo a passo a seguir:

1. Faça login na Console da AWS: Acesse o Console de Gerenciamento da AWS, usando suas credenciais de conta da AWS.
2. Acesse a página "Minhas Credenciais de Segurança": No canto superior direito do Console de Gerenciamento da AWS, clique no seu nome de usuário e , em seguida, selecione "My Security Credentials" (Minhas Credenciais de Segurança) no menu suspenso.
3. S elecione "Credenciais de acesso (chave de acesso e chave secreta)": Na seção "Acesso e segurança", clique em "Credenciais de acesso (chave de acesso e chave secreta)".
4. Crie ou visualize suas chaves de acesso: Se você já tem chaves de acesso, elas serão listadas nesta página. Se não, clique em "Criar nova chave de acesso" para gerar novas credenciais. Quando você cria uma nova chave de acesso, a AWS exibe a Access Key ID e a Secret Access Key. É importante salvar essas informações em um local seguro, pois a Secret Access Key só é mostrada uma vez.
5. Revise suas permissões: Certifique-se de que as permissões associadas a essas chaves de acesso sejam apropriadas para as tarefas que você pretende realizar. Você pode ajustar essas permissões através do IAM (Identity and Access Management) da AWS, se necessário.

# Criando um aplicativo de stream na AWS

O primeiro passo é criar um aplicativo de Stream utilizano o Kinesis. Para isso basta localizar o serviço Kinesis, selecionar stream de dados no menu lateral e criar o fluxo de dados com o nome que voce quiser. As configurções básicas foram mantidas os padrões.


## Criando arquivos geradores

Os arquivos geradores são as estruturas de códigos desenvolvidas em python, que utilizam de valores randomicos e armazenam dentro de um arquivo json no bucket S3. Então para isso é preciso criar um bucket que irá receber estes arquivos.

O primeiro passo em cada notebook precisa ser intalado a biblioteca boto3, se não você não conseguirá conectar no seu serviço AWS.

```python
!pip install boto3
```

### Gerador dados de temperatura

```python
import boto3
import json
import time
from random import uniform
from datetime import datetime

cliente = boto3.client(
    'kinesis',
    aws_access_key_id='<<id_do_usuario>>',
    aws_secret_access_key='<<chave>>>',
    region_name='<<Regiao>>'
)

id = 0
while True:
  dados = uniform(20,25)
  id += 1;
  registro = {
      'idtemp': str(id),
      'data': str(dados),
      'type': 'temperature',
      'timestamp': str(datetime.now())
      }

  cliente.put_record(
      StreamName='nome_Stream',
      Data = json.dumps(registro),
      PartitionKey='02'
      )

  time.sleep(10)
```

Importações: Importa as bibliotecas necessárias, incluindo boto3 para interagir com os serviços da AWS, json para manipulação de dados JSON, time para manipulação de tempo e datetime para obtenção de carimbos de data/hora.

Configuração do cliente: Utiliza o boto3 para configurar um cliente para interagir com o serviço Amazon Kinesis. São fornecidas as credenciais de acesso (aws_access_key_id e aws_secret_access_key) e a região (region_name) onde o stream do Amazon Kinesis está localizado.

Loop de envio de dados: O script entra em um loop infinito que envia dados simulados de temperatura para o stream do Amazon Kinesis. Em cada iteração do loop: Gera um valor de temperatura, aleatório entre 20 e 25.

Incrementa o ID do registro.

Cria um registro JSON com os campos 'idtemp' (ID da temperatura), 'data' (valor da temperatura), 'type' (tipo de dados) e 'timestamp' (carimbo de data/hora atual).

Utiliza o método put_record do cliente do Amazon Kinesis para enviar o registro para o stream especificado (windfarm). O registro é convertido em uma string JSON antes de ser enviado.

Aguarda 10 segundos antes da próxima iteração do loop.

O processo se repete para os outros dois arquivos

### Gerador dados de pressão hidráulica

```python
import boto3
import json
import time
from random import uniform
from datetime import datetime

cliente = boto3.client(
    'kinesis',
    aws_access_key_id='<<id_do_usuario>>',
    aws_secret_access_key='<<chave>>>',
    region_name='<<Regiao>>'
)

id = 0
while True:
  dados = uniform(70,80)
  id += 1;
  registro = {
      'idtemp': str(id),
      'data': str(dados),
      'type': 'hidraulic_pressure',
      'timestamp': str(datetime.now())
      }

  cliente.put_record(
      StreamName='nome_Stream',
      Data = json.dumps(registro),
      PartitionKey='02'
      )

  time.sleep(10)
  ```

### Gerador dados de potencia
```python
import boto3
import json
import time
from random import uniform
from datetime import datetime

cliente = boto3.client(
    'kinesis',
    aws_access_key_id='<<id_do_usuario>>',
    aws_secret_access_key='<<chave>>>',
    region_name='<<Regiao>>'
)

id = 0
while True:
  dados = uniform(0.7,1)
  id += 1;
  registro = {
      'idtemp': str(id),
      'data': str(dados),
      'type': 'powerfactor',
      'timestamp': str(datetime.now())
      }

  cliente.put_record(
      StreamName='<<nome_Stream>>',
      Data = json.dumps(registro),
      PartitionKey='02'
      )
  print(registro)

  time.sleep(10)
  ```


## Criar fluxo entrega

O próximo passo é armazenar os dados, para isso precisaremos criar um fluxo de entrega, que consiste em pegar os dados produzidos pelo Kinesis stream e encaminhar para um bucket s3. E para isso utilizaremos o Kinesis Firehouse.

No menu lateral localize a opção cluxo de entrega. Em seguida crie um fluxo de entrega e identifiquea origem e a saída. A origem obviamente é o Amazon Data Stream que foi criado anteriormente. E o destino é o bucket S3.

Em configurações de origem você localiza o data stream e o bucket de entrega.

Alteramos também as configurações de 300 para 60 segundos.

Ao fim dessas configurações, a aplicação ficou produzindo dados por durante certa de meia hora, para que possamos ter uma quantidade interssante de dados. Lembrando que a instancia do Kinesis precisa estar ativa para que os arquivos sejam gerados.


## Criar role para o Glue

O primeiro passo para utilizar o Glue nesse exercício, é atribuir uma função para o serviço Glue de administrador. 

1. Ir em IAM
2. Clicar em função
3. Selecionar a opção Serviço AWS
4. Pesquisar em Casos de uso para outros serviços da AWS a opção Glue
5. clicar em próximo
6. Selecionar a opção administrator
7. Dar um nome para a função
8. Clicar em criar a função

Após a criação da função, nós iremos novamente para o Glue e criamos um banco de dados. Este banco de dados terá as tabelas criadas pela ferramenta Crawler, que é uma ferramenta que faz a catalogação dos dados. Basicamente ela identifica o tipo e cria uma tabela dentro de um banco de dados RDS.

A configuração do crawler é bem simples, basta identificar as fontes e dar um nome para o crawler. No caso como será a primeira vez que etamos realizando esse processo lembre-se de selecionar a opção que os dados ainda não foram identificados. Este banco de dados irá receber as informações geradas que estão no bucket S3 de entrega.

Avançando, Em IAM role você seleciona a função que foi criada para realizar a autenticação, depois voce seleciona a base de dados que você acabou de criar.

## Criar job no Glue

O próximo passo é criar a automação dentro do Glue para que os dados sejam salvos dentro do datalake em um bucket S3. E para isso utilizaremos o glue. Iremos no glu localizar a opção Jobs e criar um novo.

O processo é bem simples iremos utilizar apenas 2 steps. Então primeiro definiremos a fonte selecionando source, nele selecionamos a opção de aws glue data catalog e em data source properties selecionamos o noss bucket onde os dados gerados estão sendo armazenados.

Com o primeiro step selecionado selecionaremos o target que também será um bucket s3. O formato a ser salvo é o parquet e a localização será dentro do nosso bucket s3. Esse bucket servirá como um datalake para realizarmos consulstas utilizando o Athena. 

Por fim rodamos e executamos o job.

## Realizar consultadas utilizando o Athena

Por fim os dados podem ser cunstultas utilizando o AThena, basta procurar o serviço dentro da AWS selecionar a base de dados e fazer as consultas.


## Conclusão

Para concluir, a criação de um Data Lake para este projeto, podemos destacar a importância de compreender e dominar as ferramentas de engenharia de dados disponíveis na nuvem, como as fornecidas pela AWS. Ao longo do projeto, exploramos desde a geração simulada de dados até a sua ingestão, transformação e armazenamento em um ambiente de Data Lake, utilizando serviços como Kinesis, Firehose, S3, Glue e Athena.

Essas ferramentas permitem não apenas a criação de um pipeline para lidar com grandes volumes de dados em tempo real, mas também a preparação desses dados para análises posteriores. O Data Lake se torna não apenas um repositório de dados, mas um centro de informações para tomada de decisões e análises preditivas.

Em resumo, este projeto demonstra como é possível criar uma infraestrutura de engenharia de dados escalável na nuvem, capaz de lidar com os desafios e oportunidades apresentados pela era do big data e da análise de dados em tempo real. Com o conhecimento adquirido neste projeto, estaremos preparados para enfrentar os desafios do mundo da análise de dados e da inteligência artificial em larga escala.

Lembrando que aqui temos os coceitos, e que cada projeto tem suas próprias regras de negócios. São a partir do dominio destes conceitos é que somos capazes de evoluir para processos maiores.

Dúvidas ou sujestões podem me procurar.
