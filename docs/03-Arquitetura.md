# 🏛️ Arquitetura do Sistema - CupForge

**Versão:** 1.0  
**Data:** Outubro de 2026  
**Status:** Aprovado  
**Autor:** Felipe Batista de Assis  

---

## 1. Visão Geral e Princípios Arquiteturais

O **CupForge** foi projetado seguindo as melhores práticas de Engenharia de Software Moderna para aplicações corporativas com alta flexibilidade de domínio, manutenibilidade a longo prazo e desacoplamento.

### Princípios Chave:
- **Separação de Preocupações (SoC):** Cada camada possui responsabilidade estrita, com o Domínio no centro sem dependências externas.
- **Isolamento de Regras de Domínio:** Entidades ricas e Value Objects encapsulam o estado e a validação das regras esportivas.
- **CQRS (Command Query Responsibility Segregation):** Segregação limpa entre operações de escrita (Commands que alteram estado e disparam eventos) e operações de leitura (Queries focadas em performance e projeção de dados).
- **Event-Driven Interno:** Disparo de Domain Events para desacoplar efeitos colaterais (ex: término de uma partida dispara o recálculo assíncrono da tabela de classificação e artilharia).
- **Observabilidade Nativa:** Telemetria com OpenTelemetry, métricas de saúde (Health Checks) e logs estruturados em JSON via Serilog.

---

## 2. Modelo C4 - Diagramas de Arquitetura

### 2.1 Nível 1: Diagrama de Contexto de Sistema (Context Diagram)

O diagrama de contexto descreve como o **CupForge** interage com usuários humanos e sistemas externos.

```mermaid
flowchart TD
    subgraph Users ["Usuários do Sistema"]
        OrgAdmin["Organizador / Administrador<br/>(Cria ligas, regulamentos e lança placares)"]
        ClubRep["Representante de Equipe / Pro-Player<br/>(Inscreve equipes e gerencia elenco opcional)"]
        PublicUser["Torcedor / Espectador / Comunidade<br/>(Consulta tabelas, estatísticas e históricos)"]
    end

    subgraph CupForgeSystem ["Sistema CupForge"]
        CupForgeCore["Plataforma CupForge<br/>(Gerenciamento de Ligas, Partidas, Classificações e Históricos)"]
    end

    subgraph ExternalServices ["Serviços Externos"]
        AzureBlobStorage["Azure Blob Storage<br/>(Armazenamento de logos, escudos e fotos)"]
        MailService["Serviço de E-mail / Notificação<br/>(Recuperação de senha e convites)"]
    end

    OrgAdmin -->|Configura torneios, regras e resultados| CupForgeCore
    ClubRep -->|Inscreve elenco e consulta jogos| CupForgeCore
    PublicUser -->|Acessa tabelas, chaveamentos e artilharia| CupForgeCore

    CupForgeCore -->|Upload e recuperação de imagens| AzureBlobStorage
    CupForgeCore -->|Disparo de notificações transacionais| MailService
```

---

### 2.2 Nível 2: Diagrama de Contêineres (Container Diagram)

O diagrama de contêineres detalha os componentes de software de alto nível executáveis no ecossistema CupForge.

```mermaid
flowchart TD
    UserBrowser["Navegador do Usuário"]

    subgraph CupForgeInfrastructure ["Infraestrutura CupForge"]
        SPA["Frontend SPA (React 19 + TypeScript + Vite + TailwindCSS)<br/>Interface responsiva e dinâmica"]
        APIGateway["ASP.NET Core 9 Web API<br/>Clean Architecture + MediatR (CQRS) + JWT"]
        
        Cache["Redis Cache<br/>Cache de tabelas, jogos do dia e artilharia"]
        Database[("SQL Server 2022<br/>Base de dados relacional principal")]
    end

    subgraph ExternalCloud ["Nuvem / Serviços"]
        BlobStorage["Azure Blob Storage<br/>Logos, Escudos e Mídias"]
    end

    UserBrowser -->|HTTPS / JSON| SPA
    SPA -->|Chamadas RESTful / JSON / Bearer JWT| APIGateway

    APIGateway -->|Leitura e escrita relacional via EF Core| Database
    APIGateway -->|Consultas em cache e invalidação| Cache
    APIGateway -->|Upload de escudos e fotos| BlobStorage
```

---

## 3. Visão Geral da Pilha Tecnológica (Tech Stack)

| Componente | Tecnologia Escolhida | Justificativa Técnica |
| :--- | :--- | :--- |
| **Linguagem Backend** | C# 13 / .NET 9 | Plataforma de altíssima performance, recursos modernos (`nullables`, `records`, `primary constructors`). |
| **Framework Web** | ASP.NET Core 9 | Minimal APIs e Controllers com injeção de dependência nativa e middleware robusto. |
| **Mediação & CQRS** | MediatR | Desacoplamento entre endpoints HTTP e lógica de negócio de aplicação. |
| **ORM & Dados** | Entity Framework Core 9 | Mapeamento relacional avançado, migrações fortemente tipadas e suporte a Query Filters globais (Soft Delete). |
| **Banco de Dados** | Microsoft SQL Server 2022 | RDBMS robusto, ACID, com suporte pleno a índices filtrados e alta integridade referencial. |
| **Cache Distribuído** | Redis | Redução de latência para consultas frequentes de classificação pública (p95 < 100ms). |
| **Frontend** | React 19 + TypeScript + Vite | SPA rápido, tipagem estrita de ponta a ponta, componentização moderna com TailwindCSS e Radix/shadcn. |
| **Observabilidade** | Serilog + OpenTelemetry | Rastreabilidade distribuída, logs estruturados e métricas padronizadas. |
| **Containerização** | Docker & Docker Compose | Reprodutibilidade de ambiente local e prontidão para orquestração em nuvem (Azure Container Apps). |
| **Testes** | xUnit + FluentAssertions + Testcontainers | Testes de unidade sem dependências e testes de integração com instâncias reais de SQL Server em contêiner. |

---

## 4. Estratégia de Comunicação e Fluxo de Dados (CQRS)

```mermaid
sequenceDiagram
    autonumber
    actor User as Cliente / Frontend
    participant API as ASP.NET Core API
    participant Pipeline as MediatR Pipeline (Logging / Validation)
    participant Handler as Command / Query Handler
    participant Domain as Entidades de Domínio
    participant Repo as EF Core DbContext
    participant Cache as Redis Cache
    participant DB as SQL Server

    alt Fluxo de Leitura (Query)
        User->>API: GET /api/v1/competitions/{id}/standings
        API->>Pipeline: Send(GetStandingsQuery)
        Pipeline->>Handler: Executa Handler
        Handler->>Cache: Verifica Cache (Redis)
        alt Cache Miss
            Handler->>Repo: Query AsNoTracking()
            Repo->>DB: SELECT com índices otimizados
            DB-->>Repo: Dados brutos
            Repo-->>Handler: Projeção DTO
            Handler->>Cache: Salva em Cache
        end
        Handler-->>API: StandingsDto
        API-->>User: 200 OK (JSON)
    else Fluxo de Escrita (Command)
        User->>API: POST /api/v1/matches/{id}/result
        API->>Pipeline: Send(UpdateMatchResultCommand)
        Pipeline->>Pipeline: FluentValidation (Valida Payload)
        Pipeline->>Handler: Executa Command Handler
        Handler->>Repo: Carrega Agregado Match + Rules
        Repo->>DB: SELECT com Tracking
        DB-->>Repo: Entidade Match
        Handler->>Domain: match.RegisterResult(score)
        Domain-->>Domain: Valida invariantes & gera Domain Event (MatchFinishedEvent)
        Handler->>Repo: Salvar alterações (SaveChangesAsync)
        Repo->>DB: UPDATE / INSERT (Transação ACID)
        Handler->>Cache: Invalida chaves de classificação no Redis
        Handler-->>API: Result.Success()
        API-->>User: 200 OK ou 204 NoContent
    end
```
