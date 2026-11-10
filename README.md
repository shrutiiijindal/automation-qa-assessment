# automation-qa-assessment
Automation &amp; QA Developer take-home assessment
# Task 2: n8n Morning Brief Workflow

## What it does
Every hour, the workflow fetches the top stories from Hacker News, looks up
details for the top 5, labels the popular ones, and posts one digest to Discord.

## APIs used and why
- **Hacker News API** (hacker-news.firebaseio.com), free, no API key needed.
  - Call 1: `topstories.json` returns a list of story IDs.
  - Call 2: `item/{id}.json` returns the title, score, link and comment count
    for each ID. The first call only gives numbers, so the second call is
    needed to make the digest readable (the "enrichment" step).
- **Discord webhook** for the output, because it is free and quick to set up.

## Workflow steps
1. **Schedule Trigger**: runs every 1 hour.
2. **HTTP Request (top IDs)**: fetches about 500 story IDs.
3. **Code node (Keep Top 5)**: keeps the first 5 IDs and outputs one item per ID.
4. **HTTP Request (details)**: runs once per ID to fetch the story details.
5. **IF node (score > 200)**: stories above 200 points go to the "hot" path and
   get a 🔥 label; others get a normal label. I chose 200 as a rough line between
   a clearly popular story and an average front-page story. It is easy to change.
6. **Merge + Code node (Build Digest)**: joins both paths and builds one message.
7. **Discord node**: posts the digest.

## Error handling
- Both HTTP nodes retry 3 times (2 seconds apart) before failing.
- Both use **On Error → Continue (using error output)**, so a failed call does
  not crash the workflow. The error output goes to a **Send Failure Alert**
  Discord node, which posts a ⚠️ message with the error details.
- I tested this by breaking the URL on purpose, and the alert arrived in Discord.
- Known limitation: if the details call fails for several stories, several
  alerts could be sent. A future improvement would combine them into one.

## Credentials
The Discord webhook URL is stored in n8n's Credentials store. The exported JSON
contains no secrets. After importing, select your own Discord credential on
the two Discord nodes.

## Files
- `Task2_Workflow_Shruti Jindal.json`: exported workflow
- `canvas.png`: the workflow canvas
- `output.png` `output2.png` : a successful run with output data
