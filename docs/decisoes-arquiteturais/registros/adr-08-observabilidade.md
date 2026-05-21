# ADR-08 — Observabilidade

> **Data:** Março 2026 · **Status:** ✅ Aceita

---

## Contexto

Uma arquitetura de microsserviços com comunicação assíncrona exige visibilidade sobre: latência de requests, throughput do Worker, tamanho de filas, erros e traces distribuídos. Os hot spots #7 (retry do Worker) e #8 (latência dos read models) reforçam essa necessidade.

## Decisão

Adotar uma estratégia de observabilidade em **3 pilares**:

### Logging — Serilog

| Aspecto | Decisão |
|---------|---------|
| **Biblioteca** | Serilog com sink para Console |
| **Formato** | Structured logging (JSON) com correlation ID propagado entre serviços |
| **Níveis** | Information (fluxo normal), Warning (retries), Error (falhas), Fatal (DLQ) |

### Métricas — Grafana

| Métrica | Serviço | Descrição |
|---------|---------|-----------|
| Request duration (p50, p95, p99) | identity-api, campanhas-api | Latência dos endpoints |
| Worker processing time | worker | Tempo de processamento por doação |
| Queue depth | RabbitMQ | Mensagens pendentes em `doacoes-recebidas` |
| DLQ count | RabbitMQ | Mensagens na DLQ (alerta se > 0) |
| Error rate | Todos | Taxa de erros 5xx por serviço |
| Active campaigns | campanhas-api | Campanhas em status EmAndamento |

### Tracing — OpenTelemetry (opcional no MVP)

| Aspecto | Decisão |
|---------|---------|
| **SDK** | OpenTelemetry .NET SDK |
| **Propagação** | W3C Trace Context entre HTTP e mensageria |

### Health Checks

| Serviço | Health Checks |
|---------|--------------|
| esperanca-identity-api | PostgreSQL (`identity_db`) |
| esperanca-campanhas-api | PostgreSQL (`campanhas_db`), MongoDB (`doacoes_db`), RabbitMQ |
| esperanca-worker | MongoDB (`doacoes_db`), RabbitMQ |
| esperanca-gateway-api | Agregado (identity-api + campanhas-api) |

## Justificativa

1. **Serilog** é o padrão de facto para structured logging em .NET — extensível, performático, sem dependência de cloud
2. **Grafana** oferece dashboards customizados locais, especialmente para métricas de RabbitMQ, sem custo adicional
3. **OpenTelemetry** garante portabilidade futura do tracing para qualquer exportador
4. **Health checks nativos** do ASP.NET Core permitem monitorar a saúde dos serviços localmente

## Consequências

- **Positivas:** visibilidade completa da plataforma sem dependência de cloud; alertas proativos (DLQ, error rate); debugging facilitado com correlation IDs
- **Negativas:** Grafana requer instância separada no Docker Compose; sem APM gerenciado fora do ambiente local
