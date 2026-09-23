1. Philosophy
   - Why lazy loading
   - Why OKF
   - Speed over quality (с оговоркой)

2. The bundle
   - What it is, where it lives
   - Lazy vs eager
   - OKF base + extensions

3. Per-file rationale
   Для каждого файла:
   - Purpose (что делает)
   - Boundaries (что НЕ делает)
   - Key sections
   - Common mistakes
   - Related files

4. Workflow
   - Issue lifecycle
   - Session lifecycle
   - CI failure → fix
   - Incident → runbook

5. Design decisions
   - Why _*.md prefix
   - Why WORK_LOG.md not committed
   - Why ADRs immutable
   - Why runbooks separate from troubleshooting
   - Why index.md and log.md as pointers

6. Non-goals
   - What we deliberately don't do

7. Anti-patterns
   - Frequent mistakes
   - How to avoid them

8. Evolution
   - How to add a new file
   - How to add a new type
   - Versioning scheme

=============

Part I — Foundations
  1. The problem: why agents drown in context
  2. What OKF is
  3. The bundle as a unit of knowledge
  4. Lazy loading

Part II — The files
  5. AGENTS.md — the entry point
  6. Onboarding: _setup, _concepts, _glossary
  7. Daily work: _templates, _worklog, _backlog, _decisions, _codestyle, _commands
  8. Diagnostics: _ci, _troubleshooting, runbooks/
  9. Navigation: _files, _env, _security, analysis/, _meta
  10. Utility: index.md, log.md, SPEC_REFERENCE.md, .gitignore, .template-version

Part III — Workflows
  11. Issue lifecycle
  12. Session lifecycle
  13. When CI fails
  14. Incident in prod
  15. Deep analysis
  16. Planning

Part IV — Operations
  17. init-opencode: install and update
  18. Extending the bundle
  19. Anti-patterns
  20. Philosophy

Part V — Appendices
  A. Type dictionary
  B. Full bundle structure
  C. OKF spec extract
  D. FAQ

=================

