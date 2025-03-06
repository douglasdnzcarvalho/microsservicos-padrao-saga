# Arquitetura de Microsserviços - Padrão Saga Orquestrado

![Arquitetura](docs/Imagem%20Curso.png)

### Sumário:

* [Arquitetura Proposta](#arquitetura-proposta)
* [Execução do projeto](#execu%C3%A7%C3%A3o-do-projeto)
* [Acessando a aplicação](#acessando-a-aplica%C3%A7%C3%A3o)
* [Acessando tópicos com Redpanda Console](#acessando-t%C3%B3picos-com-redpanda-console)
* [Dados da API](#dados-da-api)
  * [Produtos registrados e seu estoque](#produtos-registrados-e-seu-estoque)
  * [Endpoint para iniciar a saga](#endpoint-para-iniciar-a-saga)
  * [Endpoint para visualizar a saga](#endpoint-para-visualizar-a-saga)
  * [Acesso ao MongoDB](#acesso-ao-mongodb)
* [Extras](#extras)

# Arquitetura Proposta

[Voltar ao início](#sum%C3%A1rio)

No curso, desenvolveremos a seguinte aquitetura:

![Arquitetura](docs/Arquitetura%20Proposta.png)

Em nossa arquitetura, teremos 5 serviços:

* **Order-Service**: microsserviço responsável apenas por gerar um pedido inicial, e receber uma notificação. Aqui que teremos endpoints REST para inciar o processo e recuperar os dados dos eventos. O banco de dados utilizado será o MongoDB.
* **Orchestrator-Service**: microsserviço responsável por orquestrar todo o fluxo de execução da Saga, ele que saberá qual microsserviço foi executado e em qual estado, e para qual será o próximo microsserviço a ser enviado, este microsserviço também irá salvar o processo dos eventos. Este serviço não possui banco de dados.
* **Product-Validation-Service**: microsserviço responsável por validar se o produto informado no pedido existe e está válido. Este microsserviço guardará a validação de um produto para o ID de um pedido. O banco de dados utilizado será o PostgreSQL.
* **Payment-Service**: microsserviço responsável por realizar um pagamento com base nos valores unitários e quantidades informadas no pedido. Este microsserviço guardará a informação de pagamento de um pedido. O banco de dados utilizado será o PostgreSQL.
* **Inventory-Service**: microsserviço responsável por realizar a baixa do estoque dos produtos de um pedido. Este microsserviço guardará a informação da baixa de um produto para o ID de um pedido. O banco de dados utilizado será o PostgreSQL.

Todos os serviços da arquitetura irão subir através do arquivo **docker-compose.yml**.

## Execução do projeto

Basta executar o comando no diretório raiz do repositório:

`docker-compose up --detach --wait`

**Obs.: para rodar tudo desta maneira, é necessário realizar o build das 5 aplicações, veja nos passos abaixo sobre como fazer isto.**

Para parar todos os containers, basta rodar:

`docker-compose down` 

Ou então:

`docker stop ($docker ps -aq)`
`docker container prune -f`

## Acessando a aplicação

[Voltar ao início](#sum%C3%A1rio)

Para acessar as aplicações e realizar um pedido, basta acessar a URL:

http://localhost:3000/swagger-ui.html

Você chegará nesta página:

![Swagger](docs/Documentacao.png)

As aplicações executarão nas seguintes portas:

* Order-Service: 3000
* Orchestrator-Service: 8080
* Product-Validation-Service: 8090
* Payment-Service: 8091
* Inventory-Service: 8092
* Apache Kafka: 9092
* Redpanda Console: 8081
* PostgreSQL (Product-DB): 5432
* PostgreSQL (Payment-DB): 5433
* PostgreSQL (Inventory-DB): 5434
* MongoDB (Order-DB): 27017

## Acessando tópicos com Redpanda Console

[Voltar ao início](#sum%C3%A1rio)

Para acessar o Redpanda Console e visualizar tópicos e publicar eventos, basta acessar:

http://localhost:8081

Você chegará nesta página:

![Redpanda](docs/Redpanda%20Kafka.png)

## Dados da API

[Voltar ao início](#sum%C3%A1rio)

É necessário conhecer o payload de envio ao fluxo da saga, assim como os produtos cadastrados e suas quantidades.

### Produtos registrados e seu estoque

[Voltar ao nível anterior](#dados-da-api)

Existem 3 produtos iniciais cadastrados no serviço `product-validation-service` e suas quantidades disponíveis em `inventory-service`: 

* **COMIC_BOOKS** (4 em estoque)
* **BOOKS** (2 em estoque)
* **MOVIES** (5 em estoque)
* **MUSIC** (9 em estoque)

### Endpoint para iniciar a saga:

[Voltar ao nível anterior](#dados-da-api)

**POST** http://localhost:3000/api/order

Payload:

```json
{
  "products": [
    {
      "product": {
        "code": "COMIC_BOOKS",
        "unitValue": 15.50
      },
      "quantity": 3
    },
    {
      "product": {
        "code": "BOOKS",
        "unitValue": 9.90
      },
      "quantity": 1
    }
  ]
}
```

Resposta:

```json
{
  "id": "64429e987a8b646915b3735f",
  "products": [
    {
      "product": {
        "code": "COMIC_BOOKS",
        "unitValue": 15.5
      },
      "quantity": 3
    },
    {
      "product": {
        "code": "BOOKS",
        "unitValue": 9.9
      },
      "quantity": 1
    }
  ],
  "createdAt": "2023-04-21T14:32:56.335943085",
  "transactionId": "1682087576536_99d2ca6c-f074-41a6-92e0-21700148b519"
}
```

### Endpoint para visualizar a saga:

[Voltar ao nível anterior](#dados-da-api)

É possível recuperar os dados da saga pelo **orderId** ou pelo **transactionId**, o resultado será o mesmo:

**GET** http://localhost:3000/api/event?orderId=64429e987a8b646915b3735f

**GET** http://localhost:3000/api/event?transactionId=1682087576536_99d2ca6c-f074-41a6-92e0-21700148b519

Resposta:

```json
{
  "id": "64429e9a7a8b646915b37360",
  "transactionId": "1682087576536_99d2ca6c-f074-41a6-92e0-21700148b519",
  "orderId": "64429e987a8b646915b3735f",
  "payload": {
    "id": "64429e987a8b646915b3735f",
    "products": [
      {
        "product": {
          "code": "COMIC_BOOKS",
          "unitValue": 15.5
        },
        "quantity": 3
      },
      {
        "product": {
          "code": "BOOKS",
          "unitValue": 9.9
        },
        "quantity": 1
      }
    ],
    "totalAmount": 56.40,
    "totalItems": 4,
    "createdAt": "2023-04-21T14:32:56.335943085",
    "transactionId": "1682087576536_99d2ca6c-f074-41a6-92e0-21700148b519"
  },
  "source": "ORCHESTRATOR",
  "status": "SUCCESS",
  "eventHistory": [
    {
      "source": "ORCHESTRATOR",
      "status": "SUCCESS",
      "message": "Saga started!",
      "createdAt": "2023-04-21T14:32:56.78770516"
    },
    {
      "source": "PRODUCT_VALIDATION_SERVICE",
      "status": "SUCCESS",
      "message": "Products are validated successfully!",
      "createdAt": "2023-04-21T14:32:57.169378616"
    },
    {
      "source": "PAYMENT_SERVICE",
      "status": "SUCCESS",
      "message": "Payment realized successfully!",
      "createdAt": "2023-04-21T14:32:57.617624655"
    },
    {
      "source": "INVENTORY_SERVICE",
      "status": "SUCCESS",
      "message": "Inventory updated successfully!",
      "createdAt": "2023-04-21T14:32:58.139176809"
    },
    {
      "source": "ORCHESTRATOR",
      "status": "SUCCESS",
      "message": "Saga finished successfully!",
      "createdAt": "2023-04-21T14:32:58.248630293"
    }
  ],
  "createdAt": "2023-04-21T14:32:58.28"
}
```

### Acesso ao MongoDB

[Voltar ao início](#sum%C3%A1rio)

Para conectar-se ao MongoDB via linha de comando (cli) diretamente do docker-compose, basta executar o comando abaixo:

**docker exec -it order-db mongosh "mongodb://admin:123456@localhost:27017"**

Para listar os bancos de dados existentes:

**show dbs**

Para selecionar um banco de dados:

**use admin**

Para visualizar as collections do banco:

**show collections**

Para realizar queries e validar se os dados existem:

**db.order.find()**

**db.event.find()**

**db.order.find(id=ObjectId("65006786d715e21bd38d1634"))**

**db.order.find({ "products.product.code": "COMIC_BOOKS"})**

## Extras

Os fontes desse projeto foram obtidos inicialmente através do fork [desse projeto](https://github.com/vhnegrisoli/curso-udemy-microsservicos-padrao-saga-orquestrado).