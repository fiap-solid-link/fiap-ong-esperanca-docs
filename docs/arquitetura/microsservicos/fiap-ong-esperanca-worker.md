# fiap-ong-esperanca-worker

**Repo:** [`fiap-ong-esperanca-worker`](https://github.com/fiap-solid-link/fiap-ong-esperanca-worker)  
**Tipo:** .NET Worker Service  
**Banco:** MongoDB (`doacoes_db`)  
**Bounded Context:** Doações

## Responsabilidades

- Consome `DoacaoRecebidaEvent` da fila `doacoes-recebidas` no RabbitMQ
- Valida integridade dos dados do evento
- Persiste a doação no MongoDB (com verificação de `IdempotencyKey`)
- Projeta/atualiza read models de transparência no MongoDB:
  - `painel_macro` — incrementa total geral, recalcula Top 3 doadores
  - `lista_campanhas` — atualiza valor arrecadado por campanha
  - `campanha_detalhe` — adiciona doação à lista anonimizada
- Publica `DoacaoProcessadaEvent` no RabbitMQ
- Retry com backoff exponencial (3 tentativas: 1s, 4s, 16s) + DLQ + evento `ProcessamentoDoacaoFalhou`

**Entidade de domínio:** `Doacao`

## Estrutura de Pastas

```
src/
├── Esperanca.Doacao.Worker/                   # Program.cs, BackgroundService, Consumers
├── Esperanca.Doacao.Worker.Domain/            # Entidade Doacao, interfaces
├── Esperanca.Doacao.Worker.Infrastructure/    # MongoDB repository, RabbitMQ consumer/publisher
tests/
├── Esperanca.Doacao.Worker.UnitTests/
├── Esperanca.Doacao.Worker.IntegrationTests/
```

## Configuração de Resiliência

| Parâmetro | Valor |
|-----------|-------|
| Retry count | 3 tentativas |
| Backoff | Exponencial — 1s, 4s, 16s |
| DLQ | `doacoes-recebidas-dlq` |
| Prefetch count | 1 (processamento sequencial) |
| ACK mode | Manual (após persistência + publicação) |

## Dependências Principais

```
.NET 10 · Microsoft.Extensions.Hosting · MongoDB.Driver · RabbitMQ.Client
Serilog + Sinks.ApplicationInsights · Microsoft.ApplicationInsights.WorkerService
```
