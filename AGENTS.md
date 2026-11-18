# AcademyOS — Codex Project Instructions

## 1. Project Context
AcademyOS is a multi-tenant SaaS application for managing academies and training centers in Egypt.

Read the project README and IMPLEMENTATION_PLAN.md before starting a new task.

## 2. Token and Context Efficiency

- Work only on the current requested task.
- Do not re-scan the entire repository on every task.
- Do not repeatedly read files that have not changed and are not relevant.
- Inspect only the files directly related to the current task and their dependencies.
- Reuse the existing architecture, utilities, types, services, and components.
- Do not regenerate existing files or rewrite working modules without a clear technical reason.
- Avoid unnecessary explanations, repeated summaries, and large code dumps.
- Do not reproduce entire files in your response unless requested.
- Prefer small, focused changes over large rewrites.
- Do not implement future phases while working on the current phase.
- Do not add dependencies unless they provide clear value.
- Do not run expensive or broad test suites unnecessarily. Run targeted tests first, then broader relevant tests before completing a phase.

## 3. Task Execution Workflow

For every task:

1. Read the relevant project instructions and implementation plan.
2. Identify the specific files and modules involved.
3. Inspect only the necessary existing code.
4. Briefly state the implementation approach.
5. Implement the requested scope.
6. Run relevant tests and type checks.
7. Fix errors introduced by the changes.
8. Update IMPLEMENTATION_PLAN.md.
9. Report what changed, what was tested, and any remaining issues.

Do not claim that a test passed unless it was actually executed successfully.

## 4. Architecture Rules

- Preserve the modular monolith architecture.
- Keep controllers, services, repositories, validation, and frontend components separated.
- Keep business logic out of React components and route handlers.
- Use PostgreSQL as the primary database.
- Use version-controlled migrations.
- Never introduce SQLite as a fallback.
- Keep database access consistent with the existing chosen ORM/query approach.
- Never introduce a second ORM or database access library without approval.

## 5. Multi-Tenancy and Security

- Every tenant-owned record must be scoped to its authenticated tenant.
- Never trust tenant_id received from the frontend.
- Enforce tenant authorization in backend services and repositories.
- Do not weaken authentication, authorization, validation, or tenant isolation to save time.
- Add regression tests for any security-sensitive change.
- Never expose secrets, passwords, or sensitive information in logs.

## 6. Financial Integrity

- Store money as integer piastres.
- Never use floating-point arithmetic for financial calculations.
- Use database transactions for multi-step financial operations.
- Never silently delete or modify successful financial transactions.
- Preserve payment allocation and reversal history.
- Keep dashboard and report calculations consistent.
- Add tests for every change that affects balances, payments, installments, or cash flow.

## 7. Code Quality

- Use TypeScript strict mode.
- Follow existing naming conventions and project structure.
- Avoid duplicate business logic.
- Prefer reusable utilities and components.
- Avoid unnecessary abstractions and over-engineering.
- Do not introduce mock functionality in place of real backend behavior.
- Do not leave unfinished buttons or nonfunctional UI controls.

## 8. Scope Control

- Implement only the requested phase or task.
- If a requirement depends on a missing prerequisite, explain it before proceeding.
- If a task requires a database schema change, inspect the affected relationships and migration implications.
- Do not change unrelated modules.
- Ask before making major architectural changes.

## 9. Documentation

Keep these files updated when relevant:

- README.md: setup, environment, migrations, seeds, tests, and run instructions.
- IMPLEMENTATION_PLAN.md: phases, completed tasks, pending tasks, and blockers.
- .env.example: required environment variables without real secrets.

Do not create redundant documentation files without a clear reason.

## 10. Completion Report

At the end of each task, provide a concise report:

- Implemented:
- Files changed:
- Database/migration changes:
- Tests executed and results:
- Remaining issues:
- Next recommended task:

Do not repeat the entire project specification in every response.