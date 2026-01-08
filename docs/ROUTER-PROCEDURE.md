# Router Tasks Procedure

> Canonical reference for the Jen OS task routing system

## Purpose

Router Tasks is a queue-based system that allows asynchronous task processing. You drop tasks into a Notion database, and automation (or Claude directly) picks them up, processes them, and writes results back.

This decouples "deciding what to do" from "executing it" — you can queue tasks anytime, and they get processed on schedule or on-demand.

## Database Location

- **Database ID:** `f65e7d1486474bd88c0598add298f0d8`
- **URL:** [Router Tasks in Notion](https://www.notion.so/f65e7d1486474bd88c0598add298f0d8)

## Database Schema

| Field | Type | Description |
|-------|------|-------------|
| Task ID | Title | Unique identifier (e.g., `outreach-eman-001`, `diagnostic-001`) |
| Type | Select | Task type — determines what processor handles it |
| Target | Select | Where results go (e.g., `Artifacts`, `CRM`, `Sessions`) |
| Payload | Text (JSON) | Structured data the processor needs |
| Status | Status | Current state in the lifecycle |
| Executed By | Text | Who/what processed the task |
| Execution Notes | Text | Output, results, or error messages |

## Task Types

| Type | Description | Payload Format |
|------|-------------|----------------|
| `WRITE_NOTION` | Create or update a Notion page | `{"title": "...", "type": "...", "content": "..."}` |
| `DRAFT_OUTREACH` | Draft personalized outreach message | `{"target_name": "...", "context": "...", "tone": "..."}` |
| `RESEARCH` | Research a topic and summarize | `{"query": "...", "depth": "shallow|deep"}` |
| `ANALYZE` | Analyze data or content | `{"source": "...", "question": "..."}` |

*Add new types as needed. The processor checks Type and routes accordingly.*

## State Machine

```
PENDING → CLAIMED → DONE
                  ↘ ERROR
```

| Status | Meaning |
|--------|--------|
| `PENDING` | Task is queued, waiting for processing |
| `CLAIMED` | Processor has picked it up, work in progress |
| `DONE` | Successfully completed, results in Execution Notes |
| `ERROR` | Failed, error details in Execution Notes |

## How to Create a Task

1. Open [Router Tasks database](https://www.notion.so/f65e7d1486474bd88c0598add298f0d8)
2. Click **New**
3. Fill in:
   - **Task ID:** Descriptive slug (e.g., `outreach-kris-001`)
   - **Type:** Select appropriate type
   - **Target:** Where results should go
   - **Payload:** JSON with required data
   - **Status:** Set to `PENDING`
4. Save — automation will pick it up

### Example Task

```
Task ID: diagnostic-001
Type: WRITE_NOTION
Target: Artifacts
Payload: {"title": "Pipeline Test", "type": "Document", "content": "If you can read this, the pipeline works."}
Status: PENDING
```

## Processing Methods

### Method 1: GitHub Actions (Automated)

- Runs every 5 minutes via cron
- Queries for PENDING tasks
- Calls Claude API to process
- Writes results back to Notion
- **Setup:** See [SETUP.md](./SETUP.md)

### Method 2: Claude Direct (In Chat)

Ask Claude in chat:
> "Process pending Router Tasks"

Claude will:
1. Query the database for PENDING tasks
2. Claim the task
3. Process it
4. Write results and mark DONE

### Method 3: Claude Code (Local)

Run from terminal:
```bash
claude -p "Check Router Tasks database for PENDING tasks. For each: claim it, process based on Type and Payload, write results to Execution Notes, mark DONE."
```

## Troubleshooting

| Symptom | Cause | Fix |
|---------|-------|-----|
| Tasks stuck in PENDING | Automation not running | Check GitHub Actions logs or run manually |
| Tasks stuck in CLAIMED | Processor crashed mid-task | Manually set back to PENDING or investigate |
| ERROR status | Processing failed | Check Execution Notes for error details |
| No tasks appearing | Database not shared | Share with Notion integration |
| 401 errors | Bad API credentials | Regenerate and update secrets |

## Integration Requirements

1. **Notion Integration** must have access to Router Tasks database
2. **GitHub Secrets** must include:
   - `NOTION_API_KEY`
   - `NOTION_ROUTER_DB_ID`
   - `ANTHROPIC_API_KEY`
3. **Database sharing:** Router Tasks must be explicitly shared with the integration

## Design Principles

- **Idempotent:** Processing same task twice should be safe
- **Atomic:** Each task is self-contained
- **Traceable:** Execution Notes provide audit trail
- **Recoverable:** ERROR tasks can be retried by setting back to PENDING

---

*Last updated: January 8, 2026*
