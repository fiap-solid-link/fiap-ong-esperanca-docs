# fiap-ong-esperanca-identity-api

**Repo:** [`fiap-ong-esperanca-identity-api`](https://github.com/fiap-solid-link/fiap-ong-esperanca-identity-api)  
**Tipo:** ASP.NET Core Web API  
**Banco:** PostgreSQL (`identity_db`)  
**Bounded Context:** Identidade e Acesso

## Responsabilidades

- Registro de Doador (público)
- Registro/Seed de GestorONG
- Login (autenticação) → emissão de JWT (access 30min + refresh 7 dias)
- Validação de token via shared key
- Gerenciamento de perfis/roles (RBAC): Admin, GestorONG, Doador

**Entidades de domínio:** `Usuario`, `Role`

## Estrutura de Pastas (Clean Architecture)

```
src/
├── Esperanca.Identity.WebApi/         # Controllers, Middleware, Swagger, Program.cs
├── Esperanca.Identity.Application/    # Use Cases, Handlers (MediatR), DTOs, Validators
├── Esperanca.Identity.Domain/         # Entidades, Value Objects, Enums, Interfaces
├── Esperanca.Identity.Infrastructure/ # EF Core, Repositories, JWT Service
tests/
├── Esperanca.Identity.UnitTests/      # xUnit tests
├── Esperanca.Identity.IntegrationTests/
```

## Endpoints

| Método | Rota | Acesso | Descrição |
|--------|------|--------|-----------|
| POST | `/api/auth/registrar` | Público | Registrar novo doador |
| POST | `/api/auth/login` | Público | Autenticar e obter JWT |
| POST | `/api/auth/refresh` | Autenticado | Renovar access token |
| GET | `/api/auth/me` | Autenticado | Dados do usuário logado |
| PUT | `/api/usuarios/perfil` | Autenticado | Atualizar perfil (nome, apelido) |
| POST | `/api/usuarios/{id}/conceder-gestor` | Admin | Conceder perfil GestorONG |
| DELETE | `/api/usuarios/{id}/revogar-gestor` | Admin | Revogar perfil GestorONG |
| GET | `/health` | Público | Health check (PostgreSQL) |

## Dependências Principais

```
.NET 10 · ASP.NET Core · EF Core + Npgsql · MediatR · FluentValidation
BCrypt.Net-Next · Microsoft.AspNetCore.Authentication.JwtBearer
Serilog + Sinks.ApplicationInsights · Swashbuckle
```
