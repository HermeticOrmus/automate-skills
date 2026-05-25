# CLAUDE.md

A protocol for turning a repetitive task into a reliable automation script. Merge with project-specific instructions as needed.

Automation done well removes toil. Automation done carelessly fails silently at 3am, corrupts a directory it ran on twice, or hides an error behind a zero exit code. This file is the discipline that separates the two.

**Tradeoff**: this protocol biases toward scoping and hardening over speed. For a one-off command you will run once and watch, use judgment.

## 1. Scope the task before writing code

Define what you are automating before you choose how.

- What is the task, in one sentence?
- How often does it run, and what triggers it (manual, schedule, file change, event)?
- What are the inputs (files, arguments, environment, network) and outputs (files, logs, exit code, side effects)?
- What are the edge cases (empty input, partial input, already-done state, missing dependency, no network)?
- What is the cost of running it twice? Of it failing halfway?

A task you cannot describe in one sentence is two tasks. Split it.

## 2. Pick the simplest tool that fits

Match the tool to the task, not to preference. Bias toward the lightest option that covers the edge cases from step 1.

- Bash: file moves, system commands, glue between existing CLI tools, anything already shaped as shell.
- Python: branching logic, structured data (JSON, CSV), API calls, anything where Bash quoting becomes the hard part.
- Makefile: multi-step build or task graphs where steps depend on each other and on file timestamps.
- A scheduler (systemd timer or cron) wraps any of the above for unattended runs. It is not itself the automation.

If the task is three shell commands, it is a script, not a program. Do not reach for Python to move files.

## 3. Implement the smallest version that works

Write the path that handles the real inputs first. No speculative flags, no config system for a single caller, no plugin architecture.

```bash
#!/usr/bin/env bash
# what it does, how to run it, what it depends on
set -euo pipefail

readonly SRC="${1:?usage: backup.sh <src> <dest>}"
readonly DEST="${2:?usage: backup.sh <src> <dest>}"

main() {
  # the one job this script has
}

main "$@"
```

`set -euo pipefail` is the floor: exit on error, exit on undefined variable, fail a pipeline if any stage fails. A script without it continues after a failed command and reports success.

## 4. Add error handling and idempotency

A reliable automation can fail safely and can run twice without harm.

- Validate inputs before acting. A missing file is a clear error message, not a stack trace halfway through.
- Make it idempotent: running it again on the same state should be a no-op, not a duplicate or a corruption. Check for the already-done state and skip it.
- Fail loud. Never swallow an error to keep a green exit code. The exit code is the contract a scheduler reads.
- Log what it did and when, so an unattended failure leaves evidence.
- Guard destructive steps. A delete or overwrite gets a confirmation prompt when interactive, and a `--force` or `--yes` flag when not.

## 5. Dry-run, then make it runnable on a schedule

Prove the script before you trust it unattended.

- Add a `--dry-run` mode that prints what it would do and changes nothing. Run it first on real inputs.
- Test the edge cases from step 1 deliberately: empty input, missing dependency, the already-done state, a forced failure.
- Only then schedule it. A systemd timer or cron entry runs the verified script; capture its output to a log and alert on non-zero exit.
- Pin paths and environment in the unit or crontab. A schedule runs with a minimal environment, not your interactive shell, so absolute paths and explicit `PATH` prevent the most common unattended failure.

---

## Trigger this protocol when

- A task is described as "every time" or "I keep having to" → it is a candidate for automation; scope it (step 1)
- A script is being written without `set -euo pipefail` → add it (step 3)
- A script writes, deletes, or overwrites → check idempotency and guard the destructive step (step 4)
- A script is about to be scheduled → require a dry-run first (step 5)
- An error is being caught only to keep the exit code green → reject; fail loud (step 4)

---

**Companion**: pair this with [`shell-safety-skills`](https://github.com/HermeticOrmus/shell-safety-skills) for safe construction of the shell scripts this protocol produces.

**License**: MIT. Use it, fork it, merge it into your own.
