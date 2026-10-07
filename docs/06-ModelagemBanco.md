# 🗄️ Modelagem de Banco de Dados (ERD) - CupForge

**Versão:** 1.0  
**Data:** Outubro de 2026  
**Banco Alvo:** Microsoft SQL Server 2022  
**Status:** Aprovado  
**Autor:** Felipe Batista de Assis  

---

## 1. Visão Geral do Modelo de Dados

O modelo de dados do **CupForge** foi projetado para:
- Atender com fidelidade à modelagem DDD com entidades ricas;
- Garantir integridade referencial estrita e conformidade ACID;
- Oferecer suporte nativo a **Soft Delete** (`IsDeleted` + Query Filters no EF Core);
- Suportar **Auditoria Global** (`CreatedAtUtc`, `CreatedBy`, `LastModifiedAtUtc`, `LastModifiedBy`);
- Otimizar buscas e agregações de tabelas com **índices compostos e filtrados** (`Filtered Indexes`).

---

## 2. Diagrama Entidade-Relacionamento (ERD)

```mermaid
erDiagram
    ORGANIZATIONS ||--o{ COMPETITIONS : hosts
    ORGANIZATIONS ||--o{ PARTICIPANTS : registers
    
    COMPETITIONS ||--o{ EDITIONS : has
    EDITIONS ||--o{ STAGES : contains
    EDITIONS ||--o{ EDITION_PARTICIPANTS : enrolls
    EDITION_PARTICIPANTS }o--|| PARTICIPANTS : references
    
    STAGES ||--o{ GROUPS : divides
    STAGES ||--o{ MATCHES : schedules
    GROUPS ||--o{ MATCHES : organizes
    
    PARTICIPANTS ||--o{ MATCHES : "plays as home"
    PARTICIPANTS ||--o{ MATCHES : "plays as away"
    
    PARTICIPANTS ||--o{ ROSTER_MEMBERS : employs
    PLAYERS ||--o{ ROSTER_MEMBERS : belongs_to
    
    MATCHES ||--o{ MATCH_EVENTS : records
    PLAYERS ||--o{ MATCH_EVENTS : triggers

    ORGANIZATIONS {
        uniqueidentifier Id PK
        nvarchar_150 Name
        nvarchar_150 Slug UK
        bit IsDeleted
        datetime2 CreatedAtUtc
    }

    COMPETITIONS {
        uniqueidentifier Id PK
        uniqueidentifier OrganizationId FK
        nvarchar_150 Name
        nvarchar_150 Slug
        int Modality "Campo, Futsal, Society, Virtual1v1, VirtualProClubs"
        bit IsDeleted
    }

    EDITIONS {
        uniqueidentifier Id PK
        uniqueidentifier CompetitionId FK
        int Year
        nvarchar_100 Name "ex: 2026 / Edição 1"
        int Status "0=Draft, 1=InProgress, 2=Completed"
        bit IsPlayerTrackingEnabled "false=apenas clubes, true=com atletas"
        nvarchar_max RulesConfigurationJson "JSON com pontos, tiebreakers, disciplina"
        bit IsDeleted
    }

    STAGES {
        uniqueidentifier Id PK
        uniqueidentifier EditionId FK
        nvarchar_100 Name "ex: Fase de Grupos, Quartas de Final"
        int StageOrder
        int StageType "0=RoundRobin, 1=Knockout"
        int MatchFormat "0=SingleMatch, 1=TwoLegs, 2=BestOf3"
        bit IsDeleted
    }

    GROUPS {
        uniqueidentifier Id PK
        uniqueidentifier StageId FK
        nvarchar_50 Name "ex: Grupo A"
        bit IsDeleted
    }

    PARTICIPANTS {
        uniqueidentifier Id PK
        uniqueidentifier OrganizationId FK
        nvarchar_150 Name
        nvarchar_50 ShortName
        nvarchar_10 Acronym
        nvarchar_500 LogoUrl
        int ParticipantType "0=RealClub, 1=VirtualTeam"
        bit IsDeleted
    }

    EDITION_PARTICIPANTS {
        uniqueidentifier Id PK
        uniqueidentifier EditionId FK
        uniqueidentifier ParticipantId FK
        uniqueidentifier GroupId FK "Opcional"
        datetime2 EnrolledAtUtc
    }

    PLAYERS {
        uniqueidentifier Id PK
        nvarchar_150 FullName
        nvarchar_100 Nickname
        date BirthDate
        nvarchar_50 Nationality
        nvarchar_100 Gamertag "Para eSports"
        nvarchar_50 GamingPlatform "PSN, Xbox, EA_ID, PC"
        nvarchar_500 PhotoUrl
        bit IsDeleted
    }

    ROSTER_MEMBERS {
        uniqueidentifier Id PK
        uniqueidentifier ParticipantId FK
        uniqueidentifier PlayerId FK
        date JoinedAt
        date LeftAt "Nullable"
        bit IsActive
    }

    MATCHES {
        uniqueidentifier Id PK
        uniqueidentifier EditionId FK
        uniqueidentifier StageId FK
        uniqueidentifier GroupId FK "Nullable"
        uniqueidentifier HomeParticipantId FK
        uniqueidentifier AwayParticipantId FK
        int RoundNumber
        datetime2 ScheduledAtUtc
        nvarchar_150 VenueOrPlatform "Campo/Cidade ou Servidor/Console"
        int Status "0=Scheduled, 1=InProgress, 2=Finished, 3=Postponed, 4=Cancelled"
        int HomeGoals "Nullable"
        int AwayGoals "Nullable"
        int HomePenaltyGoals "Nullable"
        int AwayPenaltyGoals "Nullable"
        bit IsWalkover
        bit IsDeleted
    }

    MATCH_EVENTS {
        uniqueidentifier Id PK
        uniqueidentifier MatchId FK
        uniqueidentifier ParticipantId FK
        uniqueidentifier PlayerId FK "Nullable (se tracking ativo)"
        int Minute
        int EventType "0=Goal, 1=OwnGoal, 2=PenaltyGoal, 3=YellowCard, 4=RedCard, 5=BlueCard"
        nvarchar_250 Notes "Nullable"
    }
```

---

## 3. Estratégia de Armazenamento do Regulamento Flexível

Para garantir que o **CupForge** ofereça **100% de flexibilidade ao usuário** sem necessidade de alterar o esquema do banco a cada nova regra esportiva, o agregado `Edition` armazena as configurações de regulamento na coluna `RulesConfigurationJson` mapeada para um Value Object fortemente tipado no C#:

```json
{
  "points": {
    "win": 3,
    "draw": 1,
    "loss": 0,
    "walkoverWin": 3,
    "walkoverLoss": 0,
    "walkoverScoreHome": 3,
    "walkoverScoreAway": 0
  },
  "tieBreakerOrder": [
    "Points",
    "Wins",
    "GoalDifference",
    "GoalsScored",
    "HeadToHeadMiniTournament",
    "HeadToHeadGoalDifference",
    "FewestRedCards",
    "FewestYellowCards",
    "OfficialLottery"
  ],
  "matchRules": {
    "startersCount": 11,
    "maxSubstitutes": 12,
    "maxSubstitutions": 5,
    "maxSubstitutionWindows": 3,
    "allowReentry": false,
    "extraTimeSubstitutions": 1
  },
  "disciplinary": {
    "yellowCardsLimitForSuspension": 3,
    "redCardSuspensionMatches": 1,
    "resetYellowCardsAfterGroupStage": true
  }
}
```

> **Vantagem Técnica:** O SQL Server 2022 oferece funções nativas de alta performance para JSON (`JSON_VALUE`, `JSON_QUERY`), permitindo consultas indexadas sobre essas propriedades caso necessário, mantendo o domínio C# totalmente tipado com `System.Text.Json`.

---

## 4. Índices de Alta Performance Recomendados

Para atingir a meta do **RNF-004** (p95 < 100ms em consultas de partidas e tabelas):

1. **`IX_Matches_Edition_Stage_Status`**:
   `CREATE NONCLUSTERED INDEX IX_Matches_Edition_Stage ON Matches (EditionId, StageId, Status) INCLUDE (HomeParticipantId, AwayParticipantId, HomeGoals, AwayGoals, ScheduledAtUtc) WHERE IsDeleted = 0;`
2. **`IX_MatchEvents_MatchId`**:
   `CREATE NONCLUSTERED INDEX IX_MatchEvents_MatchId ON MatchEvents (MatchId) INCLUDE (ParticipantId, PlayerId, EventType, Minute);`
3. **`IX_EditionParticipants_Edition_Group`**:
   `CREATE NONCLUSTERED INDEX IX_EditionParticipants_Edition_Group ON EditionParticipants (EditionId, GroupId);`
