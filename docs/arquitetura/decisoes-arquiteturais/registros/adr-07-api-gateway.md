# ADR-07 — API Gateway

> **Data:** Março 2026 · **Status:** ✅ Aceita

---

## Contexto

Com 3 microsserviços expondo APIs (Identity e Campanhas via HTTP, Worker sem HTTP), o cliente precisa de um ponto de entrada único que abstraia a topologia interna, centralize CORS e forneça roteamento.

## Decisão

Adotar **YARP (Yet Another Reverse Proxy)** como API Gateway no serviço `esperanca-gateway-api`:

| Aspecto | Configuração |
|---------|-------------|
| **Tecnologia** | YARP (Microsoft.ReverseProxy) — reverse proxy nativo .NET |
| **Roteamento** | Baseado em prefixo de path |
| **CORS** | Configurado no gateway (origin do front-end) |
| **Health Checks** | Agregado — verifica health de identity e campanhas |
| **Rate Limiting** | ASP.NET Core Rate Limiting middleware (fixed window) |
| **Autenticação** | Pass-through — repassa header Authorization; validação acontece no serviço de destino |

### Tabela de Rotas

| Rota no Gateway | Serviço de Destino | Autenticação |
|-----------------|--------------------|-------------|
| `/api/identity/**` | esperanca-identity-api | Varia por endpoint |
| `/api/campanhas/**` | esperanca-campanhas-api | GestorONG (maioria) |
| `/api/doacoes/**` | esperanca-campanhas-api | Doador |
| `/api/transparencia/**` | esperanca-campanhas-api | Público |
| `/health` | Gateway (agregado) | Público |

## Justificativa

1. **YARP** é mantido pela Microsoft, nativo do ecossistema .NET, com performance superior ao Ocelot. Configuração via `appsettings.json` ou código
2. **Ponto único de entrada:** simplifica configuração de DNS, TLS termination e CORS
3. **Pass-through de autenticação:** mantém a responsabilidade de autorização nos serviços (principle of least privilege)
4. **Rate limiting nativo:** `Microsoft.AspNetCore.RateLimiting` disponível desde .NET 7, sem pacotes externos

## Alternativas Consideradas

| Alternativa | Motivo da Rejeição |
|-------------|-------------------|
| **Ocelot** | Performance inferior; projeto com manutenção menos ativa; YARP é a recomendação oficial da Microsoft |
| **NGINX/Envoy** | Adiciona componente não-.NET ao stack; configuração em formato diferente; perda de type-safety |
| **Sem gateway** | Cliente precisaria conhecer múltiplos endereços; CORS em cada serviço; sem rate limiting centralizado |

## Consequências

- **Positivas:** stack 100% .NET; configuração declarativa; health check agregado; extensível com middleware customizado
- **Negativas:** mais um serviço para deployar e monitorar; single point of failure (mitigado com réplicas em K8s)
