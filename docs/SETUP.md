# Router Orchestrator Setup

This GitHub Action automatically processes PENDING tasks from the Jen OS Router Tasks database.

## Required Secrets

You must add these secrets to your GitHub repository:

### 1. NOTION_API_KEY
- Go to [notion.so/my-integrations](https://notion.so/my-integrations)
- Create integration or copy existing token
- Starts with `ntn_` or `secret_`

### 2. NOTION_ROUTER_DB_ID
- This is the Router Tasks database ID
- Value: `f65e7d1486474bd88c0598add298f0d8`

### 3. ANTHROPIC_API_KEY
- Go to [console.anthropic.com/settings/keys](https://console.anthropic.com/settings/keys)
- Create or copy API key
- Starts with `sk-ant-`

## How to Add Secrets

1. Go to your repo: https://github.com/educence/jen-os-n8n-workflows
2. Click **Settings** (tab)
3. Click **Secrets and variables** → **Actions** (left sidebar)
4. Click **New repository secret**
5. Add each secret:
   - Name: `NOTION_API_KEY` / Value: your token
   - Name: `NOTION_ROUTER_DB_ID` / Value: `f65e7d1486474bd88c0598add298f0d8`
   - Name: `ANTHROPIC_API_KEY` / Value: your key

## Testing

1. Add secrets as described above
2. Go to **Actions** tab in your repo
3. Click **Router Orchestrator** workflow
4. Click **Run workflow** → **Run workflow**
5. Watch the run — it should process any PENDING tasks

## Schedule

By default, runs every 5 minutes. To change:
- Edit `.github/workflows/router-orchestrator.yml`
- Modify the cron expression
- Examples:
  - `*/5 * * * *` = every 5 minutes
  - `*/15 * * * *` = every 15 minutes
  - `0 * * * *` = every hour
  - `0 9 * * *` = daily at 9am UTC

## How It Works

1. **Query**: Fetches one PENDING task from Router Tasks
2. **Claim**: Marks task as CLAIMED, sets Executed By
3. **Process**: Sends payload to Claude API
4. **Complete**: Writes result to Execution Notes, marks DONE

## Troubleshooting

| Issue | Cause | Fix |
|-------|-------|-----|
| Workflow not running | Secrets missing | Add all 3 secrets |
| 401 error on Notion | Bad API key | Regenerate Notion token |
| 401 error on Claude | Bad API key | Check Anthropic key |
| No tasks processed | None PENDING | Create a PENDING task in Notion |
