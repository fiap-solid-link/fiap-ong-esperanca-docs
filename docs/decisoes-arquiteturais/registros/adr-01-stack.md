# ADR-01 — Stack Tecnológica

> **Data:** Março 2026 · **Status:** ✅ Aceita

---

## Contexto

A plataforma Conexão Solidária requer uma stack capaz de suportar múltiplos microsserviços com comunicação síncrona (REST) e assíncrona (mensageria), processamento em background, autenticação JWT e deploy em Kubernetes. A equipe possui 4 pessoas e prazo de 50 dias.

## Decisão

Adotar **.NET 10 (C#)** como plataforma de desenvolvimento para todos os microsserviços:

| Componente | Tecnologia |
|------------|------------|
| Framework Web | ASP.NET Core 10 |
| Background Processing | .NET Worker Service |
| ORM | Entity Framework Core 10 |
| Mediator | MediatR |
| Serialização | System.Text.Json |
| Validação | FluentValidation |
| Logging | Serilog |
| Containerização | Docker (multi-stage build) |

## Justificativa

**Decisão inflexível:** a pós-graduação foca na stack .NET, então adotar .NET 10 alinha com o conhecimento técnico da equipe.

## Consequências

- **Positivas:** stack uniforme em todos os serviços; maior produtividade da equipe; ecossistema maduro e performático
- **Negativas:** dependência do ecossistema Microsoft; curva de aprendizado para quem não conhece .NET
