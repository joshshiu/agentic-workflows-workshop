---
on:
  slash_command:
    name: digest-repo

permissions:
  contents: read
  issues: read
  pull-requests: read

engine: copilot

network: defaults

tools:
  github:
    toolsets: [default]

safe-outputs:
  add-comment:
    max: 1
---

# On-Demand Repo Digest (ChatOps)

ChatOps slash command. A user posts a comment that starts with `/digest-repo <owner>/<repo>`. Produce a digest of open issues and pull requests for that repository and reply in the same thread.

## Instructions

1. Read the body of the comment that triggered this run. It begins with `/digest-repo`.
2. Parse the `<owner>/<repo>` argument that follows `/digest-repo`.
   - If the argument is missing or does not match `<owner>/<repo>`, reply with `Usage: /digest-repo <owner>/<repo>` and stop.
   - If no `<owner>/<repo>` is given, default to the current repository.
3. Using the GitHub tools, list:
   - All **open issues** in that repo (exclude pull requests).
   - All **open pull requests** in that repo.
4. Cap each list at the most recent 50 items if there are more, and note the cap in the output.
5. Group both lists by primary label. Items without labels go into an "Unlabeled" group.
6. For each item include:
   - Title, linked to its URL
   - Author as `@username`
   - How long it has been open (e.g. "3 days", "2 weeks")
7. Produce a Markdown reply with:
   - `## 📊 Repo Digest — <owner>/<repo>` heading
   - **Totals line:** `**<N> open issues** · **<M> open pull requests**` (plus a stars/forks line if easily available from the repo metadata; otherwise omit).
   - `### Issues` section with one subsection per label group.
   - `### Pull Requests` section with one subsection per label group.
   - If there are no open items at all, write "🎉 This repo has no open issues or pull requests."
8. Use the `add-comment` safe output to post the reply on the triggering issue. Do not post multiple comments.
9. If you cannot access the target repository (404, private), reply with a short error message and stop.
