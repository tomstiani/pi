# ClickUp through codemode

Use Pi's `codemode` tool to call `tools.bash`, parse CLI JSON, and return focused results. Codemode scripts use JavaScript with top-level `await` and `return`; they have no Node APIs or direct filesystem/network access.

## Check capabilities

- Codemode must be available in the session. Offer `pi --tools +codemode` when it is unavailable; use direct tools only after the user agrees to that fallback. Keep unrelated tool choices unchanged.
- Inspect the installed CLI's root help before using `--json`. The new CLI build must be installed before these examples work; the legacy binary does not support the flag.
- Use `describeTool("bash")` if the callable tool or its result contract is unclear. In Pi's built-in codemode, `tools.bash` resolves to `{ output, exit_code, truncated, full_output_path?, wall_time_seconds }`, not message content blocks.

## Read and reduce

1. Run a data command with `--json` through `tools.bash`.
2. Check `exit_code` and `truncated` before parsing `output`. Nonzero exit codes can resolve normally; catching rejected promises alone is insufficient.
3. Parse and validate the expected result shape. If JSON parsing fails, investigate CLI version, stderr, or output-transforming wrappers instead of guessing fields from human text.
4. Return the fields needed for the question. Preserve task IDs, native dependency direction, and pagination state. When analyzing a task for implementation, retain its complete requirements and relevant comments rather than returning only its title.

Example: an overview of at most 20 fetched search matches, with explicit display and API completeness metadata:

```js
const result = await tools.bash({
  command: "clickup-cli task search --json"
});
if (result.exit_code !== 0) {
  return {
    error: "clickup_command_failed",
    exit_code: result.exit_code,
    diagnostic: result.output.slice(0, 1500)
  };
}
if (result.truncated) {
  return {
    error: "clickup_json_truncated",
    full_output_path: result.full_output_path ?? null
  };
}
const data = JSON.parse(result.output);
if (!Array.isArray(data.tasks)) {
  throw new Error("Expected a ClickUp tasks array");
}
const shown = data.tasks.slice(0, 20);
return {
  tasks: shown.map(task => ({
    id: task.id,
    name: task.name,
    status: task.status?.status ?? null
  })),
  fetched_matches: data.tasks.length,
  shown_matches: shown.length,
  display_truncated: shown.length < data.tasks.length,
  api_last_page: data.last_page ?? null
};
```

An omitted task `last_page` means unknown completeness, not an exhaustive result. For document search, retain `next_cursor` and use the same query with `--cursor` when continuing. JSON removes human display caps; it does not add API pages.

When bash reports truncation, process its saved full-output file with a tool or narrow a read-only query. Parse the saved full data, not the clipped JSON string. Return an explicit limitation if a complete result cannot be obtained.

Independent read-only calls may run together in one script. Keep required descriptions, comments, and relationship records available until the current question is answered; avoid printing entire workspace responses just to inspect a few IDs. Never print or store credentials.

## Approval and writes

- Show the complete draft: destination, assignee, title, full description, status, tags, fingerprint and exact creation command. A filtered codemode summary is not a substitute for the approval preview.
- End that script and wait for explicit user approval. Drafting and creation must not happen in one uninterrupted script.
- Execute the exact approved mutation command in a separate codemode call. For guarded creation, preserve the saved draft, fingerprint and configuration directory. Report verified success only from the CLI's verified receipt.
- Codemode is execution routing, not authorization or a transaction. Earlier calls are not undone if a later call fails. After an uncertain write or script failure, reconcile any completed calls before another mutation; do not rerun the script, change draft IDs, switch configuration directories or use the legacy helper to bypass a receipt.

The [native task-creation workflow](native-task-creation.md) defines the creation and recovery steps; these routing rules do not weaken them.
