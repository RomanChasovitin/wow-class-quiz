# WoW Class Quiz — agent contract

A Russian-language quiz that helps a player pick a role, class and race for WoW Forever from indirect questions. Stage: concept; the race × class × faction table for Forever comes first.

Roman's personal working agreement (`~/.agents/AGENTS.md`) applies on top of this file. This file holds
only what is specific to this project.

## Commands

| Purpose | Command |
|---------|---------|
| Install | none yet |
| Run | none yet |
| Check (types, lint) | none yet |
| Test | none yet |
| Build | none yet |

## Rules

- **Branches:** `main` is the default. Use a branch for anything that is not a small, self-contained
  change.
- **Commits:** a short imperative subject in English, one logical change per commit. Commit and push
  without asking.
- **Specs:** `specs/YYYY-MM-DD-<topic>-design.md`, written by the discuss skill.
- **Gotchas:** `docs/gotchas.md`. Create it with the first non-obvious trap.
- **Tests:** none yet. Add a test stack only as a deliberate project decision.
- **GitHub:** this is a personal repository: use `gh-rch`, not `gh`. Git picks the personal identity
  from the remote URL by itself.

## Code search

codegraph when the checkout has an index (`codegraph init .` once per checkout); ripgrep otherwise.

## Subagents

- Claude work here runs on the Personal subscription (`claudeAgent_personal`). Its budget is small:
  use subagents only for genuinely parallel work.
- No cross-vendor review in this project: the skills end with self-review and the checks above.

## Skills

`.claude/skills/` (Codex sees them through `.agents/skills`): **discuss** → **build** → **debug**, and
**review** for other people's PRs. They started from the templates in the `bond` repository. The
copies here are this project's working versions: edit them here, never in bond. Propose porting an
edit back to bond only when it would help every project.
