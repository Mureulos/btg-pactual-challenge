<p align="center" width="100%">
    <img width="50%" src="https://imgs.search.brave.com/1mj0FYgou-C23CXFXyhuNx4VVv0IN_eEyKmnNfH7hxg/rs:fit:860:0:0:0/g:ce/aHR0cHM6Ly9icmFu/ZGNlbnRlci5idGdw/YWN0dWFsLmNvbS9h/c3NldHMvaW1hZ2Vz/L3Zpc3VhbC1ndWlk/ZS9idGctZGFyay1i/bHVlLnBuZw "> 
</p>

<h3 align="center">
  Desafio Backend do BTG Pactual
</h3>

<p align="center">
  <img alt="License: MIT" src="https://img.shields.io/badge/license-MIT-%2304D361">
  <img alt="Language: Java" src="https://img.shields.io/badge/language-java-green">
  <img alt="Version: 1.0" src="https://img.shields.io/badge/version-1.0-yellowgreen">
</p>

[Se inscreva em nosso canal no Youtube!](https://www.youtube.com/@buildrun-tech?sub_confirmation=1)

Para um maior entendimento do código deste repositório, [assista nosso vídeo no Youtube, clique aqui.](https://www.youtube.com/watch?v=e_WgAB0Th_I)

## Desafio
- Confira o enunciado completo, [clicando aqui](./CHALLENGE.md).

## Tecnologias utilizadas

* Java 25
* Spring Boot 4.1
* Spring Data MongoDB
* Spring AMQP (RabbitMQ)
* Maven (wrapper incluso: `mvnw`)
* Docker e Docker Compose

## Pré-requisitos

| Dependência | Versão | Uso |
|-------------|--------|-----|
| [JDK](https://adoptium.net/) | 25 | Compilar e executar a aplicação |
| [Docker](https://docs.docker.com/get-docker/) | 20+ | Executar MongoDB e RabbitMQ |
| [Docker Compose](https://docs.docker.com/compose/install/) | v2 (`docker compose`) | Orquestrar os containers |
| Maven | opcional | Já incluso via `./mvnw` |

Verifique a instalação:

```bash
java -version
docker --version
docker compose version
```

## Subindo a infraestrutura (MongoDB e RabbitMQ)

O arquivo [`docker-compose.yaml`](./docker-compose.yaml) sobe os dois serviços:

| Serviço | Container | Portas | Credenciais |
|---------|-----------|--------|-------------|
| MongoDB | `btg-challenge-mongodb` | `27017` | `admin` / `123` |
| RabbitMQ | `btg-challenge-rabbitmq` | `5672` (AMQP), `15672` (painel) | `guest` / `guest` |

```bash
# Subir os containers em segundo plano
docker compose up -d

# Verificar se estão rodando
docker compose ps

# Acompanhar os logs
docker compose logs -f

# Parar os containers (mantém os dados do MongoDB)
docker compose down

# Parar e apagar também os dados do MongoDB
docker compose down -v
```

- Painel do RabbitMQ: http://localhost:15672 (`guest` / `guest`)
- String de conexão do MongoDB: `mongodb://admin:123@localhost:27017/orderms?authSource=admin`

## Executando a aplicação

Com os containers no ar, na raiz do projeto:

```bash
# Linux / macOS
./mvnw spring-boot:run

# Windows
mvnw.cmd spring-boot:run
```

A API ficará disponível em `http://localhost:8080`.

Outros comandos úteis:

```bash
./mvnw clean package       # Compila e gera o JAR em target/
./mvnw test                # Executa os testes
java -jar target/orderms-0.0.1-SNAPSHOT.jar   # Executa o JAR gerado
```

> As configurações (conexão com o MongoDB) ficam em `src/main/resources/application.properties`.
> A conexão com o RabbitMQ usa os padrões do Spring (`localhost:5672`, `guest`/`guest`).

## Publicando um pedido (RabbitMQ)

A aplicação consome a fila `btg-pactual-order-created`, criada automaticamente na inicialização.

1. Acesse http://localhost:15672 → **Queues and Streams** → `btg-pactual-order-created`.
2. Em **Publish message**, informe o payload abaixo e clique em **Publish message**:

```json
{
  "codigoPedido": 1001,
  "codigoCliente": 1,
  "itens": [
    { "produto": "lápis", "quantidade": 100, "preco": 1.10 },
    { "produto": "caderno", "quantidade": 10, "preco": 1.00 }
  ]
}
```

Em **Properties**, adicione `content_type` = `application/json`.

## Consultando pedidos (API)

```bash
curl "http://localhost:8080/customers/1/orders?page=0&pageSize=10"
```

Resposta de exemplo:

```json
{
  "summary": { "totalOnOrders": 120.00 },
  "data": [ { "orderId": 1001, "customerId": 1, "total": 120.00 } ],
  "pagination": { "page": 0, "pageSize": 10, "totalElements": 1, "totalPages": 1 }
}
```

| Parâmetro | Padrão | Descrição |
|-----------|--------|-----------|
| `page` | `0` | Página (começa em 0) |
| `pageSize` | `10` | Itens por página |

## Como interagir com as ferramentas?

- Banco de dados: [MongoDB Compass](https://www.mongodb.com/products/tools/compass)
- API: [Postman](https://www.postman.com/equipoweb5/web/overview?sideView=agentMode)

## Problemas comuns

- **Porta ocupada (27017/5672/15672/8080):** pare o serviço que a utiliza ou altere o mapeamento de portas no `docker-compose.yaml`.
- **Erro de autenticação no MongoDB:** confirme que o `authSource=admin` está na URI e que as credenciais batem com o compose.
- **`Connection refused` no RabbitMQ:** aguarde alguns segundos após o `docker compose up -d` e verifique com `docker compose ps`.
- **Versão do Java incorreta:** o projeto exige JDK 25 (`java -version`).

## Agradecimentos

Agradecimento especial ao canal [Build & Run](https://www.youtube.com/@buildrun-tech) pelo desafio e pela solução apresentada no vídeo [Desafio Backend do BTG Pactual](https://www.youtube.com/watch?v=e_WgAB0Th_I), que serviu de base para este projeto.
