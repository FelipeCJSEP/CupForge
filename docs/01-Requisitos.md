# 📋 Requisitos do Sistema - CupForge

**Versão:** 1.0  
**Data:** Outubro de 2026  
**Status:** Em Revisão  
**Autor:** Felipe Batista de Assis  

---

## 1. Visão Geral do Produto

O **CupForge** é uma plataforma SaaS empresarial voltada para o gerenciamento integral de ligas, copas e campeonatos de **futebol real** (profissional, amador, base e society) e **futebol virtual / eSports** (torneios de EA Sports FC, eFootball, etc.). 

O sistema permite que federações, organizadores de torneios presenciais ou de ligas virtuais/comunidades gamer configurem seus regulamentos com total flexibilidade, realizem chaveamentos, acompanhem o calendário de partidas, registrem resultados/estatísticas e mantenham o histórico consolidado de equipes e competidores.

---

## 2. Personas do Sistema

| Persona | Papel | Responsabilidades Principais |
| :--- | :--- | :--- |
| **Organizador / Administrador** | Administrador da Plataforma | Cria torneios (reais ou virtuais), parametriza regulamentos, aprova inscrições e lança resultados. |
| **Gestor de Clube / Pro-Player / Capitão** | Representante de Equipe / Jogador | Inscreve sua equipe ou participa individualmente (1v1 ou Pro Clubs), gerencia jogadores/Gamertags. |
| **Jornalista / Torcedor / Espectador** | Visitante / Consulta | Visualiza tabelas, estatísticas, artilharia, histórico de confrontos e chaveamentos em tempo real. |
| **Operador de Sistema (DevOps/SecOps)** | Manutenção & Suporte | Monitora telemetria, integridade de dados, logs de auditoria e saúde da infraestrutura. |

---

## 3. Requisitos Funcionais (RF)

### 3.1 Módulo: Gestão Organizacional e Cadastros Base
- **RF-001**: O sistema deve permitir o cadastro e gerenciamento de Organizações / Ligas esportivas e de eSports.
- **RF-002**: O sistema deve suportar modalidades de competição:
  - **Futebol Real:** Campo, Society, Futsal.
  - **Futebol Virtual / eSports:** EA Sports FC, eFootball, etc., nos formatos 1v1 ou Equipes (ex: Pro Clubs 11v11).
- **RF-003**: O sistema deve permitir o cadastro de Clubes/Equipes/Clãs (nome oficial, sigla, logo/escudo, tipo: Real ou Virtual).
- **RF-004**: O sistema deve permitir o cadastro de Jogadores / Atletas / Pro-Players:
  - Dados gerais (nome, apelido/nome esportivo, foto, nacionalidade).
  - Para futebol virtual: Gamertag / PSN ID / EA ID / Discord handle.

### 3.2 Módulo: Gestão de Elencos, Inscrições e Histórico (Opcional por Competição)
- **RF-005**: O sistema deve permitir que torneios operem em **modo simplificado**, gerindo unicamente confrontos e placares entre equipes, sem obrigatoriedade de cadastrar ou relacionar atletas.
- **RF-006**: Para competições com gestão de atletas habilitada, o sistema deve permitir parametrizar a quantidade de titulares (padrão: 11, mas ajustável livremente para 5 no futsal, 7 no society, 3 no x3, etc.) e reservas.
- **RF-007**: O sistema deve permitir a inscrição de atletas/pro-players por edição de competição e vinculá-los ao elenco do clube/clã.
- **RF-008**: O sistema deve manter o histórico de passagens do atleta pelas equipes e seus títulos/participações por edição.

### 3.3 Módulo: Motor de Competições
- **RF-009**: O sistema deve suportar a criação de edições de competições nos formatos:
  - Pontos corridos (Turno único ou Turno e Returno).
  - Mata-mata tradicional (Jogos únicos, Ida e Volta ou Séries MD3/MD5 para eSports).
  - Misto (Fase de Grupos seguida de Mata-mata/Playoffs).
- **RF-010**: O sistema deve possuir algoritmo de geração automática de rodadas (Round Robin) garantindo equilíbrio de mandos de campo.
- **RF-011**: O sistema deve permitir o sorteio e chaveamento de fases eliminatórias com ou sem restrições de cruzamento.

### 3.4 Módulo: Gestão de Partidas e Resultados
- **RF-012**: O sistema deve permitir o agendamento de partidas com data, horário, rodada/fase e campo textual opcional de local/servidor/plataforma.
- **RF-013**: O sistema deve permitir registrar o placar da partida (gols mandante x gols visitante).
- **RF-014**: O sistema deve permitir registrar eventos da partida de forma opcional (autores dos gols, cartões).
- **RF-015**: O sistema deve permitir alterar o status da partida (Agendada, Em Andamento, Encerrada, Adiada, Cancelada).

### 3.5 Módulo: Classificação, Disciplina e Estatísticas
- **RF-016**: O sistema deve recalcular automaticamente a tabela de classificação ao encerrar ou atualizar o placar de uma partida.
- **RF-017**: O sistema deve aplicar os critérios de desempate na ordem configurada pelo usuário.
- **RF-018**: O sistema deve contabilizar cartões e emitir alertas de suspensão quando habilitado pelo regulamento.
- **RF-019**: O sistema deve gerar rankings de artilheiros e estatísticas de vitórias, derrotas e gols por clube/competidor.

### 3.6 Módulo: Relatórios e Exportações
- **RF-020**: O sistema deve permitir a exportação de tabelas de jogos e classificação nos formatos Excel (XLSX) e PDF.
- **RF-021**: O sistema deve gerar relatórios históricos de edições e campeões.

---

## 4. Requisitos Não-Funcionais (RNF)

### 4.1 Arquitetura e Engenharia de Código
- **RNF-001 (Padrão Arquitetural):** O backend deve seguir os princípios de **Clean Architecture** e **Domain Driven Design (DDD)**, isolando regras de negócio em entidades ricas e Value Objects.
- **RNF-002 (Segregação de Comandos e Consultas):** A aplicação deve implementar **CQRS** utilizando MediatR para desacoplamento de handlers de leitura e escrita.
- **RNF-003 (Código Limpo e Moderno):** A solução deve ser desenvolvida em **.NET 9**, C# 13, com `Nullable` ativado e configuração de compilação sem alertas (`TreatWarningsAsErrors = true`).

### 4.2 Performance e Escalabilidade
- **RNF-004 (Tempo de Resposta de API):** 95% das requisições de leitura de tabelas e jogos devem responder em menos de **100ms** (p95 < 100ms) sob carga de até 500 RPS.
- **RNF-005 (Cache):** Deve haver camada de cache distribuído em memória (Redis) para tabelas de classificação e listas de partidas ativas, com invalidação guiada por eventos de domínio.
- **RNF-006 (Consistência Eventual):** Recálculos pesados de estatísticas pós-jogo devem ser processados de forma assíncrona desacoplada por eventos de domínio.

### 4.3 Segurança e Autenticação
- **RNF-007 (Autenticação e Autorização):** Autenticação stateless via **JWT** (JSON Web Token) acompanhado de **Refresh Tokens rotativos** armazenados de forma segura.
- **RNF-008 (Controle de Acesso):** Políticas de autorização granulares baseadas em papéis e permissões (RBAC/PBAC) para garantir que um clube só altere dados autorizados.
- **RNF-009 (Proteção de Dados):** Senhas devem ser armazenadas utilizando algoritmos modernos de hashing (PBKDF2/Argon2 via ASP.NET Identity).
- **RNF-010 (Auditoria):** Todas as entidades mutáveis devem possuir trilha de auditoria completa (`CreatedBy`, `CreatedAtUtc`, `LastModifiedBy`, `LastModifiedAtUtc`, `IsDeleted`).

### 4.4 Qualidade, Testabilidade e Manutenibilidade
- **RNF-011 (Cobertura de Testes):** A cobertura de testes do núcleo de regras de domínio (`CupForge.Domain`) e casos de uso (`CupForge.Application`) deve ser **>= 85%**.
- **RNF-012 (Análise Estática):** A base de código deve manter classificação de **Quality Gate 'A'** no SonarQube, sem débitos técnicos graves ou vulnerabilidades de segurança.
- **RNF-013 (Resiliência de Integração):** Testes de integração devem utilizar instâncias reais de banco via **Testcontainers** para validação fidedigna de queries e transações.

### 4.5 Observabilidade
- **RNF-014 (Logs Estruturados):** Uso do Serilog gerando logs em formato JSON estruturado com CorrelationId para rastreamento ponta a ponta de requisições.
- **RNF-015 (Health Checks e Métricas):** Exposição de endpoints `/health` e `/metrics` seguindo os padrões do ecossistema .NET e OpenTelemetry.

---

## 5. Matriz de Rastreabilidade Preliminar

```
[Domínio / Negócio]              [Caso de Uso / Aplicação]           [Contratos da API]
Competições & Rodadas  ------->  GenerateRoundRobinMatchesCommand  -> POST /api/v1/competitions/{id}/schedule
Escalações & Validações ------>  SubmitMatchLineupCommand          -> POST /api/v1/matches/{id}/lineup
Súmula & Eventos de Jogo ----->  RegisterMatchEventCommand         -> POST /api/v1/matches/{id}/events
Homologação & Término -------->  FinalizeMatchCommand              -> POST /api/v1/matches/{id}/finalize
Classificação (Leitura) ------>  GetStandingsQuery                 -> GET  /api/v1/competitions/{id}/standings
```
