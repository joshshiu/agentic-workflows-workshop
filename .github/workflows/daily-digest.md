---
on:
  schedule:
    - cron: "0 13 * * 1-5"  # 13:00 UTC, Monday–Friday
  workflow_dispatch:

permissions:
  contents: read
  issues: read
  pull-requests: read

engine: copilot

tools:
  github:
    toolsets: [default]

network: defaults

safe-outputs:
  create-issue:
    max: 1
---

# Daily Digest

Generate a daily digest of open issues and pull requests in this repository and post it as a new GitHub issue.

## Instructions

1. List all **open issues** in this repository (exclude pull requests).
2. List all **open pull requests** in this repository.
3. Group both lists by their primary label. Items without any label go into an "Unlabeled" group.
4. For each item include:
   - The title, linked to the issue or PR URL
   - The author as `@username`
   - How long it has been open (e.g. "3 days", "2 weeks")
5. At the top of the digest, include the **total count** of open issues and the **total count** of open pull requests.
6. Create a single new issue with:
   - **Title:** `Daily Digest – <YYYY-MM-DD>` using today's date in UTC.
   - **Body:** the grouped digest in clean Markdown with `## Issues` and `## Pull Requests` sections.
7. If there are no open issues and no open pull requests, still create the issue and state that the repository has no open items today.
