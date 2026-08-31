# Codex customizations

Personal Codex skills and hooks maintained by `arviphd`.

## Contents

- `skills/peer-review`: bounded adversarial review with Claude, Gemini, or a Codex subagent
- `skills/simplify`: conservative source-code simplification
- `skills/staged-agent-implementation`: user-gated implementation with fresh subagents
- `hooks/auto-review.py`: post-change review hook
- `hooks/hooks.json`: portable hook configuration

Generated review ledgers, local backups, caches, credentials, sessions, and other Codex state are intentionally excluded.

## Install

Copy or symlink each directory under `skills/` into your user skill directory. This machine uses `~/.codex/skills/`; current Codex documentation also supports the shared user location `~/.agents/skills/`.

For the review hook:

```sh
mkdir -p "$HOME/.codex/hooks"
cp hooks/auto-review.py "$HOME/.codex/hooks/auto-review.py"
cp hooks/hooks.json "$HOME/.codex/hooks.json"
```

The last command replaces any existing user hook configuration. Merge the entries manually if other hooks are already configured.

## Maintenance

Keep reusable source files here. Do not add generated handoff ledgers or machine-local Codex state.
