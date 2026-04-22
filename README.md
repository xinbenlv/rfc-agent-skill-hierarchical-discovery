# rfc-agent-skill-hierarchical-discovery

Draft RFC proposing **hierarchical `SKILL.md` discovery** and **remote skill sources** for open-source agent harnesses (Codex CLI, OpenCode, and others).

> Today most harnesses look for skills at exactly `<skills-root>/<name>/SKILL.md` — one level deep. This proposal generalises discovery to any depth, adds a minimal `skills.config.json` for pulling skills from remote git repos, and stays fully backward compatible.

See **[RFC.md](RFC.md)** for the full proposal.

## Status

Draft. Comments welcome — open an issue or a PR. Major revisions bump the `Rev` header at the top of `RFC.md`.

## Targets

- [Codex CLI](https://github.com/openai/codex) — OSS, reference-implementation target
- [OpenCode](https://github.com/opencode-ai/opencode) — OSS, reference-implementation target
- [Claude Code](https://code.claude.com/) — closed source; to be filed as a feature request
- Any other harness adopting a skills convention

## Author

Zainan Victor Zhou (`rfc-agent-skill-hierarchical-discovery@zzn.im`)
