# Modelagem de Domínio

Documentação da modelagem de domínio da plataforma **Conexão Solidária**, cobrindo a descoberta do domínio através de técnicas colaborativas como Event Storming e Domain Storytelling.

---

## Bounded Contexts

A modelagem identificou **4 bounded contexts** com responsabilidades bem delimitadas:

| Bounded Context | Responsabilidade | Serviço |
|-----------------|-----------------|---------|
| **Identidade** | Registro, autenticação, gestão de perfis (RBAC) | `fiap-ong-esperanca-identity-api` |
| **Campanhas** | Ciclo de vida de campanhas, scheduler de vencimento | `fiap-ong-esperanca-campanhas-api` |
| **Doações** | Processamento assíncrono de doações, retry, DLQ | `fiap-ong-esperanca-doacao-worker` |
| **Transparência** | Painel público de leitura via read models | `fiap-ong-esperanca-campanhas-api` (read side) |

---

## Agregados

| Agregado | Bounded Context | Invariantes-chave |
|----------|-----------------|-------------------|
| **Usuario** | Identidade | Email único; CPF válido; Admin via seed; GestorONG acumula perfil Doador |
| **Campanha** | Campanhas | Transições unidirecionais de status; edição só em `Cadastrada`; prorrogação só em `EmAndamento` |
| **Doacao** | Doações | Doações que passaram pela validação da API são sempre processadas; idempotência via `IdempotencyKey` |

---

## Seções

<div class="grid cards" markdown>

-   :material-timeline-clock:{ .lg .middle } **Event Storming**

    ---

    23 eventos de domínio, 3 agregados, 4 swimlanes, eventos pivotais, hot spots, comandos e políticas.

    [:octicons-arrow-right-24: Acessar](event-storming.md)

-   :material-book-story:{ .lg .middle } **Domain Storytelling**

    ---

    Narrativas de domínio que descrevem os fluxos do ponto de vista dos atores.

    [:octicons-arrow-right-24: Acessar](domain-storytelling.md)

</div>
