---
on:
  schedule:
    - cron: "0 13 * * 1-5"  # 13:00 UTC, Monday–Friday
  workflow_dispatch:

permissions:
  contents: read

engine: copilot

network:
  allowed:
    - hacker-news.firebaseio.com

tools:
  web-fetch:

safe-outputs:
  create-issue:
    max: 1
---

# Hacker News Daily Digest

Generate a daily digest of top Hacker News stories that are relevant to professional software engineers at large companies, and post the digest as a new GitHub issue.

## Instructions

1. Fetch the top story IDs from `https://hacker-news.firebaseio.com/v0/topstories.json`.
2. Take the first 30 IDs.
3. For each ID, fetch the story details from `https://hacker-news.firebaseio.com/v0/item/<id>.json`.
4. Keep only stories that:
   - Have a `score` greater than 100
   - Are about **software engineering, cloud infrastructure, AI/ML, developer tooling, or distributed systems** (judge by title and URL)
   - Are not job postings (`type` is `story`, not `job`)
5. For each qualifying story produce a row with:
   - **Title** linked to the story's `url` (fall back to the HN discussion link if `url` is missing)
   - **Score**
   - **Comments** count (`descendants`)
   - **Why relevant** — one short sentence explaining why an enterprise developer should care
6. Create a single new issue titled `HN Digest – <YYYY-MM-DD>` (today's date in UTC). The body must be:
   - A short intro sentence
   - A Markdown table with columns: `Title | Score | Comments | Why relevant`
   - A footer line with the HN discussion URL for each story (or include it as a "💬 Discuss" link in the row)
7. If fewer than 3 stories qualify, still create the issue and explain that the day's top stories did not meet the relevance criteria.
