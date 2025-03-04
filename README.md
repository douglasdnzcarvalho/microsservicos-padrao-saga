# Arquitetura de Microsserviços - Padrões *Saga Orquestrado* e *Saga Coreografado*

Este repositório contém duas aplicações que foram desenvolvidas baseadas nos padrões de arquitetura que aprendi no curso [Arquitetura de Microsserviços: Padrão Saga Orquestrado](https://www.udemy.com/course/arquitetura-de-microsservicos-padrao-saga-orquestrado/), ministrado por [Victor Hugo Negrisoli](https://github.com/vhnegrisoli).
No diretório [orquestrado](/orquestrado/) está o código seguindo o padrão *Saga Orquestrado* e no diretório [coreografado](/coreografado/) está o código seguindo o padrão *Saga Coreografado*.

Ambas as aplicações consistem em um conjunto microsserviços que fazem o registro de um pedido, depois validam os produtos, realizam o pagamento e após isso fazem a baixa do estoque. A ideia é que ambas aceitam as mesmas entradas e geram as mesmas saídas, tendo como diferencial apenas o padrão aplicado.

## Tecnologias

* **Java 17**
* **Spring Boot 3**
* **Apache Kafka**
* **API REST**
* **PostgreSQL**
* **MongoDB**
* **Docker**
* **docker-compose**
* **Redpanda Console**

# Ferramentas utilizadas

* **IntelliJ IDEA Community Edition**
* **Docker**
* **Gradle**

Espero que este projeto seja útil para você! Se tiver alguma dúvida ou precisar de ajuda, não hesite em entrar em contato!

Material de estudo extra:

* https://microservices.io/patterns/data/saga.html