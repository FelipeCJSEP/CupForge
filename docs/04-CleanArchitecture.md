# 🧱 Clean Architecture - CupForge

**Versão:** 1.0  
**Data:** Outubro de 2026  
**Status:** Aprovado  
**Autor:** Felipe Batista de Assis  

---

## 1. Visão Geral das Camadas

A arquitetura do **CupForge** adota a **Clean Architecture** (Arquitetura Limpa) de Robert C. Martin (Uncle Bob), combinada com **Domain-Driven Design (DDD)**. 

O princípio fundamental é a **Regra de Dependência**: as dependências de código-fonte apontam estritamente para o centro, onde reside a camada de Domínio. Nenhuma camada interna conhece detalhes de implementação de camadas externas.

```
       +-------------------------------------------------------+
       |               CupForge.Api (Apresentação)             |
       +-------------------------------------------------------+
                                  |
                                  v
       +-------------------------------------------------------+
       |             CupForge.Infrastructure (Infra)           |
       +-------------------------------------------------------+
                                  |
                                  v
       +-------------------------------------------------------+
       |            CupForge.Application (Casos de Uso)         |
       +-------------------------------------------------------+
                                  |
                                  v
       +-------------------------------------------------------+
       |              CupForge.Domain (Núcleo Puro)            |
       +-------------------------------------------------------+
```

---

## 2. Estrutura da Solution .NET 9

A Solution é dividida em projetos com responsabilidades bem delimitadas:

```
CupForge.sln
│
├── src/
│   ├── CupForge.Domain/              # Regras de negócio puras, entidades, VOs, Domain Events
│   ├── CupForge.Application/         # Casos de uso (CQRS Commands/Queries), DTOs, Interfaces
│   ├── CupForge.Infrastructure/      # EF Core, SQL Server, Redis, Repositórios, Serviços Externos
│   ├── CupForge.Api/                 # Controllers, Minimal APIs, Middlewares, Filtros, Swagger
│   └── CupForge.SharedKernel/        # Tipos primitivos compartilhados, Result pattern, Base Entity
│
├── tests/
│   ├── CupForge.Domain.UnitTests/    # Testes unitários puros de entidades e regras esportivas
│   ├── CupForge.Application.UnitTests/# Testes de handlers, validações e casos de uso (Mocks)
│   ├── CupForge.Infrastructure.IntegrationTests/ # Testes de persistência com Testcontainers (SQL Server)
│   └── CupForge.Api.IntegrationTests/# Testes end-to-end de endpoints via WebApplicationFactory
│
└── web/
    └── cupforge-client/              # Frontend React 19 + TypeScript + Vite + TailwindCSS
```

---

## 3. Responsabilidades Detalhadas de Cada Camada

### 3.1 `CupForge.Domain` (Core Puro)
- **Não possui dependências** de nenhum outro projeto da Solution nem de pacotes externos de infraestrutura (como EF Core, ASP.NET ou bibliotecas de terceiros pesadas).
- **Contém:**
  - **Entidades Ricas:** `Competition`, `Edition`, `Stage`, `Group`, `Match`, `Club`, `Player`.
  - **Value Objects:** `Score`, `StandingCriteriaOrder`, `PointSystem`, `Gamertag`.
  - **Domain Events:** `MatchFinishedDomainEvent`, `ScoreRectifiedDomainEvent`, `StageCompletedDomainEvent`.
  - **Exceções de Domínio:** `DomainValidationException`, `IneligiblePlayerException`.
  - **Interfaces de Repositório (Contratos):** `ICompetitionRepository`, `IMatchRepository`, `IClubRepository`.

### 3.2 `CupForge.Application` (Orquestração e Casos de Uso)
- Depende exclusivamente de `CupForge.Domain` e `CupForge.SharedKernel`.
- Orquestra os fluxos de trabalho do sistema usando **CQRS** via `MediatR`.
- **Contém:**
  - **Commands e Handlers:** (Ex: `CreateCompetitionCommand`, `RecordMatchResultCommand`, `GenerateRoundRobinScheduleCommand`).
  - **Queries e Handlers:** (Ex: `GetCompetitionStandingsQuery`, `GetMatchDetailsQuery`, `ListPlayerHistoryQuery`).
  - **Pipeline Behaviors do MediatR:**
    - `ValidationBehavior`: Valida DTOs de entrada via **FluentValidation** antes de chegar aos handlers.
    - `LoggingBehavior`: Loga início, fim e tempo de execução de cada comando.
    - `PerformanceBehavior`: Emite warnings se um handler demorar mais de 500ms.
  - **Interfaces de Serviços de Aplicação:** `ICacheService`, `ICurrentUserService`, `IDateTimeProvider`.

### 3.3 `CupForge.Infrastructure` (Implementação de Recursos Externos)
- Depende de `CupForge.Application` e `CupForge.Domain`.
- Implementa todos os contratos e detalhes técnicos de acesso a banco de dados, rede e provedores de nuvem.
- **Contém:**
  - **Persistência Relacional:** `ApplicationDbContext` (EF Core 9) com mapeamentos `IEntityTypeConfiguration<T>`.
  - **Interceptors do EF Core:**
    - `AuditableEntityInterceptor`: Preenche automaticamente `CreatedAtUtc`, `LastModifiedAtUtc`, etc.
    - `DomainEventsDispatchInterceptor`: Dispara Domain Events no momento do `SaveChangesAsync`.
  - **Repositórios Concretos:** Implementações de `ICompetitionRepository`, `IMatchRepository`.
  - **Serviços de Cache:** Implementação de `ICacheService` utilizando `StackExchange.Redis`.
  - **Segurança e Identidade:** Integração com ASP.NET Identity Core, geração e validação de JWTs.

### 3.4 `CupForge.Api` (Ponto de Entrada HTTP)
- Depende de `CupForge.Application`, `CupForge.Infrastructure` (para injeção de dependência no `Program.cs`) e `CupForge.SharedKernel`.
- Recebe requisições HTTP, converte em Commands/Queries, despacha via MediatR e mapeia o resultado para códigos HTTP adequados.
- **Contém:**
  - Controllers ou Minimal APIs com versionamento (`/api/v1/...`).
  - Middlewares customizados:
    - `GlobalExceptionHandlerMiddleware`: Captura exceções e devolve `ProblemDetails` (RFC 7807).
    - `CorrelationIdMiddleware`: Adiciona e propaga o cabeçalho `X-Correlation-Id`.
  - Configuração de OpenAPI / Swagger com suporte a Bearer Auth.
  - Health Checks (`/health/live`, `/health/ready`).

### 3.5 `CupForge.SharedKernel`
- Biblioteca utilitária compartilhada por todas as camadas.
- Contém tipos fundamentais como `Result<T>` / `Error` (evitando uso indiscriminado de Exceptions para controle de fluxo) e definições base de `Entity<TId>` e `ValueObject`.

---

## 4. Fluxo de Tratamento de Erros: O Padrão `Result<T>`

No **CupForge**, não utilizamos Exceptions para regras de validação ou casos de negócio esperados (ex: "clube não encontrado" ou "placar inválido"). Em vez disso, adotamos o padrão **Result Object**:

```csharp
// Exemplo conceitual do Result Pattern
public class Result<TValue>
{
    public bool IsSuccess { get; }
    public bool IsFailure => !IsSuccess;
    public TValue Value { get; }
    public Error Error { get; }

    public static Result<TValue> Success(TValue value) => new(value);
    public static Result<TValue> Failure(Error error) => new(error);
}
```

Isso garante respostas previsíveis, alta performance (evita o custo de *stack trace allocation*) e código declarativo nos Handlers e Controllers.
