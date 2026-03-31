# gateway-api

**Repo:** [`fiap-ong-esperanca-gateway-api`](https://github.com/fiap-solid-link/fiap-ong-esperanca-gateway-api)  
**Tipo:** API Gateway (YARP)  
**Bounded Context:** Cross-cutting / Roteamento

## Responsabilidades

- Ponto único de entrada para clientes externos
- Roteamento para `fiap-ong-esperanca-identity-api` e `fiap-ong-esperanca-campanhas-api` via prefixo de path
- CORS configurado (origin do front-end)
- Rate Limiting — fixed window via `Microsoft.AspNetCore.RateLimiting`
- Health Check agregado — verifica identity-api e campanhas-api
- Pass-through de autenticação — repassa header `Authorization` sem validar JWT

## Estrutura de Pastas

```
src/
├── Esperanca.Gateway/                  # Program.cs, appsettings.json
```

## Tabela de Rotas YARP

| Rota no Gateway | Serviço de Destino | Autenticação |
|-----------------|--------------------|-------------|
| `/api/identity/**` | `fiap-ong-esperanca-identity-api` | Varia por endpoint |
| `/api/campanhas/**` | `fiap-ong-esperanca-campanhas-api` | GestorONG (maioria) |
| `/api/doacoes/**` | `fiap-ong-esperanca-campanhas-api` | Doador |
| `/api/transparencia/**` | `fiap-ong-esperanca-campanhas-api` | Público |
| `/health` | Gateway (agregado) | Público |

## Dependências Principais

```
.NET 10 · Microsoft.ReverseProxy (YARP) · Microsoft.AspNetCore.RateLimiting
Microsoft.ApplicationInsights.AspNetCore
```
