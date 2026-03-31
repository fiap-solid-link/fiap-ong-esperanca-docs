# identity-api

**Repo:** [`fiap-ong-esperanca-identity-api`](https://github.com/fiap-solid-link/fiap-ong-esperanca-identity-api)  
**Tipo:** ASP.NET Core Web API  
**Banco:** PostgreSQL (`identity_db`)  
**Bounded Context:** Identidade e Acesso

## Responsabilidades

- Registro de Doador (público)
- Registro/Seed de GestorONG
- Login (autenticação) → emissão de JWT (access 30min + refresh 7 dias)
- Renovação de tokens (refresh token rotation)
- Validação de token via shared key
- Gerenciamento de perfis/roles (RBAC): Admin, GestorONG, Doador

## Fluxo de Dependências

```mermaid
graph TD
    A[WebApi] --> B[Application]
    A --> D[Infrastructure]
    B --> C[Domain]
    D --> B

    style C fill:#e8f5e9,stroke:#2e7d32
    style B fill:#e3f2fd,stroke:#1565c0
    style D fill:#fff3e0,stroke:#e65100
    style A fill:#fce4ec,stroke:#c62828
```

- **Domain** → zero dependências externas (apenas C# puro)
- **Application** → referencia Domain; define interfaces (`IAppDbContext`, `IJwtService`, `IPasswordHasher`)
- **Infrastructure** → implementa interfaces da Application e Domain
- **WebApi** → orquestra tudo via Modules; controllers delegam ao MediatR

## Endpoints

### Autenticação

| Método | Rota | Acesso | Descrição |
|--------|------|--------|-----------|
| POST | `/api/auth/registrar` | Público | Registro de novo doador |
| POST | `/api/auth/login` | Público | Login → emissão de tokens |
| POST | `/api/auth/refresh` | Público | Renovação de tokens (refresh token rotation) |

### Usuários

| Método | Rota | Acesso | Descrição |
|--------|------|--------|-----------|
| GET | `/api/auth/me` | Autenticado | Perfil do usuário logado |
| PUT | `/api/usuarios/perfil` | Autenticado | Atualizar nome e apelido |
| POST | `/api/usuarios/{id}/conceder-gestor` | Admin | Conceder role GestorONG |
| DELETE | `/api/usuarios/{id}/revogar-gestor` | Admin | Revogar role GestorONG |

### Infraestrutura

| Método | Rota | Acesso | Descrição |
|--------|------|--------|-----------|
| GET | `/health` | Público | Health check (PostgreSQL) |

## Fluxos Principais

### Registro + Login + Acesso Autenticado

```mermaid
sequenceDiagram
    actor U as Usuário
    participant API as Identity API
    participant DB as PostgreSQL

    Note over U,DB: 1. Registro
    U->>API: POST /api/auth/registrar<br/>{ nome, email, senha }
    API->>API: Valida campos (FluentValidation)
    API->>DB: Verifica se email já existe
    API->>API: Hash da senha (BCrypt)
    API->>DB: Insere usuário + role Doador
    API-->>U: 201 { id, nome, email }

    Note over U,DB: 2. Login
    U->>API: POST /api/auth/login<br/>{ email, senha }
    API->>DB: Busca usuário por email
    API->>API: Verifica senha (BCrypt)
    API->>API: Gera accessToken (JWT HS256, 30min)
    API->>API: Gera refreshToken (64 bytes aleatórios)
    API->>DB: Persiste refreshToken (7 dias)
    API-->>U: 200 { accessToken, refreshToken, expiraEm }

    Note over U,DB: 3. Acesso autenticado
    U->>API: GET /api/auth/me<br/>Authorization: Bearer {accessToken}
    API->>API: Valida JWT (assinatura, issuer, audience, expiração)
    API->>DB: Busca usuário por ID do claim "sub"
    API-->>U: 200 { id, nome, email, apelido, roles }
```

### Refresh Token Rotation

```mermaid
sequenceDiagram
    actor U as Usuário
    participant API as Identity API
    participant DB as PostgreSQL

    Note over U,DB: Access token expirou (30min)
    U->>API: POST /api/auth/refresh<br/>{ refreshToken: "abc123..." }
    API->>DB: Busca refreshToken "abc123..."
    API->>API: Valida: existe? não expirou? não revogado?

    alt Token inválido ou expirado
        API-->>U: 401 { erro: "Refresh token inválido ou expirado." }
    else Token válido
        API->>DB: Revoga token antigo (RevogadoEm = now)
        API->>API: Gera novo accessToken (JWT)
        API->>API: Gera novo refreshToken
        API->>DB: Persiste novo refreshToken
        API-->>U: 200 { accessToken, refreshToken, expiraEm }
    end

    Note over U,DB: Cada refresh token só pode ser usado uma vez
```

### Gestão de Roles (Admin)

```mermaid
sequenceDiagram
    actor A as Admin
    participant API as Identity API
    participant DB as PostgreSQL

    Note over A,DB: Conceder GestorONG
    A->>API: POST /api/usuarios/{id}/conceder-gestor<br/>Authorization: Bearer {adminToken}
    API->>API: Valida role "Admin" no JWT
    API->>DB: Busca usuário por ID
    API->>API: Verifica se já possui GestorONG
    API->>DB: Adiciona role GestorONG ao usuário
    API-->>A: 200 { usuarioId, mensagem }

    Note over A,DB: Revogar GestorONG
    A->>API: DELETE /api/usuarios/{id}/revogar-gestor<br/>Authorization: Bearer {adminToken}
    API->>DB: Busca usuário por ID
    API->>API: Verifica se possui GestorONG
    API->>DB: Remove role GestorONG do usuário
    API-->>A: 200 { usuarioId, mensagem }
```

## Modelo de Domínio

```mermaid
erDiagram
    USUARIOS {
        uuid Id PK
        varchar(150) Nome
        varchar(256) Email UK
        varchar(256) SenhaHash
        varchar(100) Apelido
        timestamp CriadoEm
        timestamp AtualizadoEm
    }

    PERFIS {
        uuid Id PK
        int Tipo UK
        varchar(50) Nome
    }

    USUARIO_ROLES {
        uuid UsuariosId FK
        uuid RolesId FK
    }

    REFRESH_TOKENS {
        uuid Id PK
        varchar(512) Token UK
        uuid UsuarioId FK
        timestamp CriadoEm
        timestamp ExpiraEm
        timestamp RevogadoEm
    }

    USUARIOS ||--o{ REFRESH_TOKENS : "possui"
    USUARIOS }o--o{ PERFIS : "pertence a"
```

### Roles (Seed)

| Tipo | Nome | Permissões |
|------|------|------------|
| 1 | Admin | Gerenciar roles de usuários |
| 2 | GestorONG | Gerenciar campanhas (em outro microsserviço) |
| 3 | Doador | Realizar doações e atualizar perfil |

## Autenticação JWT

| Parâmetro | Valor |
|-----------|-------|
| Algoritmo | HMAC-SHA256 |
| Access Token | 30 minutos |
| Refresh Token | 7 dias (uso único) |
| Issuer | `esperanca-identity-api` |
| Audience | `esperanca-platform` |

**Claims do Access Token:**

| Claim | Descrição |
|-------|-----------|
| `sub` | ID do usuário (GUID) |
| `email` | Email do usuário |
| `jti` | ID único do token |
| `role` | Roles do usuário (pode ser múltiplo) |

!!! note "Nota sobre Auth"
    Os demais microsserviços (`campanhas-api`, etc.) validam o JWT usando a mesma chave simétrica (shared secret via ConfigMap/Secret do K8s). Não realizam chamadas ao identity-api em runtime.

## Validações

Todas as mensagens de erro são internacionalizadas (pt-BR e en) via `IAppLocalizer` + embedded JSON resources.

### Registro

| Campo | Regra | Código |
|-------|-------|--------|
| Nome | Obrigatório | `Identity:100` |
| Nome | Máximo 150 caracteres | `Identity:101` |
| Email | Obrigatório | `Identity:102` |
| Email | Formato válido | `Identity:103` |
| Senha | Obrigatória | `Identity:104` |
| Senha | Mínimo 8 caracteres | `Identity:105` |

### Login

| Campo | Regra | Código |
|-------|-------|--------|
| Email | Obrigatório | `Identity:102` |
| Email | Formato válido | `Identity:103` |
| Senha | Obrigatória | `Identity:104` |

### Atualizar Perfil

| Campo | Regra | Código |
|-------|-------|--------|
| Nome | Obrigatório | `Identity:100` |
| Nome | Máximo 150 caracteres | `Identity:101` |
| Apelido | Máximo 100 caracteres | `Identity:106` |

## Seed Automático

Na inicialização, `DatabaseSeed.SeedAdminAsync()` executa automaticamente:

1. Aplica migrations pendentes
2. Verifica se o Admin já existe
3. Se não existir, cria:
    - **Email:** `admin@esperanca.org`
    - **Senha:** `Admin@123`
    - **Role:** Admin

## Códigos de Erro

| Código | Constante | Mensagem (pt-BR) |
|--------|-----------|-------------------|
| `Identity:001` | EmailOuSenhaInvalidos | Email ou senha inválidos. |
| `Identity:002` | TokenInvalido | Token inválido. |
| `Identity:003` | RefreshTokenInvalidoOuExpirado | Refresh token inválido ou expirado. |
| `Identity:004` | EmailJaCadastrado | Email já cadastrado. |
| `Identity:005` | UsuarioNaoEncontrado | Usuário não encontrado. |
| `Identity:006` | UsuarioJaPossuiGestor | Usuário já possui o perfil GestorONG. |
| `Identity:007` | UsuarioNaoPossuiGestor | Usuário não possui o perfil GestorONG. |
| `Identity:008` | RoleGestorNaoEncontrada | Role GestorONG não encontrada. |
| `Identity:900` | ErroDeValidacao | Erro de validação. |

## Dependências Principais

```
.NET 10 · ASP.NET Core · EF Core + Npgsql
MediatR · FluentValidation · BCrypt.Net-Next
Microsoft.AspNetCore.Authentication.JwtBearer
Serilog · Swashbuckle
```
