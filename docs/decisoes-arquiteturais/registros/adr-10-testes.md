# ADR-10 — Estratégia de Testes

> **Data:** Março 2026 · **Status:** ✅ Aceita

---

## Contexto

Com 3 microsserviços, comunicação assíncrona e 23 eventos de domínio, a suíte de testes precisa cobrir desde regras de negócio isoladas até a integração entre serviços via mensageria.

## Decisão

Adotar a **pirâmide de testes** com 3 níveis:

### Nível 1 — Testes Unitários (Domain + Application + WebApi)

| Alvo | Exemplo | Framework |
|------|---------|-----------|
| Entidades de domínio | `Campanha.Ativar()` muda status para EmAndamento | xUnit |
| Value Objects | Validação de CPF, Email | xUnit |
| MediatR Handlers | `CriarCampanhaHandler` persiste campanha | xUnit + NSubstitute |
| Validators | `CriarCampanhaValidator` rejeita MetaFinanceira <= 0 | xUnit + FluentValidation.TestHelper |

**Cobertura esperada:** invariantes dos agregados (Usuario, Campanha, Doacao), transições de status, validações de comando.

### Nível 2 — Testes de Integração (Infrastructure + API)

| Alvo | Exemplo | Framework |
|------|---------|-----------|
| Repositórios (EF Core) | Persistir e recuperar Campanha no PostgreSQL | xUnit + Testcontainers |
| Repositórios (MongoDB) | Persistir doação e consultar read model | xUnit + Testcontainers |
| Endpoints HTTP | POST `/api/campanhas` retorna 201 | xUnit + WebApplicationFactory |
| Consumer RabbitMQ | Mensagem na fila é consumida e processada | xUnit + Testcontainers |

**Infraestrutura de teste:** [Testcontainers](https://dotnet.testcontainers.org/) para subir PostgreSQL, MongoDB e RabbitMQ em containers efêmeros durante os testes.

### Nível 3 — Testes de Contrato (Mensageria)

| Alvo | Exemplo |
|------|---------|
| Schema do `DoacaoRecebidaEvent` | Producer (campanhas-api) e Consumer (worker) concordam no formato |
| Schema do `DoacaoProcessadaEvent` | Producer (worker) e Consumer (campanhas-api) concordam no formato |

**Abordagem:** testes que serializam/deserializam os eventos de integração garantindo compatibilidade bidirecional.

### Regra Obrigatória

!!! warning "Testes são requisito de conclusão"
    Testes unitários e de integração **não são uma entrega separada**. São obrigatórios em cada serviço e fazem parte dos acceptance criteria de cada issue. **Nenhuma issue é considerada concluída sem `dotnet test` passando.**

## Justificativa

1. **Testes unitários no Domain** validam as invariantes dos agregados identificados no Event Storming
2. **Testcontainers** elimina mocks de banco de dados, testando queries e migrations reais contra PostgreSQL, MongoDB e RabbitMQ em containers
3. **Testes de contrato** são essenciais em comunicação assíncrona — se o schema de um evento mudar em um serviço e não no outro, a fila quebrará silenciosamente

## Consequências

- **Positivas:** confiança nas regras de domínio; integração real com bancos; contratos de mensageria versionados
- **Negativas:** testes de integração com Testcontainers são mais lentos (~10-30s de startup); requer Docker na máquina de CI
