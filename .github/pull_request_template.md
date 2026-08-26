# Pull Request

## What's the focus of this PR?
<!-- Provide a clear description of what this PR does and why it's needed -->
…

## How to review this PR?
<!-- Guide reviewers through the changes and what to focus on -->
…

## Screenshots/Recordings
<!-- If applicable, add terminal output or recordings to help explain your changes -->

```text
Paste relevant CLI output here
```

## Testing Instructions
<!-- Step-by-step guide for reviewers to test the changes -->
1. Checkout the branch (requires `uv` and `git`)
2. Run `uv run --script scripts/skills-manager --help`
3. Rehearse against a copy: `cp -R ~/.agents/skills /tmp/skills-test && uv run --script scripts/skills-manager <command> --dir /tmp/skills-test -n`
4. Verify that...

## Related Issues

<!-- Link GitHub issues, e.g. Closes #12 — remove if none -->

## Type of changes

- [ ] 🐛 Bugfix
- [ ] ✨ New feature
- [ ] 💥 Breaking change
- [ ] 🧪 Only tests added
- [ ] ♻️ Refactor or Tech debt's pay down
- [ ] 📝 Documentation update
- [ ] ⚡ Performance optimization

## General Checklist

- [ ] The PR relates to _only_ one subject with a clear title
- [ ] I have bumped the `version` in SKILL.md frontmatter if behavior changed
- [ ] I have made corresponding changes to SKILL.md / README.md
- [ ] New commands or flags are covered by the Testing Instructions

## Impact Analysis

- [ ] Breaking changes documented (manifest schema, command surface, exit codes)
- [ ] Manifest migration needed (`manifestVersion` bump)
- [ ] Changes affect how agents invoke the skill (SKILL.md description/triggers)

## Additional Context
<!-- Any design decisions, trade-offs, or alternative solutions considered -->
…
