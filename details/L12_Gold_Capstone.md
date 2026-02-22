# L12: Gold Capstone - Full Autonomous Employee

In Lesson 11, you built the CEO Briefing -- your employee's ability to analyze a week of work and present actionable insights. Now you wire everything together. Every skill, every subagent, every watcher, every approval gate, every MCP connection -- into a single pipeline that runs without you.

A client sends an email at 2 AM requesting an invoice. Your employee wakes up, reads the message, classifies the request, drafts a response with the invoice details, places it in the approval queue, and logs the entire sequence. When you check your phone over morning coffee, you see one notification: "Invoice response drafted for Client A. Approve or reject." You tap approve. The email sends. The dashboard updates. The audit log records every step.

That is a Gold Tier employee. Not more features than Silver -- more trust. Error recovery so transient failures resolve themselves. Audit logging so every action is traceable. Documentation so you (or anyone) can understand and maintain the system.

## The Full Pipeline

Here is every component you built across L01-L11, wired end-to-end:

```
External Event (email arrives, file drops)
 │
 ▼
 ┌─────────┐
 │ Watcher │ (L08: Gmail Watcher or File Watcher)
 └────┬─────┘
 │ creates markdown file
 ▼
 /Needs_Action/request.md
 │
 ▼
 ┌──────────────┐
 │ Orchestrator │ (L07: email-assistant master skill)
 │ classifies   │
 │ delegates    │
 └──────┬───────┘
        │
    ┌───┴──────┐
    │          │
    ▼          ▼
Routine   Sensitive
    │          │
    ▼          ▼
Auto-execute  /Pending_Approval/ (L09: HITL)
              │
              Human reviews
              │
        ┌─────┴──────┐
        │            │
        ▼            ▼
    /Approved/   /Rejected/
        │            │
        ▼            ▼
    Execute    Log rejection
        │
        ▼
 ┌──────────────┐
 │ MCP Action   │ (L06: Gmail MCP sends email)
 └──────┬───────┘
        │
        ▼
/Done/ + /Logs/ + Dashboard.md + git commit
```

Every arrow is a handoff you can verify. Every box is a component you already built. The Gold capstone is wiring them together and proving each handoff works.

## Step 1: Create Dashboard.md

Your employee needs a single status file that shows what is happening right now. Create this in your vault root:

```markdown
# Employee Dashboard

## Status: Active

## Last Updated

2026-01-07T10:30:00Z

## Today's Activity

| Time | Action | Target | Status |
|------|--------|--------|--------|
| 10:30 | email_triage | inbox (12 messages) | complete |
| 10:32 | email_draft | client_a@example.com | pending_approval |

## Pending Approvals: 1

- PAYMENT_Client_A_2026-01-07.md (awaiting human review)

## Tasks in Queue: 0

## Errors Today: 0

## Weekly Summary

- Emails processed: 47
- Responses sent: 12
- Approvals requested: 3
- Errors recovered: 1
```

**Important:** This is a static template. Your orchestrator and watcher scripts do not automatically update Dashboard.md yet. For now, update it manually after each test run. Automating dashboard updates is a Hackathon deliverable (L14).

The orchestrator updates this file after every action. The watcher deposits a timestamp when it starts. HITL approval moves update the "Pending Approvals" count. Errors increment the error counter. This file is your system's heartbeat.

## Step 2: The Invoice Flow -- End-to-End

This walkthrough traces one request through every component. Follow along by creating the trigger file yourself.

### 2a. Simulate the Trigger

Create the file that a watcher would produce:

```markdown
---
source: gmail_watcher
detected: 2026-01-07T02:15:00Z
from: client_a@example.com
subject: "January invoice request"
priority: normal
---

## Message

Hi, can you send me the invoice for January? The project wrapped up last week and I need it for our accounting.

Thanks,
Client A
```

Save as: `~/projects/ai-vault/Needs_Action/EMAIL_client_a_invoice_2026-01-07.md`

### 2b. Orchestrator Classifies and Delegates

Invoke the email-assistant to process the queue:

```bash
claude -p "Check /Needs_Action/ for pending items and process them according to your workflow."
```

Expected behavior:

- Orchestrator reads the file in /Needs_Action/
- Classifies as email response requiring invoice details
- Checks Company_Handbook.md for invoice procedures
- Drafts response using email-drafter skill
- Since sending email is a sensitive action, creates approval file

Verification: Check that these files now exist:

```bash
ls ~/projects/ai-vault/Pending_Approval/
# Should show: EMAIL_REPLY_client_a_2026-01-07.md

ls ~/projects/ai-vault/Plans/
# Should show: PLAN_invoice_client_a.md
```

### 2c. Human Approves

Review the approval file, then approve:

```bash
mv ~/projects/ai-vault/Pending_Approval/EMAIL_REPLY_client_a_2026-01-07.md \
   ~/projects/ai-vault/Approved/EMAIL_REPLY_client_a_2026-01-07.md
```

### 2d. Action Executes

The approval watcher (or next orchestrator cycle) detects the approved file and:

- Sends the email via Gmail MCP
- Moves the original request to /Done/
- Writes an audit log entry to /Logs/
- Updates Dashboard.md
- Commits the state change to git

Verification -- five checks:

```bash
# 1. Original moved to Done
ls ~/projects/ai-vault/Done/ | grep client_a
# Expected: EMAIL_client_a_invoice_2026-01-07.md

# 2. Approval file cleaned up
ls ~/projects/ai-vault/Approved/
# Expected: empty (file processed and archived)

# 3. Log entry written
cat ~/projects/ai-vault/Logs/2026-01-07.json | python3 -m json.tool | grep email_send
# Expected: action_type: "email_send"

# 4. Dashboard updated
grep "client_a" ~/projects/ai-vault/Dashboard.md
# Expected: row showing email_send with status "complete"

# 5. Git commit recorded
git log --oneline -1
# Expected: "auto: processed EMAIL_client_a_invoice_2026-01-07"
```

If all five checks pass, your pipeline works end-to-end.

## Step 3: Error Recovery

A production system encounters failures. The question is not whether errors occur, but whether your employee handles them predictably.

### Error Categories

CategoryExamplesRecovery StrategyHuman Needed?
TransientNetwork timeout, API rate limit, temporary 503Exponential backoff retry (up to 3 attempts)No
AuthenticationExpired OAuth token, revoked API keyPause all operations, write alert to vaultYes
LogicMisclassified email, wrong template selectedRoute to /Needs_Action/ with review flagYes
DataCorrupted markdown, missing required fieldQuarantine file to /Errors/, alert humanYes
SystemProcess crash, disk fullPM2 auto-restart, watchdog alertOnly if persistent

### Build a Retry Wrapper

Create a Python function that handles transient failures. Add this to your watcher or orchestration scripts:

```python
import time
import logging

def retry_with_backoff(func, max_retries=3, base_delay=2):
 """Retry a function with exponential backoff.

 Args:
     func: Callable to retry
     max_retries: Maximum number of attempts
     base_delay: Initial delay in seconds (doubles each retry)

 Returns:
     Result of successful function call

 Raises:
     Last exception if all retries exhausted
 """
 for attempt in range(max_retries):
     try:
         return func()
     except (ConnectionError, TimeoutError) as e:
         if attempt == max_retries - 1:
             logging.error(f"All {max_retries} retries failed: {e}")
             raise
         delay = base_delay * (2 ** attempt)  # 2s, 4s, 8s
         logging.warning(f"Attempt {attempt + 1} failed: {e}. Retrying in {delay}s")
         time.sleep(delay)
```

Usage example:

```python
>>> retry_with_backoff(lambda: fetch_gmail_messages())
# Attempt 1 failed: ConnectionError. Retrying in 2s
# Attempt 2 failed: ConnectionError. Retrying in 4s
# Attempt 3 succeeds, returns messages
```

The delays double each time: 2 seconds, 4 seconds, 8 seconds. This gives transient issues time to resolve without hammering the API.

Save these functions in `~/projects/ai-vault/scripts/error_recovery.py`. Import them in your watcher scripts with `from scripts.error_recovery import retry_with_backoff`. This gives all your scripts a shared retry mechanism.

### Handle Authentication Failures

Authentication errors are not transient -- retrying will not fix an expired token. Your employee must stop and ask for help:

```python
import json
from datetime import datetime
from pathlib import Path

def handle_auth_failure(error, vault_path="~/projects/ai-vault"):
 """Pause operations and alert human about auth failure."""
 vault = Path(vault_path).expanduser()
 alert = {
     "timestamp": datetime.utcnow().isoformat() + "Z",
     "alert_type": "auth_failure",
     "error": str(error),
     "action_taken": "operations_paused",
     "human_action_required": "Re-authenticate and restart watchers"
 }
 alert_path = vault / "Needs_Action" / "ALERT_auth_failure.md"
 alert_path.write_text(
     f"# Authentication Failure\n\n"
     f"**Time:** {alert['timestamp']}\n"
     f"**Error:** {alert['error']}\n\n"
     f"## Required Action\n\n"
     f"1. Check OAuth token expiration\n"
     f"2. Re-run authentication flow\n"
     f"3. Restart watchers with `pm2 restart all`\n"
 )
 # Also write to log
 log_path = vault / "Logs" / f"{datetime.utcnow().strftime('%Y-%m-%d')}.json"
 with open(log_path, "a") as f:
     f.write(json.dumps(alert) + "\n")
```

When Gmail returns 401 Unauthorized:
1. Creates `~/projects/ai-vault/Needs_Action/ALERT_auth_failure.md`
2. Appends to `~/projects/ai-vault/Logs/2026-01-07.json`
3. Operations pause until human resolves

Verification: Test by intentionally invalidating your Gmail token (rename .env temporarily). Run the watcher. Confirm:
- Alert file appears in /Needs_Action/
- Log entry records the failure
- Watcher pauses instead of crashing

## Step 4: Audit Logging

Every action your employee takes must be logged. This is not optional -- without logs, the HITL guarantee is unverifiable.

### Log Entry Structure

Each log entry is a single JSON line appended to the daily log file:

```json
{
 "timestamp": "2026-01-07T10:30:00Z",
 "action_type": "email_send",
 "actor": "claude_code",
 "target": "client_a@example.com",
 "parameters": {
     "subject": "January Invoice - Project Alpha",
     "template": "invoice_response"
 },
 "approval_status": "approved",
 "approved_by": "human",
 "result": "success",
 "duration_ms": 1250
}
```

Every entry must include these required fields:

FieldPurposeExample
timestampWhen the action occurred (UTC ISO 8601)2026-01-07T10:30:00Z
action_typeWhat category of actionemail_send, file_move, triage
actorWho performed the actionclaude_code, gmail_watcher, human
targetWhat was acted uponemail address, file path, inbox
parametersAction-specific detailssubject, template, priority
approval_statusHITL statusauto_approved, approved, rejected, not_required
resultOutcomesuccess, failed, retry_scheduled

### Implement the Logger

```python
import json
from datetime import datetime, timezone
from pathlib import Path

def log_action(action_type, actor, target, parameters=None,
               approval_status="not_required", approved_by=None,
               result="success", vault_path="~/projects/ai-vault"):
 """Append a structured audit log entry."""
 vault = Path(vault_path).expanduser()
 log_dir = vault / "Logs"
 log_dir.mkdir(exist_ok=True)

 today = datetime.now(timezone.utc).strftime("%Y-%m-%d")
 log_path = log_dir / f"{today}.json"

 entry = {
     "timestamp": datetime.now(timezone.utc).isoformat(),
     "action_type": action_type,
     "actor": actor,
     "target": target,
     "parameters": parameters or {},
     "approval_status": approval_status,
     "approved_by": approved_by,
     "result": result,
 }

 with open(log_path, "a") as f:
     f.write(json.dumps(entry) + "\n")

 return entry
```

Usage:

```python
>>> log_action("email_send", "claude_code", "client_a@example.com",
...            parameters={"subject": "January Invoice"},
...            approval_status="approved", approved_by="human")
{'timestamp': '2026-01-07T10:30:00+00:00', 'action_type': 'email_send', ...}
```

### Query Your Logs

Once logs exist, you can answer questions about your employee's behavior:

```bash
# How many emails did the employee send today?
grep '"action_type": "email_send"' ~/projects/ai-vault/Logs/2026-01-07.json | wc -l

# Were there any errors?
grep '"result": "failed"' ~/projects/ai-vault/Logs/2026-01-07.json

# Did any action execute without approval?
grep -v '"approval_status": "not_required"\|"approved"' ~/projects/ai-vault/Logs/2026-01-07.json

# What happened between 2 AM and 6 AM?
grep -E '"timestamp": "2026-01-07T0[2-5]' ~/projects/ai-vault/Logs/2026-01-07.json
```

Verification: Run the full invoice flow from Step 2 again. Then query logs to confirm every step was recorded. You should find entries for: triage, draft, approval_request, email_send, file_move.

## Step 5: Architecture Documentation

Your final deliverable is a README.md that explains the complete system. This is not busywork -- it is the "future you" test. If you return to this project in six months, can you understand and maintain it from this document alone?

Create `~/projects/ai-vault/README.md`. Here is a complete example structure -- adapt it to match your actual components:

```markdown
# Personal AI Employee

An autonomous assistant that monitors email and file inputs, classifies requests,
drafts responses, gates sensitive actions through human approval, and logs every step.

## Architecture

```
External Event (email arrives, file drops)
 │
 ▼
 ┌─────────┐
 │ Watcher │ (Gmail Watcher or File Watcher)
 └────┬─────┘
 │ creates markdown file
 ▼
 /Needs_Action/request.md
 │
 ▼
 ┌──────────────┐
 │ Orchestrator │ (email-assistant master skill)
 │ classifies   │
 │ delegates    │
 └──────┬───────┘
        │
    ┌───┴──────┐
    │          │
    ▼          ▼
Routine   Sensitive
    │          │
    ▼          ▼
Auto-execute  /Pending_Approval/ (HITL)
              │
              Human reviews → /Approved/ or /Rejected/
              │
              ▼
 ┌──────────────┐
 │ MCP Action   │ (Gmail MCP sends email)
 └──────┬───────┘
        │
        ▼
/Done/ + /Logs/ + Dashboard.md + git commit
```

## Components

| Component | Location | Purpose |
|-----------|----------|---------|
| Gmail Watcher | `watchers/gmail_watcher.py` | Monitors inbox, creates action files |
| File Watcher | `watchers/file_watcher.py` | Monitors drop folder |
| Orchestrator | `.claude/skills/email-assistant/` | Routes requests to skills/subagents |
| Email Skills (4) | `.claude/skills/email-*/` | Drafting, templates, summarizing |
| Subagents (3) | `.claude/agents/` | Triage, suggest, track |
| Gmail MCP | `.mcp.json` | External email operations |
| Audit Logger | `scripts/logger.py` | Structured JSON logging |
| Retry Wrapper | `scripts/error_recovery.py` | Exponential backoff |

## Setup and Starting

```bash
# 1. Start watchers
pm2 start watchers/gmail_watcher.py --interpreter python3
pm2 start watchers/file_watcher.py --interpreter python3

# 2. Verify watchers are running
pm2 list

# 3. Persist across reboots
pm2 save && pm2 startup
```

## Operational Procedures

**Daily check (2 min):** Open Dashboard.md. Check error count and pending approvals.

**Weekly review (15 min):** Query logs for patterns --
`grep '"result": "failed"' Logs/*.json` to find recurring failures.

**Troubleshooting:**
- Watcher stopped → `pm2 restart gmail_watcher`
- Auth failure → Check `/Needs_Action/` for ALERT file, re-authenticate, restart
- MCP offline → Verify `.mcp.json` config, restart Claude Code
- Disk full → Archive old `/Done/` files, clear old logs
```

Adapt this to your actual components. Add sections for any additional domains you integrated. The goal is that someone reading only this file can start, operate, and troubleshoot the system.

## Gold Tier Verification Checklist

Run through each requirement and verify it works:

#ComponentRequirementHow to VerifyPass?
1End-to-End PipelineEvent flows from trigger to completionRun Step 2 invoice flow, check all 5 verification points
2Multi-DomainAt least 2 domains integratedEmail + files both trigger orchestrator
3Error RecoveryTransient errors retry with backoffTemporarily block network, verify retry logs
4Graceful DegradationAuth failure pauses with alertInvalidate token, check alert file created
5Audit LoggingEvery action logged as JSONQuery logs after full pipeline run
6HITL VerifiedSensitive actions require approvalCheck logs for approval_status field
724/7 OperationPM2 running, survives process killpm2 list shows online; kill -9 PID, verify restart
8DashboardReal-time status trackingRun action, verify Dashboard.md updates
9DocumentationREADME explains full systemGive README to someone unfamiliar; can they start the system?
10Git SafetyState changes committedgit log shows auto-commits after actions

Mark each row as you verify it. Any unchecked row means your employee is not yet Gold Tier.

## What You Built

Look at what exists in your vault now:

```
ai-vault/
├── .claude/
│   ├── skills/          # 4 email skills including orchestrator
│   └── agents/          # 3 email subagents
├── watchers/            # Gmail + file watchers (PM2 managed)
├── utils/               # retry.py + logger.py
├── Needs_Action/        # Watcher deposits, orchestrator reads
├── Plans/               # Claude's work plans
├── Pending_Approval/    # HITL queue
├── Approved/            # Human-approved actions
├── Rejected/            # Human-rejected actions
├── Done/                # Completed work archive
├── Logs/                # JSON audit trail
├── Accounting/          # Financial tracking
├── Dashboard.md         # System heartbeat
├── Company_Handbook.md  # Governance rules
├── Business_Goals.md    # KPIs and targets
├── CLAUDE.md            # Employee context
├── AGENTS.md            # Governance rules
├── README.md            # Architecture documentation
└── .mcp.json            # Gmail MCP configuration
```

This started as an empty Obsidian vault in Lesson 1. Twelve lessons later, it is a production-grade autonomous system. Skills give it expertise. Subagents give it judgment. Watchers give it perception. HITL gives it safety. Error recovery gives it resilience. Audit logs give it accountability. Documentation gives it maintainability.

That is a Digital FTE.

## Try With AI

### Prompt 1: Full Pipeline Test

I've built my Personal AI Employee with watchers, orchestrator, HITL approval,
Gmail MCP, error recovery, and audit logging. I want to test the complete
pipeline end-to-end.

Create a test scenario: simulate an incoming client email requesting a project
status update. Walk me through each step the system should take, and at each
step, tell me exactly what file should be created or modified and what command
I can run to verify it worked. If any step fails, tell me what to check.

What you're learning: Systematic end-to-end verification. You are learning to trace a request through every component, verify each handoff, and diagnose where failures occur -- the same process production engineers use when debugging distributed systems.

### Prompt 2: Error Recovery Design

My Personal AI Employee needs to handle these failure scenarios:

1. Gmail API returns 429 (rate limited) during a batch triage of 50 emails
2. OAuth token expires at 3 AM while the watcher is running
3. A corrupted markdown file in /Needs_Action/ crashes the orchestrator

For each scenario:
- Classify it (transient, auth, data, logic, or system)
- Design the recovery strategy
- Show me the Python code that implements it
- Show me the audit log entry it should produce

What you're learning: Error classification and recovery design. You are learning to distinguish between errors that resolve themselves (retry), errors that need human intervention (alert and pause), and errors that need quarantine (isolate and report) -- a pattern that applies to any autonomous system, not just email.

Your employee is ready for work. The next step is the chapter assessment (L13), followed by Hackathon 0 (L14) where you build a complete Personal AI Employee project from scratch and submit it as a GitHub repository. Everything you practiced here -- pipeline integration, error recovery, audit logging, documentation -- becomes the quality bar for your hackathon submission.
