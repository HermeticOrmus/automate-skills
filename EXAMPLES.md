# Examples

Two repetitive tasks taken through the protocol end to end: scope, tool choice, the smallest working version, error handling and idempotency, and a dry-run before scheduling.

---

## 1. Backup and rotate

A directory needs a nightly snapshot, keeping the last seven and deleting older ones.

### Scope the task

- Task: archive a source directory to a timestamped tarball, then delete archives older than the seven most recent.
- Frequency: nightly, unattended (scheduled).
- Inputs: source directory path, destination directory path.
- Outputs: one `.tar.gz` per run, named by date; a log line per run; an exit code.
- Edge cases: source missing, destination not writable, a backup for today already exists (re-run), fewer than seven archives present (nothing to rotate).
- Cost of running twice: without a guard, two tarballs for the same day and a wasted rotation. With a guard, a no-op.

### Pick the tool

Bash. The task is `tar`, a glob, and a delete: existing CLI tools glued together. Python would add nothing.

### Smallest version that works, with error handling and idempotency

```bash
#!/usr/bin/env bash
# backup-rotate.sh - nightly tarball of SRC into DEST, keep newest 7.
# usage: backup-rotate.sh <src-dir> <dest-dir> [--dry-run]
set -euo pipefail

readonly SRC="${1:?usage: backup-rotate.sh <src-dir> <dest-dir> [--dry-run]}"
readonly DEST="${2:?usage: backup-rotate.sh <src-dir> <dest-dir> [--dry-run]}"
readonly DRY_RUN="${3:-}"
readonly KEEP=7
readonly STAMP="$(date +%Y-%m-%d)"
readonly ARCHIVE="${DEST}/backup-${STAMP}.tar.gz"

log() { printf '%s %s\n' "$(date -Iseconds)" "$*"; }

run() {
  if [[ "$DRY_RUN" == "--dry-run" ]]; then
    log "DRY-RUN: $*"
  else
    "$@"
  fi
}

main() {
  [[ -d "$SRC" ]]  || { log "ERROR: source not found: $SRC"; exit 1; }
  [[ -d "$DEST" ]] || { log "ERROR: dest not found: $DEST"; exit 1; }

  # Idempotent: today's archive already exists -> no-op.
  if [[ -e "$ARCHIVE" ]]; then
    log "archive for ${STAMP} already exists, skipping: $ARCHIVE"
    return 0
  fi

  log "creating $ARCHIVE"
  run tar -czf "$ARCHIVE" -C "$SRC" .

  # Rotate: list newest first, drop the first KEEP, delete the rest.
  mapfile -t old < <(ls -1t "${DEST}"/backup-*.tar.gz 2>/dev/null | tail -n +$((KEEP + 1)))
  for f in "${old[@]}"; do
    log "rotating out $f"
    run rm -- "$f"
  done

  log "done"
}

main "$@"
```

What the hardening buys: `set -euo pipefail` stops the script the moment `tar` fails instead of rotating against a half-written archive. The `[[ -e "$ARCHIVE" ]]` check makes a re-run a no-op. `rm --` and quoted variables survive paths with spaces. Every action is logged with a timestamp, so an unattended failure is visible in the log.

### Dry-run, then schedule

```bash
# Prove it changes nothing first.
./backup-rotate.sh /home/ormus/data /backups --dry-run
```

```text
2026-05-24T22:00:00-05:00 creating /backups/backup-2026-05-24.tar.gz
2026-05-24T22:00:00-05:00 DRY-RUN: tar -czf /backups/backup-2026-05-24.tar.gz -C /home/ormus/data .
2026-05-24T22:00:00-05:00 rotating out /backups/backup-2026-05-17.tar.gz
2026-05-24T22:00:00-05:00 DRY-RUN: rm -- /backups/backup-2026-05-17.tar.gz
2026-05-24T22:00:00-05:00 done
```

The dry-run confirms the right archive name, the right `tar` invocation, and that exactly one old archive would be rotated. Only then schedule it with a systemd timer, capturing output to a log and alerting on non-zero exit:

```ini
# ~/.config/systemd/user/backup-rotate.service
[Unit]
Description=Nightly backup and rotate

[Service]
Type=oneshot
ExecStart=/home/ormus/bin/backup-rotate.sh /home/ormus/data /backups
```

```ini
# ~/.config/systemd/user/backup-rotate.timer
[Unit]
Description=Run backup-rotate nightly

[Timer]
OnCalendar=*-*-* 02:00:00
Persistent=true

[Install]
WantedBy=timers.target
```

The unit uses absolute paths because a timer runs with a minimal environment, not your interactive shell. `Persistent=true` runs a missed backup after the machine wakes.

---

## 2. Batch file rename

A folder of scanned invoices is named `IMG_0001.jpg`, `IMG_0002.jpg`, and so on. They need to become `invoice-0001.jpg` in place.

### Scope the task

- Task: rename every `IMG_NNNN.jpg` in a directory to `invoice-NNNN.jpg`.
- Frequency: occasional, run by hand after a scan batch.
- Inputs: a directory path.
- Outputs: renamed files; a count of what changed.
- Edge cases: a target name already exists (collision, must not clobber), no matching files, filenames with unexpected shapes, the script run twice on an already-renamed folder.
- Cost of running twice: with a collision guard and a match on the source pattern only, a second run is a no-op because no `IMG_*.jpg` remain.

### Pick the tool

Bash again. It is a glob plus a string substitution. The one real risk is clobbering an existing target, which a guard handles.

### Smallest version that works, with error handling and idempotency

```bash
#!/usr/bin/env bash
# rename-invoices.sh - IMG_NNNN.jpg -> invoice-NNNN.jpg in DIR.
# usage: rename-invoices.sh <dir> [--dry-run]
set -euo pipefail

readonly DIR="${1:?usage: rename-invoices.sh <dir> [--dry-run]}"
readonly DRY_RUN="${2:-}"

log() { printf '%s\n' "$*"; }

main() {
  [[ -d "$DIR" ]] || { log "ERROR: not a directory: $DIR"; exit 1; }

  shopt -s nullglob              # an empty glob expands to nothing, not a literal
  local count=0
  for src in "$DIR"/IMG_*.jpg; do
    local base num dst
    base="$(basename "$src")"
    num="${base#IMG_}"           # strip prefix -> NNNN.jpg
    dst="${DIR}/invoice-${num}"

    if [[ -e "$dst" ]]; then     # never clobber an existing target
      log "SKIP (target exists): $base -> $(basename "$dst")"
      continue
    fi

    if [[ "$DRY_RUN" == "--dry-run" ]]; then
      log "DRY-RUN: $base -> $(basename "$dst")"
    else
      mv -n -- "$src" "$dst"     # -n is a second guard against overwrite
      log "renamed: $base -> $(basename "$dst")"
    fi
    ((count++)) || true
  done

  log "matched $count file(s)"
}

main "$@"
```

What the hardening buys: `shopt -s nullglob` means a directory with no `IMG_*.jpg` produces "matched 0 file(s)" instead of trying to rename a file literally called `IMG_*.jpg`. The `[[ -e "$dst" ]]` check plus `mv -n` are two independent guards against destroying an existing `invoice-NNNN.jpg`. Matching only the `IMG_*` source pattern makes the script idempotent: once renamed, a second run finds nothing to do.

### Dry-run, then run

```bash
./rename-invoices.sh ~/scans/2026-05 --dry-run
```

```text
DRY-RUN: IMG_0001.jpg -> invoice-0001.jpg
DRY-RUN: IMG_0002.jpg -> invoice-0002.jpg
SKIP (target exists): IMG_0003.jpg -> invoice-0003.jpg
matched 3 file(s)
```

The dry-run surfaces the collision on `0003` before any file moves, so you can resolve it by hand rather than discovering it after a destructive run. Once the output looks right, drop `--dry-run`.

---

## A note on cost

Each step above costs time the manual task does not. Scoping the edge cases is slower than opening a terminal and typing `mv`. Writing a dry-run mode is slower than running the real thing. Two collision guards feel like overkill on a folder you can see.

The cost compounds the other way too. The rename without a clobber guard overwrites an invoice you needed. The backup without an idempotency check fills the disk with duplicate tarballs. The scheduled script with a relative path runs fine by hand and fails every night because cron's `PATH` is not yours.

The discipline is paying the up-front cost because the task recurs, and a task that recurs will eventually run on the day the edge case is present.
