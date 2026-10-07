# 🏆 CupForge - Football & eSports Championship Platform

[![.NET 9](https://img.shields.io/badge/.NET-9.0-purple.svg)](https://dotnet.microsoft.com/)
[![React 19](https://img.shields.io/badge/React-19-blue.svg)](https://react.dev/)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.x-blue.svg)](https://www.typescriptlang.org/)
[![SQL Server](https://img.shields.io/badge/SQL%20Server-2022-red.svg)](https://www.microsoft.com/sql-server)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

Plataforma empresarial moderna para gestão de ligas, copas e campeonatos de **futebol real** (campo, society, futsal) e **futebol virtual / eSports** (EA Sports FC, eFootball, Pro Clubs).

O projeto é desenvolvido com padrões de engenharia de nível **Senior / Tech Lead**, aplicando **Clean Architecture**, **Domain-Driven Design (DDD)**, **CQRS**, **Event-Driven Architecture**, testes automatizados e observabilidade completa.

---

## 🎯 Destaques do Projeto

- **Regulamentos 100% Flexíveis:** O organizador tem total autonomia para parametrizar sistemas de pontuação, formatos de fases (grupos, turno/returno, cruzamentos e mata-matas com séries MD3/MD5), além de montar a ordem exata dos critérios de desempate (incluindo minitabela para empates com 3 ou mais equipes).
- **Modo Simplificado & Modo com Atletas:** Funciona tanto no modo ágil (apenas equipes e placares) quanto no modo avançado com controle de elencos, Gamertags, artilharia e suspensões automáticas.
- **Arquitetura Limpa (Clean Architecture):** Domínio isolado sem acoplamento a frameworks, com regras de negócio expressivas.
- **CQRS com MediatR:** Segregação estrita entre comandos de mutação de estado e consultas de leitura de alta performance.
- **Resiliência e Cache:** Consultas públicas de classificação e tabela com latência p95 < 100ms utilizando Redis.

---

## 🏛️ Arquitetura

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

## 📚 Documentação do Projeto

Toda a documentação viva da arquitetura, requisitos e decisões técnicas está disponível na pasta [`/docs`](docs/):

- [00-Roadmap do Projeto](docs/00-Roadmap.md)
- [01-Requisitos Funcionais e Não-Funcionais](docs/01-Requisitos.md)
- [02-Regras de Negócio & Regulamentos](docs/02-RegrasNegocio.md)
- [03-Arquitetura do Sistema & C4 Model](docs/03-Arquitetura.md)
- [04-Clean Architecture & Diretrizes](docs/04-CleanArchitecture.md)
- [05-Domain-Driven Design (DDD)](docs/05-DDD.md)
- [06-Modelagem de Banco de Dados (ERD)](docs/06-ModelagemBanco.md)
- [13-ADRs (Architecture Decision Records)](docs/13-ADRs/)

---

## 🚀 Como Executar o Projeto Localmente

*(Instruções de execução e Docker Compose em desenvolvimento na Fase 2)*

---

## 👤 Autor

**Felipe Batista de Assis**  
- GitHub: [@FelipeCJSEP](https://github.com/FelipeCJSEP)
