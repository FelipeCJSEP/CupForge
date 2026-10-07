# 🏆 CupForge - Football Championship Platform
## Roadmap do Projeto

**Autor:** Felipe Batista de Assis  
**Projeto:** CupForge  
**Stack Principal:** .NET 9 | ASP.NET Core | React | TypeScript | SQL Server | Docker | Azure | GitHub Actions  

---

# 🎯 Objetivo

Desenvolver uma plataforma profissional para gerenciamento de campeonatos de futebol, utilizando conceitos modernos de Engenharia de Software.

O objetivo deste projeto é servir como demonstração de conhecimento em:

- Arquitetura de Software
- Domain Driven Design (DDD)
- Clean Architecture
- CQRS (Command Query Responsibility Segregation)
- Event Driven Architecture
- Princípios SOLID e Design Patterns
- DevOps & CI/CD
- Segurança e Conformidade
- Escalabilidade e Performance
- Testes Automatizados (Unitários, Integração, Carga)
- Observabilidade (Logs Estruturados, Métricas, Tracing)

O projeto deverá ser desenvolvido como um sistema comercial real.

---

# 🗺️ Roadmap de Execução

## Fase 1: Planejamento & Engenharia de Requisitos
- [x] Definição de Naming e Identidade do Projeto (**CupForge**)
- [x] Levantamento de requisitos e Personas (`01-Requisitos.md`)
- [x] Regras de negócio, flexibilidade de regulamento e eSports (`02-RegrasNegocio.md`)
- [x] Arquitetura de Alto Nível & Modelo C4 (`03-Arquitetura.md`)
- [x] Detalhamento da Clean Architecture e Padrão Result (`04-CleanArchitecture.md`)
- [x] Modelagem de Domínio & DDD (`05-DDD.md`)
- [x] Modelagem de Banco Relacional (ERD SQL Server) (`06-ModelagemBanco.md`)
- [x] ADRs Iniciais (ADR-001 Clean Architecture, ADR-002 CQRS/MediatR, ADR-003 EF Core/SQL Server)

## Fase 2: Infraestrutura & DevOps Base
- [ ] Inicialização do Repositório Git e GitHub
- [ ] Branch Protection Rules & GitFlow / Trunk-Based Strategy
- [ ] Configuração de Conventional Commits e Commitlint
- [ ] Configuração de GitHub Projects, Milestones e Issues Templates
- [ ] GitHub Actions (CI inicial: Build, Test, Lint)
- [ ] Configuração do SonarQube / SonarCloud
- [ ] Dockerfiles & Docker Compose (SQL Server, Redis, Mailpit, API, Web)
- [ ] Configuração de Ambiente de Desenvolvimento Local

## Fase 3: Arquitetura Base (.NET 9 & React)
- [ ] Estrutura da Solution .NET 9
- [ ] Clean Architecture:
  - `CupForge.Domain`
  - `CupForge.Application`
  - `CupForge.Infrastructure`
  - `CupForge.Api`
  - `CupForge.SharedKernel`
- [ ] MediatR (CQRS) & FluentValidation
- [ ] Setup do Frontend SPA (React 19 + TypeScript + Vite + TailwindCSS / shadcn/ui)

## Fase 4: Segurança & Cross-Cutting Concerns
- [ ] ASP.NET Identity Core
- [ ] Autenticação JWT com Refresh Tokens seguros (HttpOnly Cookie / Storage)
- [ ] RBAC (Role-Based Access Control) & PBAC (Policy-Based Access Control)
- [ ] Auditoria de Entidades (`CreatedBy`, `CreatedAt`, `LastModifiedBy`, etc.)
- [ ] Soft Delete global via EF Core Query Filters
- [ ] Observabilidade: Serilog + OpenTelemetry (Prometheus / Grafana / Jaeger)

## Fase 5: Cadastros Fundamentais (Core Data)
- [ ] Organizações / Ligas
- [ ] Equipes / Clubes / Clãs
- [ ] Jogadores / Pro-Players
- [ ] Competições e Modalidades (Real ou Virtual/eSports)
- [ ] Temporadas / Edições

## Fase 6: Gestão de Elencos e Inscrições
- [ ] Vínculo Jogador-Clube (Elenco ativo do clube)
- [ ] Inscrição de Atletas por Edição de Competição
- [ ] Numeração de Camisa e Posição do Atleta no Elenco
- [ ] Histórico de Passagens do Jogador por Clubes
- [ ] Histórico de Participações por Edição e Títulos Conquistados

## Fase 7: Motor de Competições (Competition Engine)
- [ ] Formato Pontos Corridos (Turno e Returno com gerador de rodadas - Round Robin)
- [ ] Formato Mata-Mata (Chaveamento olímpico, sorteios, confrontos de ida e volta)
- [ ] Formato Misto (Fase de Grupos + Playoffs / Mata-mata)
- [ ] Algoritmo de geração automática de rodadas e equilíbrio de mandos de campo

## Fase 8: Registro de Partidas e Resultados
- [ ] Agendamento e Detalhes da Partida (Data, Hora, Local, Rodada/Fase)
- [ ] Lançamento de Placar / Resultado Final (Gols Mandante x Visitante)
- [ ] Registro de Eventos da Partida (Autores dos Gols e Minutagem)
- [ ] Registro Disciplinar (Cartões Amarelos e Vermelhos)
- [ ] Status da Partida (Agendada, Em Andamento, Encerrada, Adiada, Cancelada)

## Fase 9: Classificação & Estatísticas
- [ ] Cálculo Automático de Tabela de Classificação
- [ ] Critérios de Desempate Configuráveis (Vitórias, Saldo, Gols Pró, Confronto Direto, Fair Play)
- [ ] Controle de Suspensões Automáticas (Acúmulo de Cartões Amarelos, Vermelhos)
- [ ] Ranking de Artilharia e Estatísticas dos Clubes
- [ ] Atualização via Event-Driven (Domain Events)

## Fase 10: Painéis, Dashboards & Exportações
- [ ] Dashboard da Organização / Competição
- [ ] Painel do Clube (Visão de Jogos, Desempenho e Histórico)
- [ ] Exportação de Tabelas e Classificação (Excel via ClosedXML / PDF)
- [ ] Relatórios de Desempenho e Histórico de Campeões

## Fase 11: Suíte Completa de Testes
- [ ] Testes Unitários de Domínio (xUnit + FluentAssertions + Bogus)
- [ ] Testes de Aplicação / Use Cases (NSubstitute / Moq)
- [ ] Testes de Integração com WebApplicationFactory e Testcontainers (SQL Server real)
- [ ] Testes de Carga / Stress (k6)

## Fase 12: Deploy & Cloud (Azure)
- [ ] Infraestrutura como Código (Terraform ou Bicep)
- [ ] Deploy Azure App Service / Azure Container Apps
- [ ] Azure SQL Database
- [ ] Azure Blob Storage (Escudos, Fotos de Atletas)
- [ ] Pipeline de Deploy Contínuo (CD) com Staging e Produção

---

# 📊 Estrutura de Documentação (`/docs`)

```
docs/
├── 00-Roadmap.md
├── 01-Requisitos.md
├── 02-RegrasNegocio.md
├── 03-Arquitetura.md
├── 04-CleanArchitecture.md
├── 05-DDD.md
├── 06-ModelagemBanco.md
├── 07-DomainModel.md
├── 08-API.md
├── 09-Seguranca.md
├── 10-Testes.md
├── 11-DevOps.md
├── 12-PadroesProjeto.md
├── 13-ADRs/
│   ├── ADR-001-CleanArchitecture.md
│   ├── ADR-002-CQRS-MediatR.md
│   ├── ADR-003-ORM-EFCore.md
│   ├── ADR-004-Identity-JWT.md
│   └── ...
├── diagrams/
│   ├── C4/
│   ├── UML/
│   ├── ERD/
│   ├── Sequence/
│   └── Flowcharts/
└── backlog/
    ├── Epic-01.md
    └── ...
```
