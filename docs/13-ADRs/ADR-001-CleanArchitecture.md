# ADR-001: Adoção de Clean Architecture com DDD

- **Status:** Aprovado
- **Data:** 2026-10-07
- **Decisores:** Felipe Batista de Assis
- **Contexto:**
  O CupForge é um sistema de nível empresarial voltado ao gerenciamento de competições com regras altamente customizáveis pelo usuário. É fundamental que as regras de negócio esportivas fiquem estritamente isoladas de frameworks web, bancos de dados e ferramentas de terceiros, garantindo testabilidade e longevidade do código.

- **Decisão:**
  Adotar **Clean Architecture** combinada com **Domain-Driven Design (DDD)**:
  1. `CupForge.Domain`: Camada pura sem dependência externa contendo entidades ricas, Value Objects, Domain Events e contratos de repositório.
  2. `CupForge.Application`: Orquestração de casos de uso via CQRS (MediatR), DTOs e validações (FluentValidation).
  3. `CupForge.Infrastructure`: Implementações de acesso a dados (EF Core 9, SQL Server), cache (Redis) e serviços externos.
  4. `CupForge.Api`: Ponto de entrada HTTP RESTful e documentação OpenAPI.
  5. `CupForge.SharedKernel`: Utilitários centrais como `Result<T>` e abstrações fundamentais.

- **Consequências:**
  - **Positivas:**
    - Testabilidade máxima das regras de negócio sem necessidade de banco em memória ou mocks pesados.
    - Baixo acoplamento e independência de provedores de infraestrutura.
    - Facilidade para evolução futura (ex: inclusão de apps mobile ou outros frontends).
  - **Trade-offs / Mitigações:**
    - Maior número inicial de projetos e classes mapeadoras (mitigado pelo uso de records do C# e projeções diretas em Queries).
