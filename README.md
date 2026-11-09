# RPG API

API REST para cadastro de itens de RPG, feita para o Trabalho 1 de Desenvolvimento de Sistemas.

**Alunos:**
- Miguel Hecht Lemos Gouvêa
- Fabio Roberto Queiroz Costa Junior

## Sobre o trabalho

A aplicação é um CRUD completo de uma única entidade, o `Item` (espadas, armaduras, poções, etc.), com validação dos dados e tratamento de erros.

Tecnologias usadas:

- Java 17
- Spring Boot 3 (Web, Data JPA e Validation)
- Banco H2 (salvo em arquivo)
- Maven

## Como rodar

Você precisa ter o JDK 17 (ou superior) e o Maven instalados.

Na pasta do projeto, rode:

```bash
mvn clean package
mvn spring-boot:run
```

A API sobe em `http://localhost:8081`.

O banco é criado automaticamente na primeira execução, dentro da pasta `data/`. Para ver os dados pelo navegador, acesse `http://localhost:8081/h2-console` com:

- JDBC URL: `jdbc:h2:file:./data/rpgdb`
- Usuário: `sa`
- Senha: (deixar em branco)

## Entidade Item

| Campo | Tipo | Regras |
|---|---|---|
| id | Long | Chave primária, gerada automaticamente |
| nome | String | Obrigatório, de 2 a 100 caracteres, não pode repetir |
| raridade | Enum | Obrigatório: COMUM, INCOMUM, RARO, EPICO ou LENDARIO |
| preco | Decimal | Obrigatório, maior ou igual a 0 |
| nivelMinimo | Integer | Opcional, de 1 a 100 |

## Rotas

| Método | Rota | O que faz | Resposta |
|---|---|---|---|
| GET | /api/itens | Lista todos os itens (aceita `?nome=` para filtrar) | 200 |
| GET | /api/itens/{id} | Busca um item pelo id | 200 |
| POST | /api/itens | Cria um item | 201 |
| PUT | /api/itens/{id} | Atualiza um item por completo | 200 |
| DELETE | /api/itens/{id} | Remove um item | 204 |

Códigos de erro:

- 400: dados inválidos ou JSON mal formado
- 404: id não encontrado
- 409: já existe um item com esse nome

## Exemplos de uso

Os mesmos pedidos também estão prontos no arquivo `requests.http`, para usar com a extensão REST Client do VS Code.

### Criar item (POST)

```bash
curl -i -X POST http://localhost:8081/api/itens \
  -H "Content-Type: application/json" \
  -d '{"nome":"Espada Flamejante","raridade":"EPICO","preco":1500.00,"nivelMinimo":20}'
```

Resposta (201 Created):

```json
{
  "id": 1,
  "nome": "Espada Flamejante",
  "raridade": "EPICO",
  "preco": 1500.00,
  "nivelMinimo": 20
}
```

### Listar, filtrar e buscar por id (GET)

```bash
curl -i http://localhost:8081/api/itens
curl -i "http://localhost:8081/api/itens?nome=espada"
curl -i http://localhost:8081/api/itens/1
```

Resposta (200 OK): o item, ou a lista de itens, em JSON.

### Atualizar item (PUT)

```bash
curl -i -X PUT http://localhost:8081/api/itens/1 \
  -H "Content-Type: application/json" \
  -d '{"nome":"Espada Flamejante +1","raridade":"LENDARIO","preco":3000.00,"nivelMinimo":25}'
```

Resposta (200 OK): o item com os dados atualizados.

### Remover item (DELETE)

```bash
curl -i -X DELETE http://localhost:8081/api/itens/1
```

Resposta: 204 No Content, sem corpo.

## Exemplos de erro

### 400 - dados inválidos

```bash
curl -i -X POST http://localhost:8081/api/itens \
  -H "Content-Type: application/json" \
  -d '{"nome":"","preco":-5,"nivelMinimo":500}'
```

```json
{
  "timestamp": "2026-10-26T10:00:00",
  "status": 400,
  "mensagem": "Erro de validação",
  "erros": {
    "nome": "O nome é obrigatório",
    "raridade": "A raridade é obrigatória (COMUM, INCOMUM, RARO, EPICO ou LENDARIO)",
    "preco": "O preço não pode ser negativo",
    "nivelMinimo": "O nível mínimo deve ser no máximo 100"
  }
}
```

### 404 - item não encontrado

`GET /api/itens/999`

```json
{
  "status": 404,
  "mensagem": "Item com id 999 não encontrado"
}
```

### 409 - nome repetido

Ao criar dois itens com o mesmo nome:

```json
{
  "status": 409,
  "mensagem": "Já existe um item com o nome 'Espada Flamejante'"
}
```

## Organização do código

```
src/main/java/com/rpg/api/
├── controller/   ItemController (endpoints)
├── model/        Item e Raridade
├── repository/   ItemRepository (JpaRepository)
└── exception/    exceções próprias e GlobalExceptionHandler
```
