# API de Check-in de Eventos

API simples desenvolvida em **Node.js + Express** para gerenciamento básico de eventos, participantes e check-in.

O projeto utiliza dados em memória para simular eventos cadastrados e permite listar eventos, consultar detalhes, buscar participantes e registrar presença.

---

## Tecnologias utilizadas

- Node.js
- Express
- CORS
- Helmet
- Morgan
- Dotenv

---

## Funcionalidades

- Verificar status da API.
- Listar todos os eventos cadastrados.
- Consultar detalhes de um evento.
- Listar participantes de um evento.
- Buscar participantes por nome, e-mail ou documento.
- Paginar a lista de participantes.
- Realizar check-in de participantes.
- Bloquear check-in duplicado.
- Utilizar autenticação opcional via token Bearer.

---

## Estrutura básica

```txt
projeto/
│
├── server.js
├── package.json
├── .env
└── README.md
```

---

## Instalação

Clone o repositório e acesse a pasta do projeto:

```bash
git clone <url-do-repositorio>
cd <nome-do-projeto>
```

Instale as dependências:

```bash
npm install
```

Caso esteja configurando do zero, instale:

```bash
npm install express cors helmet morgan dotenv
```

---

## Configuração do ambiente

Crie um arquivo `.env` na raiz do projeto:

```env
PORT=5044
TOKEN=seu_token_opcional
```

A variável `TOKEN` é opcional.

Se `TOKEN` não for definida, a API funcionará sem autenticação.

Se `TOKEN` for definida, todas as rotas precisarão receber o cabeçalho:

```txt
Authorization: Bearer seu_token_opcional
```

---

## Executando o projeto

Para iniciar a API:

```bash
node server.js
```

Ou, usando nodemon:

```bash
npx nodemon server.js
```

A API será iniciada na porta definida no `.env`. Caso nenhuma porta seja informada, será usada a porta padrão:

```txt
5044
```

---

## Rotas da API

### Health Check

Verifica se a API está funcionando.

```http
GET /health
```

Resposta esperada:

```json
{
  "status": "ok",
  "time": "2026-04-28T00:00:00.000Z"
}
```

---

## Eventos

### Listar todos os eventos

```http
GET /events
```

Resposta esperada:

```json
[
  {
    "id": "evt_123",
    "title": "Conferência de Tecnologia 2025",
    "startsAt": "2025-09-15T09:00:00-03:00",
    "endsAt": "2025-09-15T18:00:00-03:00",
    "location": "Auditório AMF",
    "stats": {
      "total": 3,
      "checkedIn": 1,
      "absent": 2
    }
  }
]
```

---

### Detalhar um evento

```http
GET /events/:id
```

Exemplo:

```http
GET /events/evt_123
```

Caso o evento exista, a API retorna os dados do evento e suas estatísticas.

Caso o evento não exista:

```json
{
  "error": "Event not found"
}
```

Status HTTP:

```txt
404 Not Found
```

---

## Participantes

### Listar participantes de um evento

```http
GET /events/:id/attendees
```

Exemplo:

```http
GET /events/evt_123/attendees
```

Resposta esperada:

```json
{
  "data": [
    {
      "id": "att_001",
      "name": "Ana Souza",
      "email": "ana@exemplo.com",
      "document": "123.456.789-00",
      "checkedInAt": null
    }
  ],
  "page": 1,
  "limit": 20,
  "total": 3
}
```

---

### Buscar participantes

A busca pode ser feita por nome, e-mail ou documento.

```http
GET /events/:id/attendees?search=ana
```

Exemplo:

```http
GET /events/evt_123/attendees?search=ana
```

---

### Paginar participantes

```http
GET /events/:id/attendees?page=1&limit=10
```

Exemplo:

```http
GET /events/evt_123/attendees?page=1&limit=2
```

Parâmetros disponíveis:

| Parâmetro | Descrição | Padrão |
|---|---|---|
| `search` | Termo de busca por nome, e-mail ou documento | vazio |
| `page` | Página atual | 1 |
| `limit` | Quantidade de itens por página | 20 |

---

## Check-in

### Realizar check-in de participante

```http
POST /events/:id/checkin
```

Exemplo:

```http
POST /events/evt_123/checkin
```

Body JSON:

```json
{
  "attendeeId": "att_001"
}
```

Resposta esperada:

```json
{
  "attendeeId": "att_001",
  "checkedInAt": "2026-04-28T00:00:00.000Z"
}
```

Status HTTP:

```txt
201 Created
```

---

## Erros possíveis no check-in

### Participante não informado

```json
{
  "error": "attendeeId is required"
}
```

Status HTTP:

```txt
400 Bad Request
```

### Participante não pertence ao evento

```json
{
  "error": "Attendee not in this event"
}
```

Status HTTP:

```txt
422 Unprocessable Entity
```

### Participante já realizou check-in

```json
{
  "attendeeId": "att_003",
  "checkedInAt": "2025-09-15T09:32:12-03:00"
}
```

Status HTTP:

```txt
409 Conflict
```

---

## Autenticação opcional

A autenticação é controlada pela variável `TOKEN`.

### Sem token

```env
PORT=5044
```

A API funcionará sem autenticação.

### Com token

```env
PORT=5044
TOKEN=meu_token_secreto
```

As requisições deverão enviar:

```txt
Authorization: Bearer meu_token_secreto
```

Exemplo:

```bash
curl -H "Authorization: Bearer meu_token_secreto" http://localhost:5044/events
```

---

## Exemplos de teste com curl

### Health

```bash
curl http://localhost:5044/health
```

### Listar eventos

```bash
curl http://localhost:5044/events
```

### Detalhar evento

```bash
curl http://localhost:5044/events/evt_123
```

### Listar participantes

```bash
curl http://localhost:5044/events/evt_123/attendees
```

### Buscar participante

```bash
curl "http://localhost:5044/events/evt_123/attendees?search=ana"
```

### Realizar check-in

```bash
curl -X POST http://localhost:5044/events/evt_123/checkin \
  -H "Content-Type: application/json" \
  -d "{\"attendeeId\":\"att_001\"}"
```

---

## Dados em memória

Os eventos e participantes estão armazenados diretamente no código.

Isso significa que:

- ao reiniciar a API, os dados voltam ao estado inicial;
- os check-ins realizados não são persistidos em banco de dados;
- o projeto é ideal para estudo, testes e demonstração de API REST.

Para uso real, recomenda-se integrar com um banco de dados, como MongoDB, PostgreSQL, MySQL ou SQLite.

---

## Melhorias futuras

- Persistir eventos e participantes em banco de dados.
- Criar cadastro de eventos.
- Criar cadastro de participantes.
- Adicionar autenticação com JWT.
- Criar níveis de permissão para usuários.
- Criar documentação Swagger/OpenAPI.
- Criar frontend web ou mobile para check-in.
- Gerar QR Code para participantes.
- Exportar lista de presença em PDF ou CSV.

---

## Observações importantes

Este projeto é uma API didática e utiliza dados em memória.

Ele é adequado para aprendizado de rotas REST, middlewares, autenticação simples, busca, paginação e controle de check-in.

---

## Autor

Desenvolvido para fins acadêmicos e de estudo.
