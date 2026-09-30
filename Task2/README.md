# Task 2 - n8n API Integration Workflow ("Morning Brief")

**Author:** Uday Upadhyay
**Files:** `Task2_Workflow_UdayUpadhyay.json` (exported from n8n), screenshots in this folder.

## What it does
Every hour the workflow finds the most-starred GitHub repositories for the topic `automation`, enriches the top one with its language breakdown, and posts a short digest to a Discord channel.

## Flow
```
Every 1 hour -> Search GitHub repos -> Top 5 repos -> Top repo languages -> Build digest
             -> IF top repo stars > 100000 -> Send HOT digest / Send normal digest
Search GitHub repos (error output) -> Format error message -> Send error alert
```

## APIs used and why
- **GitHub REST API - search repositories** (`/search/repositories?q=topic:automation&sort=stars&order=desc`): free, needs no API key, and returns stars, so it is easy to sort, filter and apply a threshold.
- **GitHub REST API - repo languages** (`/repos/{owner}/{repo}/languages`): a second endpoint of the same API, used to enrich the top repository with its language mix.
- **Discord webhook** for the output, because it is free and quick to set up.

## Transformation
A Code node keeps only the top 5 results and only the fields needed (name, stars, URL, description). A second Code node combines them with the language percentages (top 3 languages) into one readable message.

## Conditional branch
The IF node checks whether the top repo has **more than 100,000 stars**. If yes, the message is sent as "HOT"; if not, as a normal "Daily digest". I chose 100k as a "mega-popular" line; changing the number in the IF node shows the other branch.

## Error handling
- Both HTTP Request nodes use **On Error: Continue (using error output)**, so a failed call never stops the workflow silently.
- If the **GitHub search** fails (rate limit, network, API down), the error output goes to `Format error message` and then `Send error alert`, which posts the reason to Discord.
- If the **languages** call fails, the digest is still sent, with the line "Languages: unavailable (language API call failed)" as a fallback.
- Any other failure still shows as a failed run in the n8n Executions list.

## Credentials
No secret is stored in the workflow. The Discord webhook URL lives in n8n's **Credentials** store (`Discord Webhook account`) and the Discord nodes only reference it. The GitHub calls are unauthenticated (60 requests/hour per IP, plenty for one run per hour).

## How to run
1. In n8n, import `Task2_Workflow_UdayUpadhyay.json`.
2. Create a **Discord Webhook** credential with your own webhook URL and select it in the three Discord nodes.
3. Click **Execute workflow**.

## Screenshots
- `Task2_canvas.png` - workflow canvas (after a successful run)
- `Task2_execution_success.png` - successful execution in the Executions tab
- `Task2_execution_output.png` - output data of the `Send HOT digest` node (digest text, `success: true`)
- `Task2_discord_message.png` - the digest as received in Discord
- `Task2_error_path_canvas.png` - error test: with a deliberately broken URL the error output runs `Format error message` and `Send error alert` instead of crashing
