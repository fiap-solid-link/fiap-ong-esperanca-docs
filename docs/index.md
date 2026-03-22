# Conexão Solidária

Documentação técnica do MVP da plataforma **Conexão Solidária** — 4 microsserviços .NET 10, RabbitMQ e Kubernetes — desenvolvido para a ONG Esperança Solidária no Hackathon 9NETT FIAP.

---

!!! warning "MVP em desenvolvimento ativo — prazo de 50 dias"
    Esta documentação cobre um MVP de hackathon. Decisões arquiteturais e interfaces podem evoluir durante o período de desenvolvimento. Consulte sempre a versão mais recente.

!!! danger "Dependência crítica: Issue #1"
    A **Issue #1 (Shared Contracts + Docker Compose Infra)** bloqueia **todas** as demais frentes de desenvolvimento. Nenhum microsserviço pode ser iniciado sem que ela esteja concluída.

!!! info "Estado atual do projeto"
    **9 issues** distribuídas entre **4 desenvolvedores**, cobrindo 4 microsserviços .NET 10. A integração ponta-a-ponta está prevista para a semana 4 (Issue #9 — todos os devs).

---

## Navegação Rápida

<div class="grid cards" markdown>

-   :material-information:{ .lg .middle } **Contexto do Projeto**

    ---

    Visão geral da plataforma, stack tecnológica, microsserviços e contexto do hackathon.

    [:octicons-arrow-right-24: Acessar](contexto.md)

-   :material-sitemap:{ .lg .middle } **Modelagem de Domínio**

    ---

    Event Storming, bounded contexts, agregados, 23 eventos de domínio e read models.

    [:octicons-arrow-right-24: Acessar](modelagem/index.md)

-   :material-layers-triple:{ .lg .middle } **Arquitetura**

    ---

    Visão macro da solução, microsserviços, diagramas e 10 Decisões Arquiteturais (ADRs).

    [:octicons-arrow-right-24: Acessar](arquitetura/index.md)

-   :material-checkbox-multiple-marked:{ .lg .middle } **Execução**

    ---

    Frentes de desenvolvimento, issues do GitHub Projects e cronograma de entrega.

    [:octicons-arrow-right-24: Acessar](execucao/index.md)

</div>