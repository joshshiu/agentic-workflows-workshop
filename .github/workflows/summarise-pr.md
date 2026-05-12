---
on:
  slash_command:
    name: summarise-pr

permissions:
  contents: read
  issues: read
  pull-requests: read

engine: copilot

network: defaults

tools:
  web-fetch:
  github:
    toolsets: [default]

safe-outputs:
  add-comment:
    max: 1
---

# Summarise PR (ChatOps)

ChatOps slash command. A user posts a comment that starts with `/summarise-pr <pr-url>`. Produce a concise structured summary of the pull request and reply in the same thread.

## Instructions

1. Read the body of the comment that triggered this run. It begins with `/summarise-pr`.
2. Parse the URL argument that follows `/summarise-pr`. Expected form: `https://github.com/<owner>/<repo>/pull/<number>`.
   - If the URL is missing, malformed, or does not match that pattern, reply with a short usage message: `Usage: /summarise-pr https://github.com/<owner>/<repo>/pull/<number>` and stop.
3. Fetch the pull request using the GitHub tools (`get_pull_request` style). You need at minimum: title, author, state (open/merged/closed), base/head branches, body, labels, number of additions/deletions, number of files changed, and the list of changed files.
4. Fetch the **diff** for the PR. If it is very large, sample: include only the file paths and a short hunk excerpt for the top 10 files by change size.
5. Fetch up to the latest **10 issue comments** and **10 review comments** for context. Skip bot comments where possible.
6. Produce a Markdown summary with these sections, in order:
   - `## 📋 PR Summary` heading
   - **Header line:** `**<title>** — [#<number>](<pr-url>) by @<author> · <state>`
   - `### What this PR does` — 2–4 sentence plain-English summary derived from the title, body, and diff. Focus on intent, not line counts.
   - `### Changes at a glance` — a small Markdown table with columns `Metric | Value`: files changed, additions, deletions, labels.
   - `### Key files` — bulleted list of the top changed files with one short note each on what changed in that file (e.g. "added new auth middleware", "refactored to async").
   - `### Risks & callouts` — bullets covering anything notable: schema migrations, breaking API changes, missing tests, large blast radius, security-sensitive paths (auth, crypto, secrets, IAM), TODOs left in the diff. If nothing stands out, write "No obvious risks flagged."
   - `### Discussion highlights` — 1–3 bullets summarising the most important points from comments/reviews. Omit the section if there are no substantive comments.
7. Use the `add-comment` safe output to post the reply on the triggering issue. Do not post multiple comments.
8. If fetching the PR fails (404, private repo you cannot access, etc.), reply with a friendly error explaining what failed and stop.
---
on:
  slash_command:
    name: summarise-pr

permissions:
  contents: read
  issues: read
  pull-requests: read

engine: copilot

tools:
  github:
    toolsets: [default]

safe-outputs:
  add-comment:
    max: 1
---

# Summarise PR (ChatOps)

A ChatOps slash command. When a user posts `/summarise-pr <pr-url>` in any issue or pull request comment, fetch the referenced pull request and reply with a structured AI summary.

## Instructions

1. Use the GitHub tools to read the issue comment that triggered this run. Its body begins with `/summarise-pr`.
2. Parse the pull request URL that follows. Accept either of these forms:
   - `https://github.com/<owner>/<repo>/pull/<number>`
   - `<owner>/<repo>#<number>`
   - A bare number like `#123` — in that case use the current repository (`github.repository`).
3. If no URL/number can be parsed, reply to the issue with a short usage error: `Usage: /summarise-pr https://github.com/owner/repo/pull/123` and stop.
4. Using the GitHub tools, fetch:
   - The pull request title, author, base/head branches, state (open/closed/merged), draft flag.
   - The PR description body.
   - The list of changed files with additions/deletions.
   - Up to the **20 most recent review comments and issue comments** on the PR.
   - The list of review states (approved / changes_requested / commented) per reviewer.
5. Produce a Markdown summary with this structure and reply via `add-comment`:

   ```markdown
   ## 🔎 PR Summary: <title>

   **Repo:** <owner>/<repo> · **PR:** [#<number>](<pr-url>) · **State:** <state> · **Author:** @<user>
   **Branches:** `<head>` → `<base>` · **Files changed:** <N> (+<add> / -<del>)

   ### What this PR does
   <2–4 sentence plain-English summary inferred from title, description, and diff>

   ### Notable changes
   - <bullet per major area — group by directory or feature>
   - …

   ### Review status
   - ✅ Approvals: <count> (<@user, …>)
   - 🟡 Changes requested: <count> (<@user, …>)
   - 💬 Commented: <count>

   ### Discussion highlights
   - <2–4 bullets summarising the most substantive comments — quote sparingly>

   ### Suggested next step
   <one short sentence: ready to merge / needs review / address comments / etc.>
   ```

6. Keep the reply under ~500 words. Do not paste the diff. Do not invent reviewers or comments — only summarise what the GitHub API actually returns.
