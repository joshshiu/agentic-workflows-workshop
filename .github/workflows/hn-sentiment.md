---
on:
  slash_command:
    name: hn-sentiment

permissions:
  contents: read
  issues: read
  pull-requests: read

engine: copilot

network:
  allowed:
    - hacker-news.firebaseio.com

tools:
  web-fetch:
  github:
    toolsets: [default]

safe-outputs:
  add-comment:
    max: 1
---

# HN Sentiment Analysis (ChatOps)

A ChatOps slash command. Activate **only** when the comment body starts with `/hn-sentiment`. For any other comment, do nothing.

## Instructions

1. Use the GitHub tools to fetch the body of the issue comment that triggered this run (it is the comment whose creation invoked the `command` trigger named `hn-sentiment`). The comment body begins with `/hn-sentiment`.
2. Parse the URL argument that follows `/hn-sentiment`. Expected form: `https://news.ycombinator.com/item?id=<id>`.
   - If the URL is missing or the `id` query parameter cannot be parsed as a number, reply to the issue with a short error message explaining the expected usage: `/hn-sentiment https://news.ycombinator.com/item?id=12345`, and stop.
4. Fetch the story metadata from `https://hacker-news.firebaseio.com/v0/item/<id>.json`.
   - If the response is `null` or has `deleted: true`, reply with "Hacker News item `<id>` not found or deleted." and stop.
5. Collect up to **50 top-level comments** by walking the story's `kids` array and fetching each child from `https://hacker-news.firebaseio.com/v0/item/<kid>.json`. Skip comments that are `deleted` or `dead`.
6. Classify each comment's `text` (strip HTML tags) as **Positive**, **Negative**, or **Neutral**.
7. Compute the count and percentage of each category.
8. Pick the **top 3 most positive** and **top 3 most negative** comments. For each, extract a clean excerpt of up to ~200 characters (no HTML).
9. Reply to the issue thread with a Markdown comment that includes:
   - A heading: `## 🔍 Hacker News Sentiment Analysis`
   - `**Story:** <story title>` linked to the HN discussion URL
   - A summary table with columns `Sentiment | Count | Percentage` using emoji (😊 Positive / 😐 Neutral / 😠 Negative)
   - An "Overall" verdict line (e.g. **Overall: Mostly Positive**)
   - A `### Top Positive Comments` section with three blockquoted excerpts
   - A `### Top Negative Comments` section with three blockquoted excerpts
10. Use the `add-comment` safe output to post the reply. Do not post multiple comments.
