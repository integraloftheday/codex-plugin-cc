# Local patches to openai/codex-plugin-cc v1.0.6

These are not upstream. A plugin update overwrites them; re-apply if needed.

## 1. Auto-reviewed escalation for write tasks (2026-09-28)

**File:** `scripts/lib/codex.mjs`, functions `buildThreadParams` and `buildResumeParams`. The original is saved as `scripts/lib/codex.mjs.orig`.

**Why:** stock v1.0.6 hard-codes `approvalPolicy: "never"`. The app-server client (`scripts/lib/app-server.mjs`, `handleServerRequest`) also rejects every server→client request with -32601. Together these mean Codex can never escalate out of its sandbox. That sandbox hides `/dev/nvidia*`, so GPU work and some git writes were impossible from `/codex` tasks.

**Change** (modelled on upstream PR #494, which builds on #426 and closes #445/#412):

```js
approvalPolicy: options.approvalPolicy ?? ((options.sandbox ?? "read-only") !== "read-only" ? "on-request" : "never"),
approvalsReviewer: options.approvalsReviewer ?? ((options.sandbox ?? "read-only") !== "read-only" ? "auto_review" : null),
```

- Write-capable tasks (`--write`, sandbox `workspace-write`) may request escalation. Codex's automatic reviewer (`auto_review`) decides, so no client round trip is needed.
- Read-only tasks keep `never` and no reviewer, so they stay fail-closed as upstream.
- Valid reviewer values (Codex CLI 0.155.1): `"user" | "auto_review" | "guardian_subagent"`.
- This does **not** select `danger-full-access`. Codex must still request escalation per command, and Luna only does so when told explicitly (e.g. "the sandbox hides /dev/nvidia*; request escalated permissions for CUDA commands").

**Verified 2026-09-28** (run by the user): write task, escalation approved, `nvidia-smi` saw the RTX A5000, `torch.cuda.is_available()` returned `True`.

**Revert:** `cp scripts/lib/codex.mjs.orig scripts/lib/codex.mjs`

Related upstream work: PR #494, PR #426, issue #445, issue #145/PR #147 (`--full-access`), PR #742 (`--sandbox <mode>`).
