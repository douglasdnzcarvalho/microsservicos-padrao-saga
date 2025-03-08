<!-- omit from toc -->
# Arquitetura de Microsserviços - Padrão Saga Coreografado

![Arquitetura](docs/Imagem%20Curso.png)

<!-- omit from toc -->
### Sumário:

- [Arquitetura Proposta](#arquitetura-proposta)
- [Execução do projeto](#execução-do-projeto)
- [Acessando a aplicação](#acessando-a-aplicação)
- [Acessando tópicos com Redpanda Console](#acessando-tópicos-com-redpanda-console)
- [Dados da API](#dados-da-api)
  - [Produtos registrados e seu estoque](#produtos-registrados-e-seu-estoque)
  - [Endpoint para iniciar a saga](#endpoint-para-iniciar-a-saga)
  - [Endpoint para visualizar a saga](#endpoint-para-visualizar-a-saga)
  - [Acesso ao MongoDB](#acesso-ao-mongodb)
- [Extras](#extras)

## Arquitetura Proposta

[Voltar ao início](#sumário)

No curso, desenvolveremos a seguinte aquitetura:

![Arquitetura](docs/Arquitetura%20Proposta.png)

Em nossa arquitetura, teremos 4 serviços:

* **Order-Service**: microsserviço responsável apenas por gerar um pedido inicial, e receber uma notificação. Aqui que teremos endpoints REST para inciar o processo e recuperar os dados dos eventos. O banco de dados utilizado será o MongoDB.
* **Product-Validation-Service**: microsserviço responsável por validar se o produto informado no pedido existe e está válido. Este microsserviço guardará a validação de um produto para o ID de um pedido. O banco de dados utilizado será o PostgreSQL.
* **Payment-Service**: microsserviço responsável por realizar um pagamento com base nos valores unitários e quantidades informadas no pedido. Este microsserviço guardará a informação de pagamento de um pedido. O banco de dados utilizado será o PostgreSQL.
* **Inventory-Service**: microsserviço responsável por realizar a baixa do estoque dos produtos de um pedido. Este microsserviço guardará a informação da baixa de um produto para o ID de um pedido. O banco de dados utilizado será o PostgreSQL.

Todos os serviços da arquitetura irão subir através do arquivo **docker-compose.yml**.

## Execução do projeto

[Voltar ao início](#sumário)

Basta executar o comando no diretório raiz do repositório:

`docker-compose up --detach --wait`

**Obs.: Para rodar tudo desta maneira, é necessário primeiramente realizar o build das 5 aplicações.**

Para parar todos os containers, basta rodar:

`docker-compose down` 

Ou então:

`docker stop ($docker ps -aq)`
`docker container prune -f`

## Acessando a aplicação

[Voltar ao início](#sumário)

Para acessar as aplicações e realizar um pedido, basta acessar a URL:

http://localhost:3000/swagger-ui.html

Você chegará nesta página:

![Swagger](docs/Documentacao.png)

As aplicações executarão nas seguintes portas:

* Order-Service: 3000
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

[Voltar ao início](#sumário)

Para acessar o Redpanda Console e visualizar tópicos e publicar eventos, basta acessar:

http://localhost:8081

Você chegará nesta página:

![Redpanda](docs/Redpanda%20Kafka.png)

## Dados da API

[Voltar ao início](#sumário)

É necessário conhecer o payload de envio ao fluxo da saga, assim como os produtos cadastrados e suas quantidades.

### Produtos registrados e seu estoque

[Voltar ao nível anterior](#dados-da-api)

Existem 3 produtos iniciais cadastrados no serviço `product-validation-service` e suas quantidades disponíveis em `inventory-service`: 

* **COMIC_BOOKS** (4 em estoque)
* **BOOKS** (2 em estoque)
* **MOVIES** (5 em estoque)
* **MUSIC** (9 em estoque)

### Endpoint para iniciar a saga

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
  "id": "65235b034a6fa17dc661679b",
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
  "createdAt": "2023-10-09T01:44:35.655",
  "transactionId": "1696815875655_44ae5c2d-5549-427f-861c-9eef24676b7c",
  "totalAmount": 0,
  "totalItems": 0
}
```

### Endpoint para visualizar a saga

[Voltar ao nível anterior](#dados-da-api)

É possível recuperar os dados da saga pelo **orderId** ou pelo **transactionId**, o resultado será o mesmo:

**GET** http://localhost:3000/api/event?orderId=65235b034a6fa17dc661679b

**GET** http://localhost:3000/api/event?transactionId=1696815875655_44ae5c2d-5549-427f-861c-9eef24676b7c

Resposta:

```json
{
  "id": "65235b034a6fa17dc661679c",
  "transactionId": "1696815875655_44ae5c2d-5549-427f-861c-9eef24676b7c",
  "orderId": "65235b034a6fa17dc661679b",
  "payload": {
    "id": "65235b034a6fa17dc661679b",
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
    "createdAt": "2023-10-09T01:44:35.655",
    "transactionId": "1696815875655_44ae5c2d-5549-427f-861c-9eef24676b7c",
    "totalAmount": 56.4,
    "totalItems": 4
  },
  "source": "ORDER_SERVICE",
  "status": "SUCCESS",
  "eventHistory": [
    {
      "source": "ORDER_SERVICE",
      "status": "SUCCESS",
      "message": "Saga started!",
      "createdAt": "2023-10-09T01:44:35.728"
    },
    {
      "source": "PRODUCT_VALIDATION_SERVICE",
      "status": "SUCCESS",
      "message": "Products are validated successfully!",
      "createdAt": "2023-10-09T01:44:36.196"
    },
    {
      "source": "PAYMENT_SERVICE",
      "status": "SUCCESS",
      "message": "Payment realized successfully!",
      "createdAt": "2023-10-09T01:44:36.639"
    },
    {
      "source": "INVENTORY_SERVICE",
      "status": "SUCCESS",
      "message": "Inventory updated successfully!",
      "createdAt": "2023-10-09T01:44:37.117"
    },
    {
      "source": "ORDER_SERVICE",
      "status": "SUCCESS",
      "message": "Saga finished successfully!",
      "createdAt": "2023-10-09T01:44:37.21"
    }
  ],
  "createdAt": "2023-10-09T01:44:37.209"
}
```

### Acesso ao MongoDB

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

**db.order.find(id=ObjectId("65235b034a6fa17dc661679b"))**

**db.order.find({ "products.product.code": "COMIC_BOOKS"})**

## Extras

Os fontes desse projeto foram obtidos inicialmente através do fork [desse projeto](https://github.com/vhnegrisoli/curso-udemy-microsservicos-padrao-saga-coreografado).