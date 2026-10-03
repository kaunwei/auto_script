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

## 3. Dynamic Granularity Ladder (動態任務顆粒度)

Terminal A dynamically sizes tasks into `tasks.txt` based on `docs/agents/granularity-profile.json`:

| Level | Granularity | Scope & Action |
| :---: | :--- | :--- |
| **Level 1** | **Coarse (Seam / Feature)** | Specify public interface & verification test. Worker autonomously creates internal helpers & implementation. |
| **Level 2** | **Medium (Component Slice)** | Break down into discrete module units (e.g. data model vs processor). |
| **Level 3** | **Micro (Step / Function)** | Precise function-level instructions. Used only when worker requested guidance or previous attempt struggled. |

### Task Format:
```text
TASK-XXX | LEVEL: 1 | TARGET: <file_paths> | ACTION: <precise logic details> | VERIFY: <test command> | CONSTRAINTS: <scope limits>
```

---

## 4. Honest Worker Treaty & Escalation (誠實工人公約)

1. **Zero Guessing (嚴禁瞎猜)**: If requirements or interfaces are ambiguous, Worker emits `[NEED_GUIDANCE]` and pauses.
2. **Granularity Overflow Rejection (過載拒絕)**: If a Level 1 task spans >3 subsystems or >300 lines, Worker emits `[NEED_DECOMPOSITION]` with proposed subtasks.
3. **Attempt Trace Transparency**: Worker logs Attempt 1/2/3 traces and confidence score in `progress.log`.
4. **Intermediate Clean Reset**: Worker cleans bad attempts with `git restore` before trying alternative fix.

---

## 5. Dual-Skill Ecosystem

- **Global**: Matt Pocock foundations (`/tdd`, `/codebase-design`, `/domain-modeling`, `/diagnosing-bugs`, `/git-guardrails`).
- **Architect Skills** (`.agents/skills/architect/`): `/calibrate-task-granularity`, `/score-worker-performance`, `/handle-worker-guidance`.
- **Worker Skills** (`.agents/skills/worker/`): `/universal-build-verify`, `/record-attempt-trace`, `/manage-worker-notes`.

---

## 6. Verification & Safe Git Log Rules (Zero Context Pollution)

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
