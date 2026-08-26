# skills-manager

Manage your own agent skills — install, update, remove, and track SKILL.md
folders across Claude Code, OpenCode, and any harness that reads
`~/.agents/skills` — without depending on a registry or being tracked.

A JSON manifest (`skills-manifest.json`, written next to your skills) is the
single source of truth: per skill it records the **channel** it came from,
the upstream repo, the pinned commit, and a content hash of what was
installed, so your local edits are always detected before an update can
clobber them.

## Channels

| Channel | Upstream | Update behavior |
| --- | --- | --- |
| `git` | any git repo (GitHub, GitLab, gists, local paths) | fetch, diff, interactive apply with commit pinning |
| `cli` | a tool owns the skill set (e.g. hyperframes, plannotator) | delegates to that tool's refresh command, or hands-off |
| `monorepo` | a path on disk (one-way sync) | syncs when the source is present |
| `local` | none | never touched |

## Install

Requires [`uv`](https://docs.astral.sh/uv/) and `git`. Pick one:

```bash
# with skills-manager itself (once you have it)
skills-manager add pateroski/skills-manager

# with vercel-labs skills
npx skills add pateroski/skills-manager

# or plain clone + copy
git clone https://github.com/pateroski/skills-manager
cp -R skills-manager ~/.agents/skills/skills-manager
ln -s ~/.agents/skills/skills-manager/scripts/skills-manager ~/.local/bin/skills-manager
```

## Quick start

```bash
# adopt an existing skills folder (nothing written until you confirm)
skills-manager init -n                # preview classification
skills-manager init                   # adopt for real

# day 2
skills-manager status                 # what do I have, where from, any drift?
skills-manager update -n              # what's outdated? (exit 2 if anything)
skills-manager update                 # interactive apply
skills-manager add owner/repo         # install a new skill (repos, URLs, gists, local paths)
skills-manager remove NAME            # moved to backup, never rm -rf
```

## Safety model

- **Local edits win by default.** Every install records a tree hash; `update`
  compares disk against it and shows a diff + asks before touching anything
  you changed. `hold` skills additionally require typing `overwrite`.
- **Breaking-change advisories**: major version bumps, `BREAKING`/`!:`
  commits, SKILL.md trigger changes, and CHANGELOG edits are flagged and
  excluded from `-y` auto-apply.
- **Nothing is deleted**: removals and overwrites move the old tree to
  `~/.cache/skills-manager/backups/`.
- `-n` never writes outside the clone cache.

## Finding where an untracked skill came from

`skills-manager discover --json` emits a research brief (distinctive phrases,
URL hints, suggested queries) for each unsourced skill. Feed it to your
agent's search tools (web search, GitHub code search, grep.app), verify the
candidate, then record it with
`skills-manager set-source <name> --repo <URL>`.

See [SKILL.md](SKILL.md) for the full agent-facing instructions.

## License

[MIT](LICENSE)
