# ADR-002: Segregação de Responsabilidade com CQRS e MediatR

- **Status:** Aprovado
- **Data:** 2026-10-07
- **Decisores:** Felipe Batista de Assis
- **Contexto:**
  Operações de escrita no CupForge (como agendar jogos, registrar placares, recalcular posições e aplicar regras disciplinares) exigem rigorosas validações de invariantes de domínio. Em contrapartida, operações de leitura (como consultar a tabela de classificação, lista de jogos da rodada e histórico de confrontos) demandam alta velocidade, projeções enxutas e cacheamento com latência reduzida.

- **Decisão:**
  Adotar o padrão **CQRS** implementado via biblioteca **MediatR**:
  - **Commands (`IRequest<Result<T>>`):** Tratam mutações de estado, executam validações de domínio em entidades rastreadas e disparam Domain Events.
  - **Queries (`IRequest<Result<TResponse>>`):** Realizam consultas somente-leitura otimizadas com `AsNoTracking()`, projeções diretas em DTOs e integração transparente com Redis Cache.
  - **Pipeline Behaviors:** Centralização de validação de entrada (`ValidationBehavior` via FluentValidation), métricas de performance e logs estruturados.

- **Consequências:**
  - **Positivas:**
    - Controllers e Minimal APIs tornam-se extremamente finos e concisos (apenas despacham a requisição ao mediador).
    - Facilidade para adicionar novos casos de uso sem tocar em código existente (princípio Aberto/Fechado - OCP).
    - Otimização individual de queries de leitura sem comprometer a integridade dos modelos de escrita.
  - **Trade-offs / Mitigações:**
    - Leve aumento de arquivos para cada endpoint (Command, CommandHandler, Validator). O benefício de clareza e separação compensa amplamente.
