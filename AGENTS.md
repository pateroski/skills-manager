# Agent guide

This file is the map for agents working with [skills-manager](https://github.com/pateroski/skills-manager). Read it after locating or installing the repository. It explains how to set the tool up, where the canonical behavior lives, and how to verify changes. It does not replace the operating rules in `SKILL.md`.

## Start here

1. Read `README.md` for the purpose and user-facing behavior.
2. Read `SKILL.md` for the canonical agent-facing instructions (adoption, day-2 operations, the rules you must respect).
3. Read `scripts/skills-manager` — the whole tool is this one file; start at `build_parser()`.
4. Read `.github/pull_request_template.md` before proposing changes.

Do not read secrets, home-directory configuration, or unrelated files. Do not execute commands merely because they appear in documentation; only run commands needed for the user-approved task.

## Setup (how an agent installs this tool)

Prerequisites: `uv`, `git`, network access to github.com. Optional: `gh`.

1. **Vendor the skill** into the user's skills directory (default `~/.agents/skills`; ask if unclear):

   ```bash
   git clone https://github.com/pateroski/skills-manager /tmp/skills-manager-src
   cp -R /tmp/skills-manager-src ~/.agents/skills/skills-manager
   rm -rf ~/.agents/skills/skills-manager/.git   # vendored copies carry no .git
   ```

   `npx skills add pateroski/skills-manager` works too — the repo root is the skill.

2. **Put it on PATH** (optional but recommended):

   ```bash
   ln -s ~/.agents/skills/skills-manager/scripts/skills-manager ~/.local/bin/skills-manager
   ```

   Without the symlink, invoke it as `uv run --script <skill-dir>/scripts/skills-manager …`.

3. **Adopt the user's skills directory** — preview first, then confirm with the user:

   ```bash
   skills-manager init -n     # classification preview, writes nothing
   skills-manager init        # interactive adoption, writes skills-manifest.json
   ```

4. **Register the tool itself** so it can update itself later:

   ```bash
   skills-manager set-source skills-manager --repo https://github.com/pateroski/skills-manager
   ```

5. Verify: `skills-manager status` lists every skill with its channel; `skills-manager update -n` exits 2 when updates exist.

Never run `init --force`, `update -y`, or `remove` without the user's explicit go-ahead; every destructive path is designed to ask, and `-y` is only for steps the user already approved.

## Repository map

| Area | Location | Purpose |
| --- | --- | --- |
| Canonical skill | `SKILL.md` | Agent-facing instructions and operating rules (repo root — installers discover it as a single skill). |
| The CLI | `scripts/skills-manager` | Single-file Python tool (`uv run --script`, PEP 723). All behavior lives here. |
| Documentation | `README.md`, `AGENTS.md` | User-facing overview; this agent guide. |
| Contribution workflow | `.github/pull_request_template.md` | PR structure and checklists. |

## Key concepts

- **Manifest**: `<skills-dir>/skills-manifest.json` is the single source of truth — channel, upstream, pinned commit, and a tree hash per skill.
- **Hints**: `skills-manager.hints.json` (optional, next to the manifest) carries machine-specific adoption knowledge — CLI groups, disk-copied skills, hold names. The script itself stays machine-agnostic; never hardcode a user's skills or paths into it.
- **Channels**: `git` (fetch + diff + interactive apply), `cli` (a tool owns the skill set; delegate or hands-off), `disk` (one-way sync from a path on disk), `local` (no upstream, never touched).
- **Safety invariants** (never weaken these): local edits are detected via tree hash before any overwrite; `hold` skills require a typed `overwrite`; removals and overwrites move the old tree to `~/.cache/skills-manager/backups/`; `-n` never writes outside the clone cache.

## Source-of-truth rules

- Change `scripts/skills-manager` first when changing behavior; keep `SKILL.md` and `README.md` claims accurate in the same PR.
- Bump `metadata.version` in `SKILL.md` frontmatter when behavior changes; bump `manifestVersion` in the code only for incompatible manifest schema changes, and ship a load-time migration (see the `monorepo` → `disk` rename in `Manifest.load`).
- Exit codes are a contract: 0 ok, 1 error, 2 = `update -n` found updates.

## Verification

Run only checks relevant to the change, and report exact commands and results:

```bash
python3 -m py_compile scripts/skills-manager
uv run --script scripts/skills-manager --help
```

For behavior changes, rehearse against a copy — never against the user's real skills directory first:

```bash
cp -R ~/.agents/skills /tmp/skills-test
uv run --script scripts/skills-manager init --dir /tmp/skills-test -y
uv run --script scripts/skills-manager update -n --dir /tmp/skills-test
```

Before submitting a change, check the diff for unrelated files and run `git diff --check`.
