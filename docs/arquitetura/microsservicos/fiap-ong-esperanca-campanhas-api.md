# fiap-ong-esperanca-campanhas-api

**Repo:** [`fiap-ong-esperanca-campanhas-api`](https://github.com/fiap-solid-link/fiap-ong-esperanca-campanhas-api)  
**Tipo:** ASP.NET Core Web API  
**Bancos:** PostgreSQL (`campanhas_db`) + MongoDB (read models de transparência)  
**Bounded Context:** Campanhas + Transparência

## Responsabilidades

- CRUD de Campanhas (GestorONG)
- Ciclo de vida completo: Cadastrada → EmAndamento → Concluida/Cancelada
- Endpoint de intenção de doação → valida campanha → publica `DoacaoRecebidaEvent` no RabbitMQ
- Consumer de `DoacaoProcessadaEvent` → atualiza `ValorArrecadado` → verifica meta → conclui campanha se necessário
- Scheduler de vencimento (BackgroundService) → verifica campanhas expiradas e próximas do vencimento
- Endpoints de Transparência — leitura de read models no MongoDB

**Entidade de domínio:** `Campanha`

!!! note "Nota sobre Auth"
    Este serviço valida o JWT emitido pelo `fiap-ong-esperanca-identity-api` usando a mesma chave simétrica (shared secret via ConfigMap/Secret do K8s). Não realiza chamadas ao identity-api em runtime.

## Estrutura de Pastas (Clean Architecture)

```
src/
├── Esperanca.Campanhas.WebApi/         # Controllers, Middleware, Swagger, Program.cs
├── Esperanca.Campanhas.Application/    # Use Cases, Handlers (MediatR), DTOs, Validators
├── Esperanca.Campanhas.Domain/         # Entidades, Value Objects, Enums, Interfaces
├── Esperanca.Campanhas.Infrastructure/ # EF Core, Repositories, RabbitMQ Publisher/Consumer
tests/
├── Esperanca.Campanhas.UnitTests/
├── Esperanca.Campanhas.IntegrationTests/
```

## Endpoints

| Método | Rota | Acesso | Descrição |
|--------|------|--------|-----------|
| POST | `/api/campanhas` | GestorONG | Criar campanha |
| PUT | `/api/campanhas/{id}` | GestorONG | Editar campanha (status Cadastrada) |
| POST | `/api/campanhas/{id}/ativar` | GestorONG | Ativar campanha |
| POST | `/api/campanhas/{id}/prorrogar` | GestorONG | Prorrogar DataFim |
| POST | `/api/campanhas/{id}/cancelar` | GestorONG | Cancelar campanha |
| GET | `/api/campanhas/{id}` | GestorONG | Detalhe da campanha (gestão) |
| GET | `/api/campanhas` | GestorONG | Listar campanhas do gestor |
| POST | `/api/doacoes` | Doador | Enviar intenção de doação |
| GET | `/api/transparencia/painel` | Público | Visão macro arrecadação |
| GET | `/api/transparencia/campanhas` | Público | Lista de campanhas |
| GET | `/api/transparencia/campanhas/{id}` | Público | Detalhe com doações anonimizadas |
| GET | `/health` | Público | Health check (PostgreSQL + MongoDB + RabbitMQ) |

## Dependências Principais

```
.NET 10 · ASP.NET Core · EF Core + Npgsql · MongoDB.Driver
MediatR · FluentValidation · RabbitMQ.Client
Microsoft.AspNetCore.Authentication.JwtBearer
Serilog + Sinks.ApplicationInsights · Swashbuckle
```
