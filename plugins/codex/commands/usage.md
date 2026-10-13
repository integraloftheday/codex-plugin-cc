---
description: Show Codex usage levels and rate limit reset times
argument-hint: '[--json]'
disable-model-invocation: true
allowed-tools: Bash(node:*)
---

!`node "${CLAUDE_PLUGIN_ROOT}/scripts/codex-companion.mjs" usage "$ARGUMENTS"`

Present the command output to the user as-is. Do not summarize or add extra prose.
