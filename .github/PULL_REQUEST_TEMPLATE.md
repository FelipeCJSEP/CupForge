## 📌 Descrição das Alterações

<!-- Resuma de forma clara e concisa o que este Pull Request introduz, corrige ou refatora. -->

Closes #<!-- Número da Issue associada, ex: #12 -->

---

## 🏷️ Tipo de Mudança

- [ ] ✨ `feat`: Nova funcionalidade ou caso de uso
- [ ] 🐛 `fix`: Correção de bug ou inconsistência de regra de negócio
- [ ] ♻️ `refactor`: Refatoração de código sem alteração no comportamento externo
- [ ] 🧪 `test`: Adição ou ajuste de testes automatizados (unitários/integração)
- [ ] 📝 `docs`: Alteração em documentações (`/docs`, ADRs ou README)
- [ ] 🚀 `perf`: Melhoria de performance ou otimização de queries/cache
- [ ] 🔧 `chore` / `ci`: Ajuste em pipelines, contêineres Docker ou dependências

---

## 🧱 Camadas Afetadas (Clean Architecture)

- [ ] `CupForge.Domain` (Entidades, Value Objects, Domain Events)
- [ ] `CupForge.Application` (Commands, Queries, Handlers, Validators)
- [ ] `CupForge.Infrastructure` (EF Core, Repositórios, Migrações, Cache)
- [ ] `CupForge.Api` (Controllers, Middlewares, Endpoints)
- [ ] `CupForge.SharedKernel` (Abstrações centrais)
- [ ] Frontend React (SPA)

---

## ✅ Checklist de Qualidade & Engenharia

- [ ] **Conventional Commits:** Todas as mensagens de commit seguem o padrão (ex: `feat(matches): ...`, `fix(standings): ...`).
- [ ] **Testes Automatizados:** Testes unitários ou de integração foram adicionados/atualizados e cobrem as alterações.
- [ ] **Compilação Limpa:** Projeto compila com **zero warnings** (`TreatWarningsAsErrors = true`).
- [ ] **Nullable Reference Types:** Sem violações ou supressões indevidas (`!`) de nullability.
- [ ] **Regras de Negócio e Invariantes:** Nenhuma regra de negócio vazou para fora da camada de domínio.
- [ ] **Documentação Atualizada:** Se aplicável, a documentação em `/docs` ou novas ADRs foram criadas.

---

## 🧪 Como Testar as Mudanças

<!-- Instruções passo a passo de como verificar este PR localmente ou via endpoint/testes. -->

1. Executar a suíte de testes: `dotnet test`
2. Executar localmente via Docker Compose: `docker compose up -d`
3. Chamar o endpoint / executar o comando: ...
