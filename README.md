<p align="center">
  <img src="https://ormus.solutions/mascot/pixellab_liquid_to_mercury.gif" alt="Automate Skills" width="128" style="image-rendering: pixelated;" />
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
