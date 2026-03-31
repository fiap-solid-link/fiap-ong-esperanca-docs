# fiap-ong-esperanca-docs

Documentação centralizada da plataforma **Conexão Solidária** — ONG Esperança Solidária — Hackathon 9NETT FIAP.

MVP da plataforma digital com 4 microsserviços .NET 10, RabbitMQ, Kubernetes (AKS) e observabilidade.

---

## Estrutura

```
docs/
├── index.md                              # Visão Geral
├── contexto.md                           # Contexto do Projeto
├── modelagem/                            # Modelagem de Domínio
│   ├── index.md
│   ├── event-storming.md
│   └── domain-storytelling.md
├── arquitetura/                          # Arquitetura (visão geral, diagrama, stack)
│   └── index.md
├── microsservicos/                       # → seção Microsserviços (nível superior na nav)
│   ├── index.md                          # Index da seção
│   ├── fiap-ong-esperanca-gateway-api.md
│   ├── fiap-ong-esperanca-identity-api.md
│   ├── fiap-ong-esperanca-campanhas-api.md
│   └── fiap-ong-esperanca-worker.md
├── decisoes-arquiteturais/               # → seção Decisões Arquiteturais (nível superior na nav)
│   ├── index.md
│   └── registros/                        # 10 ADRs individuais
└── execucao/                             # Execução
    ├── index.md
    └── tarefas.md
```

---

## Desenvolvimento Local

```bash
pip install zensical
zensical serve
```

Acesse `http://localhost:8000`

### Via Docker Compose

```bash
docker compose up --build
```

Acesse `http://localhost:8002` sem precisar instalar o Zensical na máquina host. O diretório do projeto é montado como volume para permitir hot reload dos arquivos em `docs/`.
