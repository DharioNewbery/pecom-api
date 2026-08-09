# PECOM-API

Backend do **Sistema de Gestão de Turmas e Tarefas** - API RESTful em Go, responsável por autenticação, gestão de turmas/tarefas/materiais/calendário, métricas e correção automatizada de algoritmos.

> Visão geral do projeto, contrato de API e ADRs: ver o repositório [`pecom`](../pecom/).

## Stack

| Item | Tecnologia |
|---|---|
| Linguagem | Go |
| Framework HTTP | Gin |
| Acesso a dados | sqlc (SQL puro, código type-safe gerado) |
| Migrations | golang-migrate |
| Autenticação | JWT (access + refresh) |
| Filas/jobs | Asynq (Redis) |
| Cache | Redis |
| Banco de dados | PostgreSQL |
| Testes | `testing` + testify + httptest |
| Lint | golangci-lint |
| Docs da API | swaggo (gerado a partir de comentários) — contrato oficial em `pecom/openapi.yaml` |

## Estrutura de pastas

```
pecom-api/
├── cmd/
│   ├── api/              # entrypoint da API HTTP
│   └── worker/           # entrypoint do worker de correção automatizada
├── internal/
│   ├── turma/
│   ├── tarefa/
│   ├── submissao/
│   ├── usuario/
│   ├── material/
│   ├── calendario/
│   ├── metrica/
│   ├── sandbox/          # execução isolada de código em containers Docker
│   ├── middleware/
│   └── config/
├── db/
│   ├── migrations/       # golang-migrate
│   └── queries/          # .sql usadas pelo sqlc
├── pkg/
├── docker/
│   └── runners/          # Dockerfiles das imagens de execução por linguagem
├── sqlc.yaml
├── go.mod
└── Dockerfile
```

Cada módulo em `internal/` segue sempre o padrão: `handler.go` (HTTP) -> `service.go` (regra de negócio) -> `repository.go` (acesso a dados) -> `model.go`.

## Rodando localmente

Este repositório normalmente é executado via `docker compose` a partir do repositório [`pecom`](../pecom/). Para rodar isoladamente:

```bash
cp .env.example .env

# subir apenas dependências (postgres + redis) via pecom/docker-compose.dev.yml
# ou apontar para instâncias já rodando

go mod download
migrate -path db/migrations -database "$DATABASE_URL" up

go run ./cmd/api      # sobe a API HTTP
go run ./cmd/worker   # sobe o worker de correção (em outro terminal)
```

## Testes e lint

```bash
go test ./...
golangci-lint run
```

- Testes unitários obrigatórios na camada de `service`.
- Testes de integração para endpoints críticos (auth, submissão, correção).

## Convenções

- Erros sempre tratados explicitamente.
- Interfaces de `Repository` em cada módulo, para permitir mocks nos testes de `service`.
- Rotas versionadas: `/api/v1/...`.
- Respostas de erro em formato único e padronizado.
- Paginação padronizada (`?page=&limit=`) em todas as listagens.
- Commits seguindo [Conventional Commits](https://www.conventionalcommits.org/).

## Documentação viva

Após subir o serviço, a documentação Swagger fica disponível em `/docs`. O contrato formal (fonte de verdade) fica em `pecom/openapi.yaml`.

## Repositórios relacionados

- [`pecom-web`](../pecom-web) — frontend que consome esta API
- [`pecom`](../pecom) — contrato de API, ADRs, orquestração local