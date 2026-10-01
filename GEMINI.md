# Agent Rules & Treaties (24-Hour Unattended Framework)

This document defines the mandatory rules, operational guardrails, and conventions that all AI agents MUST strictly observe when executing commands, modifying code, or performing workflows in this repository.

---

## 1. 24h Unattended Dual-Terminal Workflow

This repository operates on a **Dual-Terminal Architecture** separating interactive architectural planning from background unattended execution:

```
┌─────────────────────────────────────────────────────────────┐
│                 Terminal A: Interactive Architect           │
│  • Discuss architecture & requirements with user.           │
│  • Decompose goals into single-line atomic English tasks.   │
│  • Autonomously manages .worker.env (models, ext dirs).    │
│  • Inspect progress.log (top <=10 lines) for acceptance.    │
└──────────────────────────────┬──────────────────────────────┘
                               │ (FIFO Queue)
┌──────────────────────────────▼──────────────────────────────┐
│                 Terminal B: Unattended Background Worker    │
│  • Pure zero-argument execution (just run ./worker.sh).     │
│  • Auto-sources .worker.env if configured by Terminal A.    │
│  • Stateless CLI process (agy --model gemini-3.7-flash-low).│
│  • Follows .skills/universal-build-verify.md.               │
│  • Runs builds, unit tests, GUI smoke tests.                │
│  • Max 3 repair attempts -> auto rollback on block.         │
│  • Auto Git Commit & prepends <=10 lines to progress.log.   │
└─────────────────────────────────────────────────────────────┘
```

---

## 2. Strict Inter-AI English Protocol

To optimize token efficiency (50-70% savings) and ensure maximum instruction-following precision across weaker models:
- **Human-AI Discussions**: Traditional Chinese (繁體中文) is welcomed in Terminal A.
- **All Inter-AI Files & Artifacts**: MUST be written in **Strictly English**:
  - `tasks.txt`
  - `progress.log`
  - `worker.status`
  - `tasks.done`
  - Git Commit messages and code comments.

---

## 3. Atomic Task Format Guide (for Terminal A AI)

When Terminal A decomposes user requests into `tasks.txt`, each task MUST occupy exactly one line and adhere to this structured contract:

```text
TASK-XXX | TARGET: <file_paths> | ACTION: <precise implementation details> | VERIFY: <test command> | CONSTRAINTS: <scope limits>
```

### Examples:
```text
TASK-001 | TARGET: src/auth.py, tests/test_auth.py | ACTION: Implement hash_password(password: str) -> str using SHA-256 with salt | VERIFY: pytest tests/test_auth.py -k test_hash | CONSTRAINTS: Use standard library hashlib only
TASK-002 | TARGET: src/window.cpp, tests/test_window.cpp | ACTION: Initialize OpenGL window with 60fps timer | VERIFY: cmake --build build && timeout 2s ./build/test_window | CONSTRAINTS: Maintain C++17 compatibility, headless safe
```

---

## 4. Verification & Safe Git Log Rules (Zero Context Pollution)

To keep Terminal A's context window clean and avoid context exhaustion:
1. **Primary Acceptance**: Terminal A only reads the top 10 lines of `progress.log` (`head -n 15 progress.log`).
2. **Safe Git Log Whitelist (If deeper check is needed)**:
   - `git log -n 1 --stat`
   - `git show --stat <commit-hash>`
   - `git log --oneline -n 5`
3. **STRICTLY FORBIDDEN in Terminal A**:
   - ❌ Bare `git log` (dumps unbounded commit history).
   - ❌ `git log -p` / bare `git show <hash>` (dumps hundreds of lines of code diffs).
   - ❌ `git diff main...HEAD` (full diff).

---

## 5. Branch Strategy: Linear Git Rebase & Auto-Cleanup

All batch feature development should happen on integration branches and integrate linearly into `main`:

```bash
# 1. Rebase feature branch onto latest main
git checkout feature/<batch-name>
git rebase main

# 2. Fast-forward merge into main
git checkout main
git merge --ff-only feature/<batch-name>

# 3. Safely delete the feature branch after confirmed merged
git branch -d feature/<batch-name>
```

---

## 6. Mandatory Command Execution & Git Guardrails

Whenever executing commands in this workspace, the agent MUST obey the following safety rules:
- **No Destructive Git Operations**: Under NO circumstances should the agent run:
  - `git push` / `git push --force`
  - `git reset --hard` (except worker.sh automated isolated rollback)
  - `git clean -f` / `git clean -fd`
  - `git branch -D`
  unless specifically and explicitly requested by the user.
- **No Host-Level Modifying Commands**: Under NO circumstances run `sudo`, `apt`, `yum`, `dnf`, `pacman`.

---

## 7. Engineering & Domain Treaties

- **Test-Driven Development (TDD)**: Follow Red-Green-Refactor cycle. Unit tests must pass 100%.
- **Deep Modules**: Design deep modules with simple, narrow interfaces.
- **Glossary Adherence**: Strictly adhere to terms in [GLOSSARY.md](file:///home/kw/workspace/auto_script/GLOSSARY.md).
- **ADR Review**: Respect architectural records in [docs/adr/](file:///home/kw/workspace/auto_script/docs/adr).

---

## 8. Framework Portability & Adoption

If you are an AI migrating this framework to another project, read the autonomous porting manual at [docs/PORTING_GUIDE.md](file:///home/kw/workspace/auto_script/docs/PORTING_GUIDE.md) and execute [.agents/skills/engineering/setup-unattended-workflow/SKILL.md](file:///home/kw/workspace/auto_script/.agents/skills/engineering/setup-unattended-workflow/SKILL.md).
