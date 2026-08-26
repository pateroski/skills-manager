---
name: skills-manager
description: >
  Install, update, remove, and track agent skills (SKILL.md folders) in
  ~/.agents/skills or any skills directory. Use when the user asks to update
  their skills, install a new skill from a repo/URL, remove a skill, adopt an
  existing skills folder under management, check which skills have upstream
  updates, or find where an untracked skill came from. Wraps the
  skills-manager CLI (JSON manifest, per-channel updates, local-edit
  protection, breaking-change advisories).
metadata:
  source: github.com/pateroski/skills-manager
  version: 0.2.0
---

# skills-manager

Manage a directory of agent skills with a JSON manifest as the single source
of truth: `<skills-dir>/skills-manifest.json` records every skill's channel,
upstream, pinned commit, and a content hash of what was installed, so local
edits are always detected before an update can overwrite them.

## Prerequisites

- `uv` (runs the script — PEP 723 inline deps, no install step)
- `git` (fetch engine)
- Network access to github.com for git-channel operations
- Optional: `gh` CLI (improves upstream discovery)

The CLI lives at `scripts/skills-manager` inside this skill folder. If the
`skills-manager` command is not on PATH, invoke it as
`uv run --script <this-skill-dir>/scripts/skills-manager …`.

## Conventions

- `-n` / `--dry-run`: report only, change nothing (safe to run anytime)
- `-y` / `--yes`: auto-confirm *safe* steps only (clean fast-forward updates);
  held/edited/breaking skills always stay interactive
- `--dir PATH`: operate on any skills directory (default `~/.agents/skills`)
- Exit code 2 from `update -n` = updates are available
- Nothing is ever deleted: removals and overwrites move the previous tree to
  `~/.cache/skills-manager/backups/<name>-<timestamp>/`

## Adopting an existing skills folder (setup)

```bash
skills-manager init --dir <folder> -n   # preview the classification table
skills-manager init --dir <folder>      # review, edit rows, confirm, write manifest
```

Works on any folder containing `*/SKILL.md`; no prior metadata needed.
Classification order: CLI groups from the optional hints file → frontmatter
`metadata.source` → URL hints → disk hints → `local`. The script knows no
specific skill or tool — CLI ownership is something you discover while using
the skill (see the research rule below) and record via `set-source
--channel cli` or the hints file. Skills with an embedded `.git` trigger a
prompt to archive it to the backup area.

Machine-specific knowledge (CLI groups, disk-copied skills, hold names)
never lives in the script — put it in `skills-manager.hints.json` next to
the manifest (schema documented at the top of `scripts/skills-manager`).
An existing manifest can regenerate it: group `cli` entries, `disk` entries,
and `hold` flags.

## Day-2 operations

```bash
skills-manager status              # the register: channel, upstream, pin, drift
skills-manager update -n           # check everything (network, no writes)
skills-manager update              # interactive apply; -y auto-applies clean updates
skills-manager add owner/repo      # install; also URLs, /tree/<ref>/<path>, gists, local repo paths
skills-manager add owner/repo --skill NAME --name ALIAS
skills-manager update NAME --apply # apply a flagged (breaking) update after user approval
skills-manager remove NAME         # moves to backup, drops manifest entry
skills-manager snapshot NAME       # accept local edits as the new baseline
skills-manager hold NAME [--off] [--note TEXT]
skills-manager adopt-one NAME      # register a dir that appeared after init
```

## Files in the skills root (not skills)

The tool owns exactly three non-skill files next to the skill folders:
`skills-manifest.json` (the register), `skills-manifest.json.bak` (the single
rotated backup of the previous manifest — there is never more than one), and
`skills-manager.hints.json` (optional per-machine hints). Anything else is
flagged by `status`/`init` as a stray: investigate which tool put it there
(installers like hyperframes' drop their own artifacts, e.g. a lockfile or
test files) before whitelisting it in the hints `knownRootFiles` list or
removing it — never delete a file another tool owns, it will come back.

## Rules the agent must respect

- **Never classify a copied skill as `disk` or `local` without researching its
  real upstream first.** A skill copied from a directory usually originates
  from a git repo or a CLI tool. Before adopting/adding it, run `discover`,
  search the web and GitHub code for its distinctive phrases, check for CLI
  ownership markers (e.g. OpenSpec's `.openspec-target`), and only fall back
  to `disk`/`local` when no verifiable upstream exists.
- **Never hand-update `cli`-channel skills.** The plannotator group is
  hands-off (its binary rewrites those skills); the hyperframes group is
  refreshed only via its own command, which `update` runs for you.
- **`hold: true` skills carry deliberate local adaptations.** Never overwrite
  them without the user explicitly deciding; the CLI requires typing
  `overwrite` and re-prints the adaptations note.
- **Breaking-change flags** (major version bump, `BREAKING`/`!:` commits,
  frontmatter name/description changes, CHANGELOG touched) exclude a skill
  from `-y` auto-apply — show the user the evidence first, then apply the
  ones they approve with `update <name> --apply` (hold skills still refuse).
- When a plugin repo has several skills **and** extra artifacts
  (hooks/commands/agents/MCP), recommend installing it as a harness plugin
  instead of vendoring; vendor only the skills if the user prefers.

## Finding the upstream of an untracked skill (agent-assisted discovery)

For skills the manifest classifies as `local` with no known source:

```bash
skills-manager discover [NAME ...] --json
```

This emits a research brief per skill: description, URL hints found in its
files, distinctive verbatim phrases, and suggested queries. Then, as the
agent:

1. Search with WebSearch, code-search MCPs (grep.app / gh_grep, exa), or
   `gh api search/code` using the exact phrases as quoted strings.
2. Verify a candidate by comparing its SKILL.md name/description against the
   local one — never trust a name match alone.
3. Record the confirmed upstream:
   `skills-manager set-source <name> --repo <URL> [--subdir P] [--commit SHA]`

A wrong upstream would let `update` overwrite the skill with a stranger's
content, so only record sources you verified.
