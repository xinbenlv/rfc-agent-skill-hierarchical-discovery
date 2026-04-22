# Agent Skills — Hierarchical Discovery + Remote Sources

- **Status**: Draft · Rev 2 · 2026-04-22
- **Author**: Zainan Victor Zhou (`rfc-agent-skill-hierarchical-discovery@zzn.im`)
- **Targets**: open-source agent harnesses (Codex CLI, OpenCode, others). Prior art: Claude Code's one-level-deep skill convention.

## Problem

Current harnesses discover skills at exactly `<skills-root>/<name>/SKILL.md` — one level deep. This blocks two things: **nested layouts** (an upstream monorepo can't be dropped in) and **remote sources** (every skill has to be committed locally).

## Proposal

### 1. Hierarchical discovery

Treat `SKILL.md` as the skill marker at **any depth**. Walker descends into every dir and registers each `SKILL.md` it finds; once found, that dir is a leaf (no further descent). Dot-dirs (`.git`, `.github`, …) are skipped.

Skill identity = the `name:` field in `SKILL.md` frontmatter. Duplicate names across discovered skills → hard error reporting both paths.

Fully backward compatible: one-level-deep layouts still work because a skill at depth 1 is the same as a skill at depth N — the walker just terminates earlier.

### 2. Remote sources via `skills.config.json`

At the skills root, an optional file:

```json
{
  "version": 1,
  "sources": [
    "https://github.com/vercel-labs/agent-skills",
    "https://github.com/vercel-labs/agent-browser",
    "https://github.com/anthropics/skills#main"
  ]
}
```

For each entry, the harness:
1. Clones into a harness-managed cache (e.g. `~/.cache/agent-skills/<hash>/`), **never into the user's skills root**.
2. Applies the Part-1 hierarchical walker over the clone.
3. Registers every `SKILL.md` it finds, subject to the same name-collision rule.

`#ref` fragment optionally pins a branch/tag/commit. Absent = default branch.

No `id`, no per-skill path, no submodules, nothing to commit. The user declares *which repos to trust*; the harness handles fetching, caching, and updating.

## Backward compatibility

Harnesses that don't implement this spec see a skills root with zero or more local skills (depth 1) and silently ignore `skills.config.json`. Users who want the same behavior against legacy harnesses can run a tiny shim that clones the listed sources and symlinks discovered skills into the one-level-deep layout — unspecified here because it's a local concern, not part of the protocol.

## Security

`skills.config.json` is an allowlist of trusted remote sources. Running skills executes code, so this is equivalent to `curl | sh` in trust scope. Harnesses MUST:
- Pin content-address (commit SHA) after first fetch, or at least surface the resolved SHA to the user.
- Never auto-upgrade without explicit user action (e.g. `skills update`).
- Refuse non-HTTPS / non-SSH URLs.

## Open questions

1. **Cache layout** — harness-specific or standardized (`$XDG_CACHE_HOME/agent-skills/`)?
2. **Non-git sources** — tarball URLs? OCI artifacts? v1 proposes git only.
3. **Subpath scoping** — should an entry restrict to `<repo>#ref:subpath` so a user opts into only part of a monorepo?
4. **Pin file** — separate `skills.lock.json` with resolved SHAs, or inline in `skills.config.json` after first fetch?

## Reference implementation

~100 LOC per harness. Planned targets: Codex CLI, OpenCode. Claude Code (closed) to be filed as a feature request.
