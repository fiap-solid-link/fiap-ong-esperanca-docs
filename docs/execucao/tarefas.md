# Tarefas de Execução — Conexão Solidária

Divisão do trabalho em **frentes de desenvolvimento** para 4 devs, formatadas como GitHub Issues prontas para criação no GitHub Projects.

---

## Issue #1 — Shared Contracts + Docker Compose Infra

**Labels:** `setup`, `infra`, `blocking`  
**Responsável:** Dev 4  
**Prioridade:** 🔴 Crítica (bloqueia todas as outras frentes)

### Descrição

Preparar a infraestrutura de desenvolvimento local e os contratos de integração compartilhados entre os microsserviços. Esta é a primeira entrega e desbloqueia o trabalho de todos os devs.

### Entregáveis

- [ ] **Docker Compose de infraestrutura** com PostgreSQL 16, MongoDB 7 e RabbitMQ 3.13 (management plugin)
  - Script `init-db.sql` criando `identity_db` e `campanhas_db`
  - Health checks em todos os containers
  - Portas: PostgreSQL `5432`, MongoDB `27017`, RabbitMQ `5672`/`15672`
- [ ] **Contratos de eventos de integração** — records C# compartilhados:
  - `DoacaoRecebidaEvent` (IdDoacao, IdCampanha, IdDoador, Valor, DataIntencao, IdempotencyKey)
  - `DoacaoProcessadaEvent` (IdDoacao, IdCampanha, Valor, DataProcessamento)
  - Definir estratégia de compartilhamento (cópia direta nos repos ou NuGet package local)
- [ ] **Documentação** de como subir o ambiente local (`README.md`)
- [ ] **Topologia RabbitMQ** documentada: exchange `esperanca.doacoes` (direct), filas `doacoes-recebidas`, `doacoes-processadas`, DLQ via Dead Letter Exchange `esperanca.doacoes.dlx`

### Acceptance Criteria

- `docker compose up` sobe PostgreSQL, MongoDB e RabbitMQ sem erros
- Ambos os databases PostgreSQL existem e aceitam conexões
- RabbitMQ Management UI acessível em `localhost:15672`
- Contratos publicados/disponíveis para os demais devs

---

## Issue #2 — esperanca-identity-api

**Labels:** `microsserviço`, `identity`, `auth`  
**Responsável:** Dev 1  
**Prioridade:** 🔴 Crítica  
**Depende de:** #1

### Descrição

Implementar o microsserviço de identidade completo: registro de usuários, autenticação JWT, RBAC com 3 perfis (Admin, GestorONG, Doador) e gestão de perfis. Inclui scaffold Clean Architecture, todas as camadas (Domain → Application → Infrastructure → WebApi), Dockerfile e testes.

### Entregáveis

- [ ] **Repo** `fiap-ong-esperanca-identity-api` criado com Clean Architecture (4 projetos src + 2 projetos test)
- [ ] **Domain Layer**
  - Entidade `Usuario` com métodos de negócio (AdicionarPerfil, RemoverPerfil, AtualizarPerfil)
  - Value Objects `Email` e `Cpf` com validação
  - Enum `Perfil` (Doador, GestorONG, Admin)
  - Interfaces: `IUsuarioRepository`, `IPasswordHasher`, `IJwtService`
- [ ] **Application Layer** (Vertical Slice com MediatR)
  - `CadastrarUsuario`, `Autenticar`, `AtualizarPerfil`
  - `ConcederPerfilGestor`, `RevogarPerfilGestor`, `ObterPerfilUsuario`
  - Pipeline behaviors: ValidationBehavior + LoggingBehavior
- [ ] **Infrastructure Layer**
  - EF Core com PostgreSQL (`IdentityDbContext`), migrations, Fluent API (UNIQUE email/CPF)
  - `BcryptPasswordHasher` (BCrypt.Net-Next)
  - `JwtService` — emissão e validação HMAC-SHA256 via config
  - Seed de Admin (`admin@esperanca.org`, senha via env var)
- [ ] **WebApi Layer**
  - Controllers: `AuthController`, `UsuarioController`
  - JWT Authentication + `[Authorize(Roles)]` nos endpoints protegidos
  - Swagger, Health Check (PostgreSQL), ExceptionHandlingMiddleware
- [ ] **Dockerfile** multi-stage build
- [ ] **Testes unitários** (obrigatório)
  - Entidade Usuario: criação com perfil Doador automático, adicionar/remover perfil, perfil duplicado
  - Value Objects: Email (formatos válidos/inválidos), Cpf (dígitos verificadores)
  - Handlers: CadastrarUsuarioHandler, AutenticarHandler
- [ ] **Testes de integração** (obrigatório)
  - Repositório EF Core contra PostgreSQL real (Testcontainers)
  - Endpoints auth com WebApplicationFactory

### Acceptance Criteria

- Registrar doador → login → JWT retornado com claims corretas (`sub`, `email`, `roles`)
- Admin (seed) consegue conceder e revogar GestorONG
- Endpoints protegidos retornam 401 sem token e 403 sem role adequada
- **`dotnet test` passa**
- Health check `/health` respondendo

!!! note "Nota sobre JWT"
    A chave simétrica de assinatura é compartilhada via env var `Jwt__SecretKey`. O token deve conter as claims `sub` (userId), `email` e `roles` (array). Dev 2 precisa validar tokens com a mesma chave.

---

## Issue #3 — esperanca-campanhas-api — Domínio e CRUD

**Labels:** `microsserviço`, `campanhas`, `core`  
**Responsável:** Dev 2  
**Prioridade:** 🔴 Crítica  
**Depende de:** #1

### Descrição

Implementar o domínio de campanhas e os endpoints de CRUD com ciclo de vida completo (Cadastrada → EmAndamento → Concluida/Cancelada).

### Entregáveis

- [ ] **Repo** `fiap-ong-esperanca-campanhas-api` criado com Clean Architecture
- [ ] **Domain Layer**
  - Entidade `Campanha` com máquina de estados: Editar, Ativar, Prorrogar, Cancelar, ConcluirPorData, ConcluirPorMeta, AtualizarArrecadacao
  - Enums: `StatusCampanha` (Cadastrada, EmAndamento, Concluida, Cancelada), `ModoEncerramento` (PorData, PorMeta, PorDataOuMeta)
  - Invariantes: DataFim > agora, MetaFinanceira > 0, transições unidirecionais
  - Interface `ICampanhaRepository`
- [ ] **Application Layer** — use cases CRUD
  - `CriarCampanha`, `EditarCampanha`, `AtivarCampanha`, `ProrrogarCampanha`, `CancelarCampanha`
  - `ObterCampanha`, `ListarCampanhasGestor`
- [ ] **Infrastructure Layer** — EF Core + PostgreSQL (`CampanhasDbContext`)
- [ ] **WebApi Layer** — `CampanhaController` com todos os endpoints CRUD
- [ ] **Testes unitários** (obrigatório)
  - Entidade Campanha: 12+ cenários de transição de estado
  - Handlers: CriarCampanhaHandler, AtivarCampanhaHandler

### Acceptance Criteria

- GestorONG cria campanha (status Cadastrada)
- GestorONG edita campanha em Cadastrada; rejeita edição em EmAndamento
- GestorONG ativa, prorroga e cancela com validações corretas
- JWT do identity-api é validado corretamente
- **`dotnet test` passa**

---

## Issue #4 — esperanca-campanhas-api — Doações e Mensageria

**Labels:** `microsserviço`, `campanhas`, `mensageria`, `doações`  
**Responsável:** Dev 2  
**Prioridade:** 🔴 Crítica  
**Depende de:** #1, #3

### Descrição

Implementar o fluxo de entrada de doações: endpoint de intenção de doação, publicação de `DoacaoRecebidaEvent` no RabbitMQ, e consumer de `DoacaoProcessadaEvent` que atualiza a arrecadação e verifica encerramento por meta.

### Entregáveis

- [ ] **Application Layer** — `EnviarIntencaoDoacao`, interface `IDoacaoPublisher`
- [ ] **Infrastructure — Publisher** — `RabbitMqDoacaoPublisher` publicando na exchange `esperanca.doacoes`
- [ ] **Infrastructure — Consumer** — `DoacaoProcessadaConsumerService` (BackgroundService)
- [ ] **WebApi** — `DoacaoController` — `POST /api/doacoes` [Authorize(Roles = "Doador")]
- [ ] **Testes de contrato** (Dev 2 + Dev 3): compatibilidade bidirecional de `DoacaoRecebidaEvent` e `DoacaoProcessadaEvent`

### Acceptance Criteria

- Doador envia intenção → mensagem na fila `doacoes-recebidas`
- Doação rejeitada se campanha não está EmAndamento ou valor <= 0
- Consumer processa `DoacaoProcessadaEvent` → ValorArrecadado incrementado
- Campanha encerra automaticamente por meta nos modos PorMeta e PorDataOuMeta
- **`dotnet test` passa**

---

## Issue #5 — esperanca-campanhas-api — Transparência e Scheduler

**Labels:** `microsserviço`, `campanhas`, `transparência`, `scheduler`  
**Responsável:** Dev 2  
**Prioridade:** 🟡 Alta  
**Depende de:** #4

### Descrição

Implementar os endpoints públicos de transparência (read models no MongoDB) e o scheduler de vencimento de campanhas.

### Entregáveis

- [ ] **Application Layer — Transparência**
  - `ConsultarPainelMacro`, `ConsultarListaCampanhas`, `ConsultarDetalheCampanha`
  - Interface `ITransparenciaReadRepository`
- [ ] **Infrastructure — MongoDB Read** — `TransparenciaMongoRepository` lendo collections `painel_macro`, `lista_campanhas`, `campanha_detalhe`
- [ ] **Application Layer — Scheduler**
  - `VerificarVencimento` — campanhas EmAndamento com DataFim próxima (3 dias, configurável)
  - `EncerrarPorData` — campanhas com DataFim expirada e modo PorData ou PorDataOuMeta
- [ ] **Infrastructure — Scheduler** — `CampanhaSchedulerService` (BackgroundService, timer 1 min)
- [ ] **WebApi** — `TransparenciaController` — 3 endpoints públicos

### Acceptance Criteria

- Endpoints de transparência retornam dados corretos do MongoDB sem autenticação
- Scheduler encerra campanhas com DataFim expirada automaticamente
- Scheduler loga alerta para campanhas próximas do vencimento (3 dias)

---

## Issue #6 — esperanca-doacao-worker

**Labels:** `microsserviço`, `worker`, `mensageria`, `mongodb`  
**Responsável:** Dev 3  
**Prioridade:** 🔴 Crítica  
**Depende de:** #1

### Descrição

Implementar o Worker Service que consome doações da fila RabbitMQ, persiste no MongoDB, projeta read models de transparência e publica `DoacaoProcessadaEvent`. Inclui retry com backoff exponencial e DLQ.

### Entregáveis

- [ ] **Repo** `fiap-ong-esperanca-worker` criado
- [ ] **Domain Layer** — Entidade `Doacao`, Enum `StatusDoacao`, interfaces
- [ ] **Infrastructure — MongoDB** — `MongoDoacaoRepository` com verificação de `IdempotencyKey`
- [ ] **Infrastructure — RabbitMQ**
  - `DoacaoRecebidaConsumerService` (BackgroundService) — ACK manual, retry 3x com backoff exponencial
  - Envio para DLQ após esgotamento de retries
  - `RabbitMqDoacaoProcessadaPublisher`
- [ ] **Projeção de Read Models** — atualiza `painel_macro`, `lista_campanhas`, `campanha_detalhe`
- [ ] **Dockerfile** multi-stage build
- [ ] **Testes unitários e de integração** (obrigatório)

### Acceptance Criteria

- Worker consome → persiste no MongoDB → publica `DoacaoProcessadaEvent`
- Mensagem duplicada (mesmo IdempotencyKey) não duplica doação
- Após 3 falhas → mensagem na DLQ + log de `ProcessamentoDoacaoFalhou`
- Read models atualizados após cada doação processada
- Health check respondendo (MongoDB + RabbitMQ)
- **`dotnet test` passa**

---

## Issue #7 — esperanca-gateway-api (YARP)

**Labels:** `microsserviço`, `gateway`, `infra`  
**Responsável:** Dev 4  
**Prioridade:** 🟡 Alta  
**Depende de:** #2, #3

### Descrição

Implementar o API Gateway com YARP como ponto único de entrada, roteando requisições para identity-api e campanhas-api. Inclui CORS, rate limiting e health check agregado.

### Entregáveis

- [ ] **Repo** `fiap-ong-esperanca-gateway-api` criado
- [ ] **YARP configurado** via `appsettings.json` com as 4 rotas
- [ ] **CORS** configurado (origin do front-end)
- [ ] **Rate Limiting** — fixed window via ASP.NET Core middleware
- [ ] **Health Check agregado** (`/health`) — verifica identity-api e campanhas-api
- [ ] **Pass-through de autenticação**
- [ ] **Dockerfile** multi-stage build

### Acceptance Criteria

- Requisições via gateway chegam ao serviço correto
- CORS headers presentes nas respostas
- Health check agregado retorna status dos downstream
- Rate limiting funcional (retorna 429 ao exceder limite)

---

## Issue #8 — Docker Compose Full Stack

**Labels:** `infra`, `docker`, `integração`  
**Responsável:** Dev 4  
**Prioridade:** 🟡 Alta  
**Depende de:** #2, #3, #6, #7

### Descrição

Montar o Docker Compose final que sobe todos os 7 containers (3 infra + 4 serviços) com configurações de ambiente, health checks e dependências corretas.

### Entregáveis

- [ ] **Docker Compose** com todos os serviços: `postgres`, `mongodb`, `rabbitmq`, `identity-api`, `campanhas-api`, `worker`, `gateway`
- [ ] **Health checks** e `depends_on` com `condition: service_healthy`
- [ ] **Env vars** padronizadas: `Jwt__SecretKey` compartilhada, connection strings
- [ ] **README** com instruções de uso

### Acceptance Criteria

- `docker compose up` sobe todos os 7 containers sem erro
- Migrations EF Core rodam automaticamente ao iniciar identity-api e campanhas-api
- Gateway acessível em `localhost:5000`
- RabbitMQ Management UI em `localhost:15672`

---

## Issue #9 — Integração Ponta-a-Ponta

**Labels:** `integração`, `qa`, `smoke-test`  
**Responsável:** Todos  
**Prioridade:** 🟢 Média  
**Depende de:** #8

### Descrição

Validar o fluxo completo da plataforma com todos os serviços rodando via Docker Compose. Smoke test manual cobrindo o caminho crítico: registro → login → criar campanha → ativar → doar → worker processa → transparência atualizada.

### Entregáveis

- [ ] **Smoke test manual** (Postman ou curl) cobrindo:
  1. Registrar doador → obter JWT
  2. Login Admin (seed) → conceder GestorONG a um usuário
  3. Login GestorONG → criar campanha → ativar
  4. Login Doador → enviar doação
  5. Verificar Worker processou (logs + MongoDB)
  6. Consultar transparência (painel, lista, detalhe)
  7. Verificar arrecadação atualizada na campanha
  8. Testar encerramento por meta
  9. Testar encerramento por data
  10. Testar DLQ (simular falha no Worker)
- [ ] **Coleção Postman** exportada (opcional mas recomendado)
- [ ] **Bug fixes** identificados durante o smoke test

### Acceptance Criteria

- Fluxo completo funcional ponta-a-ponta via gateway
- JWT do identity-api aceito pelo campanhas-api
- Doação percorre: API → RabbitMQ → Worker → MongoDB → DoacaoProcessadaEvent → campanhas-api atualiza arrecadação
- Transparência reflete doações processadas
- DLQ funcional (mensagem preservada após falhas)
