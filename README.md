<p align="center">
  <img src="https://ormus.solutions/mascot/pixellab_liquid_to_ouroboros.gif" alt="Automate Skills" width="128" style="image-rendering: pixelated;" />
</p>

<h1 align="center">Automate Skills</h1>

<p align="center">
  <em>A Claude Code skill for turning repetitive tasks into reliable automation scripts — scope, design, implement, and harden.</em>
</p>

<p align="center">
  <a href="https://github.com/HermeticOrmus/automate-skills/stargazers"><img src="https://img.shields.io/github/stars/HermeticOrmus/automate-skills?style=flat-square&color=aa8142" alt="Stars" /></a>
  <a href="https://github.com/HermeticOrmus/automate-skills/blob/main/LICENSE"><img src="https://img.shields.io/github/license/HermeticOrmus/automate-skills?style=flat-square&color=aa8142" alt="License" /></a>
  <a href="https://github.com/HermeticOrmus/automate-skills/commits"><img src="https://img.shields.io/github/last-commit/HermeticOrmus/automate-skills?style=flat-square&color=aa8142" alt="Last Commit" /></a>
  <img src="https://img.shields.io/badge/Claude_Code-aa8142?style=flat-square&logo=anthropic&logoColor=white" alt="Claude Code" />
</p>

---

## The problem

Doing the same task by hand is error-prone and does not scale. You forget a step, you fat-finger a path, you run it on the wrong directory, and the cost grows every time the task recurs.

Automating it carelessly is worse. A script without error handling continues past a failed command and reports success. A script that is not idempotent corrupts the directory it runs on twice. A script that swallows errors to keep a green exit code fails silently at 3am and leaves no evidence. The manual task at least fails in front of you.

The discipline is in the middle: scope the task honestly, pick the simplest tool, implement the smallest version that works, then harden it so it fails safely and runs twice without harm.

## The design process

Five steps, each gating the next.

| Step | What it answers |
|---|---|
| **Scope the task** | What runs, how often, on what inputs, with what edge cases, at what cost if it fails or repeats |
| **Pick the simplest tool** | Bash, Python, Makefile, or a scheduler wrapping one of them |
| **Implement the smallest version** | The path that handles the real inputs, with `set -euo pipefail`, nothing speculative |
| **Add error handling and idempotency** | Validate inputs, fail loud, run twice without harm, guard destructive steps, log |
| **Dry-run, then schedule** | Prove it changes nothing in `--dry-run`, test the edge cases, then run it unattended |

Full content: [`CLAUDE.md`](CLAUDE.md). Worked automations: [`EXAMPLES.md`](EXAMPLES.md).

## Install

### As a project CLAUDE.md

Drop [`CLAUDE.md`](CLAUDE.md) at the root of your repository. Claude Code picks it up automatically. Merge with existing project instructions if any.

```bash
curl -o CLAUDE.md https://raw.githubusercontent.com/HermeticOrmus/automate-skills/main/CLAUDE.md
```

### As a Claude Code skill

The same content is packaged as a skill under [`skills/automate/`](skills/automate/) for `~/.claude/skills/`. See the `SKILL.md` inside for installation.

### As an `/automate` slash command

Save [`CLAUDE.md`](CLAUDE.md) (or a thin wrapper that points at it) as `~/.claude/commands/automate.md`. Claude Code then exposes it as `/automate`, which runs the protocol against whatever task you describe.

```bash
curl -o ~/.claude/commands/automate.md https://raw.githubusercontent.com/HermeticOrmus/automate-skills/main/CLAUDE.md
```

### In Cursor

See [`CURSOR.md`](CURSOR.md) for the Cursor-rule equivalent at [`.cursor/rules/automate.mdc`](.cursor/rules/automate.mdc).

### In other AI coding tools

If your tool reads a single instruction file at the project root, copy `CLAUDE.md` to whatever name your tool expects (`AGENTS.md`, `INSTRUCTIONS.md`, etc.).

## See also

- [`shell-safety-skills`](https://github.com/HermeticOrmus/shell-safety-skills): the companion, covering how to write the shell scripts this protocol produces so they do not destroy data. Pair the two when the automation is a Bash script.

## Contributing

PRs welcome, especially for additional worked automations in [`EXAMPLES.md`](EXAMPLES.md), translations of the README, and adaptations of `CURSOR.md` for other AI coding tools (Windsurf, Cline, Aider, Continue, etc.).

## License

MIT. Use it, fork it, merge it into your own CLAUDE.md.

---

## Part of the Libre Open-Source Stack for Claude Code

This repository is part of a growing family of open-source toolkits for Claude Code.

### Libre suite — comprehensive plugin bundles

- [LibreUIUX-Claude-Code](https://github.com/HermeticOrmus/LibreUIUX-Claude-Code) — UI/UX development (152 agents, 70 plugins, 76 commands, 74 skills)
- [LibreArch-Claude-Code](https://github.com/HermeticOrmus/LibreArch-Claude-Code) — Software architecture and system design
- [LibreCopy-Claude-Code](https://github.com/HermeticOrmus/LibreCopy-Claude-Code) — Technical writing and documentation engineering
- [LibreDevOps-Claude-Code](https://github.com/HermeticOrmus/LibreDevOps-Claude-Code) — DevOps engineering and infrastructure automation
- [LibreEmbed-Claude-Code](https://github.com/HermeticOrmus/LibreEmbed-Claude-Code) — Embedded systems, firmware, and IoT development
- [LibreFinTech-Claude-Code](https://github.com/HermeticOrmus/LibreFinTech-Claude-Code) — Financial technology development
- [LibreGEO-Claude-Code](https://github.com/HermeticOrmus/LibreGEO-Claude-Code) — AI-search optimization (ChatGPT, Perplexity, Gemini, Google AI Overviews)
- [LibreGameDev-Claude-Code](https://github.com/HermeticOrmus/LibreGameDev-Claude-Code) — Game development across Godot, Unity, Unreal
- [LibreMLOps-Claude-Code](https://github.com/HermeticOrmus/LibreMLOps-Claude-Code) — ML engineering and AI operations
- [LibreMobileDev-Claude-Code](https://github.com/HermeticOrmus/LibreMobileDev-Claude-Code) — Mobile app development (Flutter, React Native, native iOS, native Android)
- [LibreSecOps-Claude-Code](https://github.com/HermeticOrmus/LibreSecOps-Claude-Code) — Security operations
- [LibreSessionFlow-Claude-Code](https://github.com/HermeticOrmus/LibreSessionFlow-Claude-Code) — Session lifecycle: handoff, pickup, absorb, explore, close

### Skills mini-repos — single CLAUDE.md drop-ins

- [vibe-engineer-skills](https://github.com/HermeticOrmus/vibe-engineer-skills) — Direct AI codegen well: hypothesis before help, scoped prompts, validate before accepting
- [markdown-discipline-skills](https://github.com/HermeticOrmus/markdown-discipline-skills) — Strip AI-slop from markdown (no em dashes, no marketing fluff)
- [shell-safety-skills](https://github.com/HermeticOrmus/shell-safety-skills) — `set -euo pipefail` discipline plus 15 failure-mode examples
- [commit-standard-skills](https://github.com/HermeticOrmus/commit-standard-skills) — Ormus Commit Standard v1.0 plus commit-msg hook and commitlint
- [unwoke-skills](https://github.com/HermeticOrmus/unwoke-skills) — Strip AI theater (ten sins to eliminate, symmetric engagement)
- [python-conventions-skills](https://github.com/HermeticOrmus/python-conventions-skills) — Modern Python 3.11+ (types, pathlib, async, ruff, mypy, uv)
- [typescript-conventions-skills](https://github.com/HermeticOrmus/typescript-conventions-skills) — TypeScript strict mode, discriminated unions, Result types
- [hermetic-laws-skills](https://github.com/HermeticOrmus/hermetic-laws-skills) — Seven Hermetic Principles applied to engineering
- [riper-workflow-skills](https://github.com/HermeticOrmus/riper-workflow-skills) — Research / Innovate / Plan / Execute / Review systematic dev
- [six-day-cycle-skills](https://github.com/HermeticOrmus/six-day-cycle-skills) — Sustainable shipping cadence with mandatory rest
- [token-optimization-skills](https://github.com/HermeticOrmus/token-optimization-skills) — Claude Code token and context optimization
- [osint-skills](https://github.com/HermeticOrmus/osint-skills) — OSINT research methodology (multi-wave investigative spiral)
- [calcinate-skills](https://github.com/HermeticOrmus/calcinate-skills) — Stage 1 of the Magnum Opus (burn project bloat)
- [claude-md-overhaul-skills](https://github.com/HermeticOrmus/claude-md-overhaul-skills) — Audit CLAUDE.md and MEMORY.md against caps
- [session-handoff-skills](https://github.com/HermeticOrmus/session-handoff-skills) — Session handoff and pickup discipline
- [naming-skills](https://github.com/HermeticOrmus/naming-skills) — Product naming methodology (mine the brand's vocabulary)
- [magnum-opus-skills](https://github.com/HermeticOrmus/magnum-opus-skills) — Seven-stage alchemy applied to project transformation
- [mem-search-skills](https://github.com/HermeticOrmus/mem-search-skills) — Search claude-mem cross-session memory: search, filter, fetch
- [hypothesis-debugging-skills](https://github.com/HermeticOrmus/hypothesis-debugging-skills) — Hypothesis-driven debugging: reproduce, isolate, test, fix
- [vibe-proof-skills](https://github.com/HermeticOrmus/vibe-proof-skills) — Security hardening for vibe-coded full-stack apps
- [tdd-skills](https://github.com/HermeticOrmus/tdd-skills) — Test-driven development (Red-Green-Refactor) for JS/TS and Python
- [mars-skills](https://github.com/HermeticOrmus/mars-skills) — Production-readiness audit: the five mortal sins of vibe-coded MVPs
- [git-workflow-skills](https://github.com/HermeticOrmus/git-workflow-skills) — Clean git workflow: branch, atomic commits, reviewable PRs
- [code-review-skills](https://github.com/HermeticOrmus/code-review-skills) — Domain-aware code review: classify the code, then focus
- [code-comprehension-skills](https://github.com/HermeticOrmus/code-comprehension-skills) — Understand an unfamiliar codebase fast
- [dx-audit-skills](https://github.com/HermeticOrmus/dx-audit-skills) — Audit developer experience: docs, onboarding, tooling friction
- [setup-env-skills](https://github.com/HermeticOrmus/setup-env-skills) — Set up a project's development environment
- [quick-fix-skills](https://github.com/HermeticOrmus/quick-fix-skills) — Fast troubleshooting for common issues
- [prime-context-skills](https://github.com/HermeticOrmus/prime-context-skills) — Prime project context at the start of a session
- [auto-docs-skills](https://github.com/HermeticOrmus/auto-docs-skills) — Generate and maintain project documentation
- [learning-skills](https://github.com/HermeticOrmus/learning-skills) — Learn any technology: roadmaps, explanations, practice, cheatsheets, comparisons
- [linux-sysadmin-skills](https://github.com/HermeticOrmus/linux-sysadmin-skills) — Linux system administration: security, performance, diagnostics, monitoring, maintenance

### Template source

- [andrej-karpathy-skills](https://github.com/HermeticOrmus/andrej-karpathy-skills) — the canonical single-file CLAUDE.md pattern (fork of jiayuan_jy's original)

Star the family, not just one — that's how the suite stays coherent.
