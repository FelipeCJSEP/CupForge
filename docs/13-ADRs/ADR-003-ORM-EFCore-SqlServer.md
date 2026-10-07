# ADR-003: Persistência com SQL Server 2022 e Entity Framework Core 9

- **Status:** Aprovado
- **Data:** 2026-10-07
- **Decisores:** Felipe Batista de Assis
- **Contexto:**
  O CupForge exige confiabilidade transacional estrita (ACID) para garantir que cálculos de pontos, saldos e registros de partidas nunca fiquem em estado inconsistente. Além disso, a flexibilidade de regulamento requer capacidade de persistir configurações dinâmicas de torneio sem necessidade de migrações estruturais contínuas.

- **Decisão:**
  Adotar **Microsoft SQL Server 2022** como banco de dados relacional principal gerenciado pelo **Entity Framework Core 9**:
  - Mapeamento explícito com `IEntityTypeConfiguration<T>` isolando anotações do domínio.
  - Colunas de configuração dinâmica armazenadas em JSON nativo no SQL Server mapeadas para Value Objects imutáveis no C#.
  - **Global Query Filters** configurados no `DbContext` para garantir Soft Delete transparente (`IsDeleted == false`).
  - **Interceptors do EF Core** para auditoria automática de timestamps UTC e despacho de Domain Events pós-transação.
  - Para testes de integração, utilização de **Testcontainers** executando o contêiner oficial do SQL Server para máxima fidelidade.

- **Consequências:**
  - **Positivas:**
    - Consistência referencial e suporte pleno a transações distribuídas e locais.
    - Suporte avançado do EF Core 9 a consultas complexas compiladas e migrações seguras.
    - Testes de integração 100% idênticos ao ambiente de produção graças aos Testcontainers.
  - **Trade-offs / Mitigações:**
    - EF Core pode gerar overhead se usado incorretamente para relatórios pesados. Para leituras analíticas complexas, o uso de `AsNoTracking()` e projeções explícitas mitiga qualquer perda de performance.
