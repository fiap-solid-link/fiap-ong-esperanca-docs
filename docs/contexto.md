# Contexto do Projeto

Visão geral da plataforma **Conexão Solidária**, suas tecnologias, microsserviços e o contexto do hackathon que originou o projeto.

---

## Sobre o Projeto

A **Conexão Solidária** é uma plataforma digital desenvolvida para a ONG Esperança Solidária com o objetivo de digitalizar e escalar campanhas de doação, garantindo transparência para doadores e eficiência operacional para gestores.

O MVP entrega:

- **Cadastro e autenticação** de doadores e gestores com RBAC (3 perfis: Admin, GestorONG, Doador)
- **Gestão de campanhas** com ciclo de vida completo (Cadastrada → EmAndamento → Concluída/Cancelada)
- **Processamento assíncrono de doações** via mensageria (RabbitMQ + Worker)
- **Painel de transparência público** com dados em tempo real

---

## Visão Geral da Solução

| Aspecto | Detalhe |
|---------|---------|
| **Stack** | .NET 10 (C#) — ASP.NET Core + Worker Service |
| **Arquitetura** | Microsserviços com Clean Architecture + Vertical Slice |
| **Mensageria** | RabbitMQ |
| **Bancos de dados** | PostgreSQL (identidade e campanhas) + MongoDB (doações e read models) |
| **API Gateway** | YARP (Yet Another Reverse Proxy) |
| **Observabilidade** | Application Insights + Grafana + Serilog |
| **Infra** | Docker Compose (dev) + Kubernetes AKS (produção) |
| **CI/CD** | GitHub Actions → Azure Container Registry → AKS |
| **Testes** | xUnit + Testcontainers |

---

## Microsserviços

| Serviço | Tipo | Bounded Context | Banco |
|---------|------|-----------------|-------|
| `fiap-ong-esperanca-identity-api` | ASP.NET Core Web API | Identidade e Acesso | PostgreSQL (`identity_db`) |
| `fiap-ong-esperanca-campanhas-api` | ASP.NET Core Web API | Campanhas + Transparência | PostgreSQL (`campanhas_db`) + MongoDB |
| `fiap-ong-esperanca-doacao-worker` | .NET Worker Service | Doações | MongoDB (`doacoes_db`) |
| `fiap-ong-esperanca-gateway-api` | API Gateway (YARP) | Roteamento | — |

---

## Contexto do Hackathon

| Item | Detalhe |
|------|---------|
| **Evento** | Hackathon 9NETT — FIAP |
| **Projeto** | ONG Esperança Solidária — Plataforma Digital |
| **Equipe** | 4 desenvolvedores |
| **Prazo** | 50 dias |
| **Entregáveis** | MVP funcional + documentação + vídeo demonstrativo |

---

!!! note "Documentação técnica detalhada"
    Para decisões arquiteturais, consulte os 10 ADRs em [Arquitetura → Decisões Arquiteturais](arquitetura/decisoes-arquiteturais/index.md).
    Para a modelagem de domínio completa (Event Storming), acesse [Modelagem → Event Storming](modelagem/event-storming.md).
