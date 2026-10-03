---
name: universal-build-verify
description: "Universal build, test, GUI smoke verification, honest retry tracking, and auto-commit SOP for unattended background workers."
---

# Universal Build, Test & Verification SOP (Background Worker)

This document is the **mandatory standard operating procedure** that the unattended background Worker AI MUST strictly follow when executing tasks popped from `tasks.txt`.

---

## 1. Strict Security & Anti-Rot Constraints (安全與防腐門禁)

1. **NO Global System Modifications**:
   - Under NO circumstances run `sudo`, `apt`, `apt-get`, `yum`, `dnf`, `pacman`, or any host system-level package manager.
2. **Local Environment Only**:
   - Dependencies must be managed locally within the project (`.venv`, `cmake` local prefix, `pkg-config`).
3. **Zero Compiler / Lint Warnings (零警告門禁)**:
   - Code must build cleanly with zero compiler warnings (e.g. `-Wall -Wextra`) and pass static checks without errors.
4. **Interface Immutability**:
   - When a task specifies `CONSTRAINTS: Do not change public API`, do not alter public method signatures.

---

## 2. Implementation, Build & Verification Workflow

### Step 2.0: Consult Notes & Real-Time Step Logging
- Check `.worker.notes.md` for project-specific quirks.
- Before invoking any command or modifying files, print an explicit line:
  - `[EXEC] Modifying <filename>...`
  - `[EXEC] Running build command: <command>`
  - `[EXEC] Running test command: <command>`
  - `[EXEC] Running git commit...`

### Step 2.1: Code Implementation & Honest Granularity Check
- **Granularity Rejection (過載拒絕)**: If a Level 1 task turns out to span across > 3 unrelated subsystems or > 300 lines of complex changes, DO NOT guess or struggle. Stop immediately, write `[NEED_DECOMPOSITION]` with a proposed subtask list, and exit cleanly.
- Implement the requested feature cleanly, following TDD.

### Step 2.2: Compilation & Syntax Verification
- **C / C++ Projects**: `cmake --build build -j$(nproc)` / `ninja -C build`
- **Python Projects**: `source .venv/bin/activate` -> syntax check `python3 -m py_compile` -> `pytest`

### Step 2.3: Automated Test Execution
- Run targeted tests for the modified component (e.g. `ctest -R <test_name>` or `pytest tests/test_<module>.py`).
- All tests must pass (100% green).

### Step 2.4: Desktop GUI & Window Smoke Testing
- Use `xvfb-run -a <executable>` or `timeout 2s <executable_path> || [ $? -eq 124 ]`.

---

## 3. Error Retry Limit & Intermediate Clean Reset (修復上限與中間重置)

When build, compilation, or tests fail:
1. **Maximum 3 Fix Attempts**:
   - **Attempt 1**: Analyze error trace, apply surgical fix, rebuild and retest.
   - **Attempt 2**: If Attempt 1 made things worse, run `git restore <file>` to reset back to baseline before trying an alternative fix.
   - **Attempt 3**: Final attempt.
2. **Immediate Stop on 3rd Failure**:
   - If unable to pass after 3 attempts, STOP immediately. Do NOT enter an infinite loop.
   - Mark the task as `[BLOCKED]` in `progress.log` and exit.

---

## 4. Honest Escalation Protocol (誠實求助協議)

If at any point:
- You discover requirement ambiguity, contradictory interfaces, or missing decisions:
- **DO NOT GUESS OR INVENT FAKE DATA.**
- Emit a `[NEED_GUIDANCE]` report at the top of `progress.log`:
```text
================================================================================
[NEED_GUIDANCE] TASK-XXX: <Task Title>
Status: BLOCKED_FOR_GUIDANCE
Issue: <Detailed explanation of the ambiguity or conflicting spec>
Question for Manager: <Direct, clear multiple-choice question for human/architect>
================================================================================
```
Then exit cleanly with exit code `2`.

---

## 5. Completion Standard & Evidence Logging (結案標準)

Upon 100% verification success:
1. **Auto Git Commit**:
   ```bash
   git add <modified-target-files>
   git commit -m "feat/fix: <task title> [TASK-ID]"
   ```
2. **Prepend Structured Summary & Attempt Trace to `progress.log`**:
```text
================================================================================
[SUCCESS] TASK-XXX: <Task Title>
Time: <YYYY-MM-DD HH:MM:SS> | Duration: <Xs> | Attempts: <N>/3 | Confidence: <HIGH|MEDIUM|LOW>
Task Granularity: <Level 1|Level 2|Level 3>
Changes: <List of modified files>
Verification: Build PASS, Tests PASS (<N> passed), GUI Smoke PASS

Attempt Trace:
  • Attempt 1: <PASS or failure reason>
  • Attempt 2: <Fix applied, if any>

Granularity Feedback: 
  <Worker observation on whether this module was comfortable at current granularity>
================================================================================
```
