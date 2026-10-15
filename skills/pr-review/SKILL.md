---
name: pr-review
description: Review someone else's GitHub pull request using Matt's two-axis code-review workflow, with PR context, a recommended verdict, and approval-gated posting.
---

# PR review

Use Matt's `code-review` skill for the review itself. This skill adds GitHub context and posting only; do not run a separate checklist review.

## Workflow

1. Confirm the PR is authored by someone else. Gather its context:

   ```bash
   gh pr view PR_NUMBER --comments
   gh pr checks PR_NUMBER
   gh pr diff PR_NUMBER
   ```

   Read the description, linked issue/spec, test instructions, existing discussion, and relevant surrounding code. Identify the PR's base ref. Check out the PR branch only when needed for local inspection or to run Matt's workflow against its `HEAD`.

2. Run Matt's `code-review` workflow against the PR branch, using the PR base ref as the fixed point. Follow its spec discovery, standards discovery, parallel Standards/Spec reviews, and separate reporting requirements. If the base ref or PR branch cannot be reviewed locally, explain the limitation rather than substituting a different review process.

3. Recommend `approve`, `comment`, or `request changes` based on the findings. Show the complete draft review and wait for explicit approval before posting.

## Posting

After approval, use exactly one GitHub CLI action:

```bash
gh pr review PR_NUMBER --approve --body "..."
gh pr review PR_NUMBER --request-changes --body "..."
gh pr review PR_NUMBER --comment --body "..."
```

Never post, resolve threads, or reply to comments without explicit approval. Keep the review factual, concise, and free of generic praise or AI disclaimers.
