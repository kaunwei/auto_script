# Agent Rules & Treaties

This document defines the mandatory rules, operational guardrails, and conventions that all AI agents MUST strictly observe when executing commands, modifying code, or performing workflows in this repository.

---

## 1. Mandatory Command Execution & Git Guardrails

Whenever executing commands in this workspace, the agent MUST obey the following safety rules:
- **No Destructive Git Operations**: Under NO circumstances should the agent run destructive commands such as:
  - `git push` / `git push --force`
  - `git reset --hard`
  - `git clean -f` / `git clean -fd`
  - `git branch -D`
  - `git checkout .` / `git restore .`
  unless specifically and explicitly requested by the user.
- **Non-Destructive Workflows**: Prefer staged changes, explicit branch workflows, and safe rollbacks.

---

## 2. Engineering & Development Treaties

- **Test-Driven Development (TDD)**: Follow the Red-Green-Refactor cycle. Write failing tests before implementation, mock only at system boundaries, and ensure regression safety.
- **Codebase Design & Deep Modules**: Design deep modules with simple, narrow interfaces and clean architectural seams. Avoid shallow wrappers and leaky abstractions.
- **Code Review**: For non-trivial changes, conduct a two-axis review (Standards Compliance and Spec Fidelity) before completing the work.
- **Bug Diagnosis**: When troubleshooting bugs, follow disciplined diagnosis: reproduce with failing test -> minimize -> formulate hypothesis -> instrument -> fix -> verify regression test.

---

## 3. Domain Modeling & Architecture Treaties

- **Glossary Adherence**: Strictly use canonical terminology defined in [GLOSSARY.md](file:///home/kw/workspace/auto_script/GLOSSARY.md). Avoid creating confusing synonyms.
- **Architectural Decision Records (ADRs)**: Review [docs/adr/](file:///home/kw/workspace/auto_script/docs/adr) before introducing architectural changes. If a change contradicts an existing ADR, explicitly surface the conflict before proceeding.

---

## 4. Agent Skills & Workspace Configurations

### Issue Tracker
- GitHub Issues is the primary tracker (with local `.scratch/` markdown fallback). See [docs/agents/issue-tracker.md](file:///home/kw/workspace/auto_script/docs/agents/issue-tracker.md).

### Triage Labels
- Canonical 5-role triage system (`needs-triage`, `needs-info`, `ready-for-agent`, `ready-for-human`, `wontfix`). See [docs/agents/triage-labels.md](file:///home/kw/workspace/auto_script/docs/agents/triage-labels.md).

### Domain Docs
- Single-context domain layout (`GLOSSARY.md` and `docs/adr/`). See [docs/agents/domain.md](file:///home/kw/workspace/auto_script/docs/agents/domain.md).

### Customization Skills
- Custom skills are organized in [.agents/skills/](file:///home/kw/workspace/auto_script/.agents/skills) and registered via [.agents/skills.json](file:///home/kw/workspace/auto_script/.agents/skills.json).
