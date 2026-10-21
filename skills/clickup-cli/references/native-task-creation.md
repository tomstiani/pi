# Native reviewed task creation

Use only when the installed CLI lists `task draft`, `task preview`, and `task create`. This workflow creates one top-level task; it does not publish batches or dependencies.

## Resolve the destination and draft content

1. Inspect the request and relevant code, PR, or discussion. Respect constraints such as “describe the problem, not the solution.” Propose a status appropriate to the work; completed PR work is not automatically “to do.”
2. Discover the workspace, space, and sprint folder using `space search` and `space structure SPACE_ID`. Confirm the intended context before saving defaults:

   ```bash
   clickup-cli config set --team WORKSPACE_ID --space SPACE_ID --sprint-folder FOLDER_ID
   ```

   Flags override environment variables, which override saved defaults. Save only IDs, never credentials. Keep the same `--config PATH` throughout the workflow if overriding the defaults file; its directory also contains creation receipts.
3. Resolve the current sprint and inspect valid fields:

   ```bash
   clickup-cli whoami
   clickup-cli sprint current
   clickup-cli tag list
   clickup-cli list info RESOLVED_LIST_ID
   ```

   Use exact existing tag and status names. Sprint discovery uses API dates inside the configured folder. If no sprint or several match, ask for clarification rather than choosing a different list or folder.
4. Write the complete description to a local Markdown file, then save a new draft:

   ```bash
   clickup-cli task draft "Task title" \
     --description-file /tmp/task.md \
     --status "to do" \
     --assign-me --tags frontend \
     --out /tmp/task-draft.json
   ```

   Drafting performs read-only API calls. Choose a fresh output path; existing drafts are not overwritten.

## Approval and creation

1. Run `clickup-cli task preview /tmp/task-draft.json`. Present the full title, description, workspace/space, sprint folder/list, assignee, status, tags, and the exact command containing the returned fingerprint:

   ```bash
   clickup-cli task create --draft /tmp/task-draft.json \
     --approve APPROVED_SHA256 --yes
   ```

2. Wait for explicit approval of that preview and command. `--yes` and a fingerprint are CLI inputs, not user authorization. Every draft edit requires a new preview and renewed approval.
3. Run the approved command once, using the same configuration directory. The CLI revalidates live selections and verifies the created task. Report success only when it returns `state: verified`, including the task ID and URL.

## Changed or uncertain outcomes

- Changed sprint, schedule, identity, invalid status, or removed tag: resolve the discrepancy; obtain renewed approval if the draft changes. Keep the reviewed destination explicit.
- `created_unverified`: report the retained task ID and verification error. Inspect that existing task, not a replacement. Markdown comparison is intentionally conservative.
- `unknown` or `attempting` receipt: stop and reconcile with the operator. Inspect the destination for an existing task before any further creation.
- Existing receipt: creation is guarded against another attempt, including across local processes or renamed draft files. Keep the receipt and configuration directory intact; do not use the legacy helper or another draft to bypass the guard.

Receipts are local files under `receipts/` beside the selected defaults file. They are not server-side idempotency keys and do not coordinate across machines.
