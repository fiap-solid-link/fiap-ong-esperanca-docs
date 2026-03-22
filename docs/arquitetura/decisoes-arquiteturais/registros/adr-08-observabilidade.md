# ADR-08 — Observabilidade

> **Data:** Março 2026 · **Status:** ✅ Aceita

---

## Contexto

Uma arquitetura de microsserviços com comunicação assíncrona exige visibilidade sobre: latência de requests, throughput do Worker, tamanho de filas, erros e traces distribuídos. Os hot spots #7 (retry do Worker) e #8 (latência dos read models) reforçam essa necessidade.

## Decisão

Adotar uma estratégia de observabilidade em **3 pilares**:

### Logging — Serilog + Application Insights

| Aspecto | Decisão |
|---------|---------|
| **Biblioteca** | Serilog com sinks para Console (dev) e Application Insights (produção) |
| **Formato** | Structured logging (JSON) com correlation ID propagado entre serviços |
| **Níveis** | Information (fluxo normal), Warning (retries), Error (falhas), Fatal (DLQ) |

### Métricas — Application Insights + Grafana

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
| **Exportador** | Application Insights (via Azure Monitor Exporter) |
| **Propagação** | W3C Trace Context entre HTTP e mensageria |

### Health Checks

| Serviço | Health Checks |
|---------|--------------|
| esperanca-identity-api | PostgreSQL (`identity_db`) |
| esperanca-campanhas-api | PostgreSQL (`campanhas_db`), MongoDB (`doacoes_db`), RabbitMQ |
| esperanca-worker | MongoDB (`doacoes_db`), RabbitMQ |
| esperanca-gateway-api | Agregado (identity-api + campanhas-api) |

## Justificativa

1. **Serilog** é o padrão de facto para structured logging em .NET — extensível, performático, integra nativamente com Application Insights
2. **Application Insights** oferece APM completo (traces, métricas, logs) com mínima configuração para apps .NET no Azure
3. **Grafana** complementa com dashboards customizados, especialmente para métricas de RabbitMQ
4. **Health checks nativos** do ASP.NET Core integram com Kubernetes readiness/liveness probes

## Consequências

- **Positivas:** visibilidade completa da plataforma; alertas proativos (DLQ, error rate); debugging facilitado com correlation IDs
- **Negativas:** Application Insights tem custo proporcional ao volume de telemetria; Grafana requer instância separada
