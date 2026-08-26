<p align="center">
  <strong align="center">Manage your own agent skills. No registry, no tracking.</strong>
</p>
<p align="center">
  <a href="LICENSE"><img src="https://img.shields.io/github/license/pateroski/skills-manager?style=flat" alt="License"></a>
</p>

## Install

Copy/paste into your CLI prompt:

```text
Install the skills-manager skill from https://github.com/pateroski/skills-manager, refer to the repo's AGENTS.md for setup instructions.
```

Or manually:

```bash
git clone https://github.com/pateroski/skills-manager /tmp/skills-manager-src
cp -R /tmp/skills-manager-src ~/.agents/skills/skills-manager
rm -rf ~/.agents/skills/skills-manager/.git
ln -s ~/.agents/skills/skills-manager/scripts/skills-manager ~/.local/bin/skills-manager
```

Requires [`uv`](https://docs.astral.sh/uv/) and `git`. Also installable with `npx skills add pateroski/skills-manager`.

## What it does

Install, update, remove, and track SKILL.md folders across Claude Code, OpenCode, and any harness that reads `~/.agents/skills` — with a JSON manifest (`skills-manifest.json`) as the single source of truth, so you always know what you have, where it came from, and whether you've edited it.

```bash
skills-manager init -n      # adopt an existing skills folder (preview first)
skills-manager status       # what do I have, where from, any drift?
skills-manager update -n    # what's outdated? (exit 2 if anything)
skills-manager update       # interactive apply
skills-manager add owner/repo
skills-manager remove NAME  # moved to backup, never rm -rf
```

## Channels

| Channel | Upstream | Update behavior |
| --- | --- | --- |
| `git` | any git repo (GitHub, GitLab, gists, local paths) | fetch, diff, interactive apply with commit pinning |
| `cli` | a tool owns the skill set (e.g. hyperframes, plannotator) | delegates to that tool's refresh command, or hands-off |
| `disk` | a path on disk (one-way sync) | syncs when the source is present |
| `local` | none | never touched |

## Why not just re-download?

Because you edit your skills. Every install records a content hash; `update` diffs your disk against it and **asks** before touching anything you changed. `hold` skills require typing `overwrite`. Breaking changes (major bumps, `BREAKING` commits, trigger changes) are flagged and never auto-applied. Nothing is ever deleted — old trees go to `~/.cache/skills-manager/backups/`.

## Untracked skill? Find its upstream

```bash
skills-manager discover --json   # research brief: distinctive phrases, URL hints, queries
skills-manager set-source <name> --repo <URL>
```

Feed the brief to your agent's search tools; record only verified sources.

## Docs

- [SKILL.md](SKILL.md) — the agent-facing instructions
- [AGENTS.md](AGENTS.md) — how an AI should set this up and work on the repo

## License

MIT.
