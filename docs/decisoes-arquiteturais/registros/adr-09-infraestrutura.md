# ADR-09 — Infraestrutura e Deploy

> **Data:** Março 2026 · **Status:** ✅ Aceita

---

## Contexto

O hackathon exige demonstração de deploy em Kubernetes. Para desenvolvimento local, a equipe precisa de um ambiente funcional com todos os serviços, bancos e mensageria rodando localmente.

## Decisão

Adotar **Docker Compose** como único ambiente de execução (desenvolvimento e testes locais) e **Docker Hub** como registro de imagens.

### Ambiente Local — Docker Compose

```yaml
services:
  postgres:           # PostgreSQL 16 (identity_db + campanhas_db)
  mongodb:            # MongoDB 7 (doacoes_db)
  rabbitmq:           # RabbitMQ 3.13 (management plugin)
  identity-api:       # esperanca-identity-api
  campanhas-api:      # esperanca-campanhas-api
  worker:             # esperanca-worker
  gateway:            # esperanca-gateway-api
```

- Todos os serviços e dependências em um único `docker-compose.yml`
- RabbitMQ Management UI acessível em `localhost:15672`
- Gateway acessível em `localhost:5000`

### Publicação de Imagens — Docker Hub

As imagens são publicadas no Docker Hub sob a organização do projeto, permitindo que qualquer membro da equipe execute o ambiente com `docker compose pull && docker compose up`.

### CI/CD — GitHub Actions

```mermaid
flowchart LR
    PUSH["Push / PR\nGitHub"]
    GA["GitHub Actions"]
    BUILD["Build + Test\ndotnet test"]
    DOCKER["Docker Build\nmulti-stage"]
    HUB["Push\nDocker Hub"]

    PUSH --> GA
    GA --> BUILD
    BUILD --> DOCKER
    DOCKER --> HUB
```

| Pipeline | Trigger | Ações |
|----------|---------|-------|
| **CI** | Push/PR em qualquer branch | Restore, Build, Test, Lint |
| **CD** | Merge na `main` | CI + Docker Build + Push Docker Hub |

## Justificativa

1. **Docker Compose para dev/testes:** paridade entre ambientes; zero instalação local de PostgreSQL/MongoDB/RabbitMQ; onboarding em `docker compose up`
2. **Docker Hub:** gratuito para repositórios públicos; sem dependência de cloud provider; integração direta com GitHub Actions
3. **GitHub Actions:** CI/CD nativo do GitHub; sem custo adicional para repos públicos
4. **Um repo por serviço:** permite CI/CD independente — deploy de um serviço não afeta os outros

## Consequências

- **Positivas:** ambiente simples e reproduzível; sem custo de infraestrutura cloud; onboarding rápido
- **Negativas:** 4 pipelines de CI/CD para manter; sem ambiente de produção dedicado — adequado ao escopo de hackathon
