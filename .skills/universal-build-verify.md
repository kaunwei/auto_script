---
name: universal-build-verify
description: "Universal build, test, GUI smoke verification, and auto-commit SOP for unattended background workers."
---

# Universal Build, Test & Verification SOP (Background Worker)

This document is the **mandatory standard operating procedure** that the unattended background Worker AI MUST strictly follow when executing tasks popped from `tasks.txt`.

---

## 1. Strict Security Constraints (安全約束)

1. **NO Global System Modifications**:
   - Under NO circumstances run `sudo`, `apt`, `apt-get`, `yum`, `dnf`, `pacman`, or any host system-level package manager.
2. **Local Environment Only**:
   - Dependencies must be managed locally within the project.
   - For Python: use local virtual environments (`.venv` or `venv`).
   - For C/C++: use local build paths, `FetchContent`, `vcpkg` (local), or `pkg-config` checks.
3. **Missing System Dependencies**:
   - If a required system library or compiler tool is absent on the host, **DO NOT attempt to install it via sudo**.
   - Mark the task as `[BLOCKED: Missing dependency <name>]`, output the remediation command for the human in `progress.log`, and terminate immediately.

---

## 2. Implementation, Build & Verification Workflow

### Step 2.1: Code Implementation
- Open only the specified `TARGET` files.
- Implement the requested feature, bug fix, or refactor cleanly according to the `ACTION` specification.
- Follow Test-Driven Development (TDD) where applicable. Write unit tests for new logic.

### Step 2.2: Compilation & Syntax Verification
- **C / C++ Projects**:
  - Prefer Ninja: `cmake -B build -S . -G Ninja && cmake --build build --parallel`
  - Fallback to Make: `cmake -B build -S . && cmake --build build -j$(nproc)`
- **Python Projects**:
  - Activate venv if present: `source .venv/bin/activate`
  - Syntax check: `python3 -m py_compile $(git ls-files '*.py')`
  - Optional static check (if configured): `mypy` or `flake8` / `ruff`

### Step 2.3: Automated Test Execution
- **C / C++**: Run `ctest --test-dir build --output-on-failure` or execute test binaries directly.
- **Python**: Run `pytest -v` or `python3 -m unittest discover tests`.
- **All tests must pass (100% green).**

### Step 2.4: Desktop GUI & Window Smoke Testing
For projects containing desktop GUI windows (e.g. Qt, GTK, Tkinter, Pygame, SDL, ImGui, X11/Wayland apps):
1. **Prevent Process Hanging**: Never launch a GUI event loop without a timeout or headless wrapper in unattended mode.
2. **Headless Execution**:
   - If `xvfb-run` is available: `xvfb-run -a <executable>`
   - If `xvfb-run` is not installed or testing native binary: wrap with timeout:
     ```bash
     timeout 2s <executable_path> || [ $? -eq 124 ]
     ```
   - An exit code of `124` (SIGTERM by timeout) after clean startup without crash/exceptions is treated as a **PASS** for the GUI smoke test.

---

## 3. Error Retry Limit (修復上限原則)

When compilation, linking, tests, or smoke tests fail:
1. **Maximum 3 Fix Attempts**:
   - Attempt 1: Read the exact error trace, hypothesize the root cause, apply a surgical fix, rebuild and retest.
   - Attempt 2: If secondary errors occur, re-evaluate and refine fix.
   - Attempt 3: Final attempt to resolve any remaining issues.
2. **Immediate Stop on Exceeded Limit**:
   - If the task cannot pass after the 3rd attempt, **IMMEDIATELY STOP**.
   - Do NOT enter an infinite loop.
   - Mark the task as `[BLOCKED]` in `progress.log` with the exact failure reason and exit with error status.

---

## 4. Completion Standard & Evidence Logging (結案標準)

Upon 100% verification success:
1. **Auto Git Commit**:
   ```bash
   git add <modified-target-files>
   git commit -m "feat/fix: <task title> [TASK-ID]"
   ```
2. **Prepend Structured Summary to `progress.log`**:
   Write a concise summary (strictly **≤ 10 lines**) at the **VERY TOP** of `progress.log`:

```text
================================================================================
[SUCCESS] TASK_ID: <Task Title>
Time: <YYYY-MM-DD HH:MM:SS> | Commit: <Short Hash> | Duration: <Xs>
Changes: <List of modified files and functions>
Verification: Build PASS, Tests PASS (<N> passed), GUI Smoke PASS
Notes: <Brief note or N/A>
================================================================================
```

If blocked, prepend the `[BLOCKED]` report:
```text
================================================================================
[BLOCKED] TASK_ID: <Task Title>
Time: <YYYY-MM-DD HH:MM:SS> | Status: BLOCKED after 3 attempts
Failure Reason: <Exact error summary>
Action Needed for Human: <Clear instructions for the human operator>
================================================================================
```
