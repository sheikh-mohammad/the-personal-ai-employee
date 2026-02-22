# L09: Trust But Verify

In L08, you gave your employee senses -- watchers that detect new emails and files arriving. Now your employee notices things on its own. But noticing and acting are different responsibilities. A junior employee can schedule meetings and order office supplies without asking. But signing contracts, approving expenses over a threshold, or emailing the board? Those go through a manager. Your AI employee needs the same governance structure: clear boundaries defining what it can do autonomously and what requires your explicit sign-off.

You will build a folder-based approval workflow where your employee writes requests to a /Pending_Approval/ folder, waits for you to move them to /Approved/ or /Rejected/, and only then acts (or logs the rejection). By the end, you will have a working safety gate that you can test end-to-end.

## The Trust Spectrum

Not every action carries the same risk. Reading a file is harmless. Deleting one could be catastrophic. The goal is not to approve everything -- that defeats the purpose of automation. The goal is to draw the line in the right place.

Full Auto Full Manual
 |------------|------------|------------|------------|
 Read files Reply to Send email Delete Wire
 Analyze known to new files payment
 Summarize contacts contacts outside to new
 vault recipient

Your permission boundaries define where the line falls for each action category.

## Permission Boundaries

This table defines what your employee can do on its own versus what needs your sign-off. These boundaries go into your AGENTS.md file so every component in your system respects them.

Action CategoryAuto-ApproveRequire Approval
Email repliesTo known contacts (3+ prior exchanges in 90 days)New contacts, bulk sends, emails to executives
PaymentsUnder $50 to recurring payeesAll new payees, any amount over $100, any amount $50-100 to recurring
Social mediaScheduled posts (pre-approved content calendar)Replies to comments, DMs, unscheduled posts
File operationsCreate, read, copy within vaultDelete, move outside vault, rename shared files
CalendarAccept meetings during work hoursDecline meetings, reschedule, create meetings with external attendees

Add this table to your AGENTS.md:

## Permission Boundaries

| Action | Auto-Approve | Require Approval |
| --------------- | ------------------------ | ------------------------------ |
| Email replies | Known contacts | New contacts, bulk, executives |
| Payments | < $50 recurring payees | New payees, > $100 |
| Social media | Scheduled posts | Replies, DMs |
| File operations | Create, read | Delete, move outside vault |
| Calendar | Accept during work hours | Decline, reschedule, external |

### Known Contact Definition

A "known contact" is someone you have exchanged 3+ emails with in the past
90 days AND whose address is in your contacts. Everyone else requires
draft review before sending.

## The Approval Workflow

The workflow uses four folders. If you completed L08, you already have /Needs_Action/. Now add three more:

ai-vault/
├── Needs_Action/ ← Watchers deposit items here (L08)
├── Pending_Approval/ ← Employee requests approval here (NEW)
├── Approved/ ← You move files here to approve (NEW)
├── Rejected/ ← You move files here to reject (NEW)
├── Done/ ← Processed items archived here (NEW)
└── Logs/ ← Execution logs stored here (NEW)

Create the folders:

mkdir -p ~/projects/ai-vault/{Pending_Approval,Approved,Rejected,Done,Logs}

The flow works like this:

Employee detects sensitive action
 │
 ▼
Writes request to /Pending_Approval/
 │
 ▼
Human reviews the request
 │
 ┌────┴────┐
 ▼ ▼
/Approved/ /Rejected/
 │ │
 ▼ ▼
Execute Log rejection
 │ (no action taken)
 ▼
/Done/
(archived with log)

## Approval Request Format

Each approval request is a Markdown file with YAML frontmatter. The frontmatter contains structured metadata that a script can parse. The body contains human-readable context.

Create a sample approval request to test the workflow:

```markdown
---
type: approval_request
action: send_email
target: "new-client@example.com"
subject: "Project Proposal - Q1 Engagement"
reason: "New client contact requesting project proposal. Not in known contacts list."
created: "2026-01-15T10:30:00Z"
expires: "2026-01-16T10:30:00Z"
status: pending
---

## Proposed Action

Send email to new-client@example.com with subject "Project Proposal - Q1 Engagement"

## Draft Content

Hi Alex,

Thank you for your interest in our services. I have attached our standard
project proposal for Q1 engagement. The key deliverables include...

[Draft continues]

## Why This Needs Approval

This is a NEW contact (first email exchange). Per permission boundaries,
emails to new contacts require human review before sending.

## To Approve

Move this file to the /Approved/ folder.

## To Reject

Move this file to the /Rejected/ folder.
```

Required frontmatter fields:

FieldPurposeExample
typeAlways approval_requestapproval_request
actionWhat the employee wants to dosend_email, payment, delete_file
targetWho or what is affectedclient@example.com, Vendor Inc, /reports/q4.xlsx
reasonWhy this action is neededNew client, not in known contacts
createdWhen the request was generatedISO 8601 timestamp
expiresWhen the request becomes staleISO 8601 timestamp (typically 24h later)
statusCurrent statepending, approved, rejected

## Building the Approval Watcher

The approval watcher is a Python script that monitors the /Approved/ folder. When you move a file there, the watcher detects it, logs the action, and moves the file to /Done/.

Create the watcher script:

```python
#!/usr/bin/env python3
"""
Approval Watcher - monitors /Approved/ folder for approved actions.

Workflow:
1. Scans /Approved/ for .md files
2. Logs each approved action with timestamp
3. Moves processed files to /Done/
4. Repeats every 5 seconds

Also scans /Rejected/ to log rejections (no action taken).
"""

import time
import datetime
import shutil
from pathlib import Path

# Configure paths relative to your vault
VAULT = Path.home() / "projects" / "ai-vault"
APPROVED_DIR = VAULT / "Approved"
REJECTED_DIR = VAULT / "Rejected"
DONE_DIR = VAULT / "Done"
LOGS_DIR = VAULT / "Logs"

def log_action(filename: str, status: str, details: str = "") -> None:
 """Append a timestamped entry to today's log file."""
 log_file = LOGS_DIR / f"{datetime.date.today()}.log"
 timestamp = datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")
 entry = f"[{timestamp}] {status}: {filename}"
 if details:
 entry += f" | {details}"
 with open(log_file, "a") as f:
 f.write(entry + "\n")
 print(entry)

def process_approved() -> None:
 """Process all files in /Approved/ directory."""
 for filepath in APPROVED_DIR.glob("*.md"):
 log_action(filepath.name, "APPROVED_AND_EXECUTED")
 # Move to Done
 dest = DONE_DIR / filepath.name
 shutil.move(str(filepath), str(dest))
 print(f" -> Archived to Done/{filepath.name}")

def process_rejected() -> None:
 """Log all files in /Rejected/ directory (no action taken)."""
 for filepath in REJECTED_DIR.glob("*.md"):
 log_action(filepath.name, "REJECTED_NOT_EXECUTED")
 # Move to Done (with rejection prefix for audit trail)
 dest = DONE_DIR / f"REJECTED_{filepath.name}"
 shutil.move(str(filepath), str(dest))
 print(f" -> Archived to Done/REJECTED_{filepath.name}")

def main() -> None:
 """Main loop: ensure directories exist, then poll."""
 for d in [APPROVED_DIR, REJECTED_DIR, DONE_DIR, LOGS_DIR]:
 d.mkdir(parents=True, exist_ok=True)

 print(f"[HITL Watcher] Monitoring:")
 print(f" Approved: {APPROVED_DIR}")
 print(f" Rejected: {REJECTED_DIR}")
 print(f" Logs: {LOGS_DIR}")
 print(f" Archive: {DONE_DIR}")
 print(f" Polling every 5 seconds. Press Ctrl+C to stop.\n")

 while True:
 process_approved()
 process_rejected()
 time.sleep(5)

if __name__ == "__main__":
 main()
```

Why polling instead of watchdog? The approval workflow doesn't need sub-second response times. A 5-second polling loop is simpler to debug and sufficient for human-paced approvals.

## Testing the Full Approval Cycle

This is the critical test. You will run the watcher, approve a request, and verify every step.

### Step 1: Start the Watcher

Open a terminal and run:

python3 ~/projects/ai-vault/approval_watcher.py

Output:

[HITL Watcher] Monitoring:
 Approved: /Users/you/projects/ai-vault/Approved
 Rejected: /Users/you/projects/ai-vault/Rejected
 Logs: /Users/you/projects/ai-vault/Logs
 Archive: /Users/you/projects/ai-vault/Done
 Polling every 5 seconds. Press Ctrl+C to stop.

The watcher is now running. Leave this terminal open.

### Step 2: Approve the Request

Open a second terminal. Move the approval request from Pending_Approval to Approved:

mv ~/projects/ai-vault/Pending_Approval/EMAIL_client_response_2026-01-15.md \
 ~/projects/ai-vault/Approved/

### Step 3: Verify Processing

Within 5 seconds, the watcher terminal shows:

Output:

[2026-01-15 10:32:15] APPROVED_AND_EXECUTED: EMAIL_client_response_2026-01-15.md
 -> Archived to Done/EMAIL_client_response_2026-01-15.md

### Step 4: Verify the Audit Trail

Check that the file moved to /Done/:

ls ~/projects/ai-vault/Done/

Output:

EMAIL_client_response_2026-01-15.md

Check the log entry:

cat ~/projects/ai-vault/Logs/$(date +%Y-%m-%d).log

Output:

[2026-01-15 10:32:15] APPROVED_AND_EXECUTED: EMAIL_client_response_2026-01-15.md

Confirm /Approved/ is now empty:

ls ~/projects/ai-vault/Approved/

Output:

(empty - file has been processed and archived)

## Testing the Rejection Flow

Rejection testing is equally important. The system must provably NOT execute a rejected action.

### Step 1: Create a Second Approval Request

```markdown
---
type: approval_request
action: payment
target: "unknown-vendor@billing.com"
amount: 750.00
reason: "Invoice received from new vendor. Not in approved payees list."
created: "2026-01-15T11:00:00Z"
expires: "2026-01-16T11:00:00Z"
status: pending
---

## Proposed Action

Send payment of $750.00 to unknown-vendor@billing.com

## Why This Needs Approval

New payee (never paid before) AND amount exceeds $100 threshold.
Both conditions require human approval per permission boundaries.

## To Approve

Move this file to the /Approved/ folder.

## To Reject

Move this file to the /Rejected/ folder.
```

### Step 2: Reject the Request

Move it to /Rejected/:

mv ~/projects/ai-vault/Pending_Approval/PAYMENT_vendor_2026-01-15.md \
 ~/projects/ai-vault/Rejected/

### Step 3: Verify Rejection Was Logged

The watcher terminal shows:

Output:

[2026-01-15 11:01:20] REJECTED_NOT_EXECUTED: PAYMENT_vendor_2026-01-15.md
 -> Archived to Done/REJECTED_PAYMENT_vendor_2026-01-15.md

### Step 4: Verify No Action Was Taken

Check the log file -- it should show REJECTED_NOT_EXECUTED, confirming no payment was sent:

cat ~/projects/ai-vault/Logs/$(date +%Y-%m-%d).log

Output:

[2026-01-15 10:32:15] APPROVED_AND_EXECUTED: EMAIL_client_response_2026-01-15.md
[2026-01-15 11:01:20] REJECTED_NOT_EXECUTED: PAYMENT_vendor_2026-01-15.md

Check the Done folder -- rejected files are prefixed with REJECTED_ for audit clarity:

ls ~/projects/ai-vault/Done/

Output:

EMAIL_client_response_2026-01-15.md
REJECTED_PAYMENT_vendor_2026-01-15.md

The REJECTED_ prefix makes it immediately visible during audits which items were declined.

## Connecting to Your Orchestrator

You will integrate HITL with your full orchestrator pipeline in L12 (Gold Capstone). For now, your approval watcher works independently -- test it by manually creating approval requests in /Pending_Approval/, moving them to /Approved/ or /Rejected/, and verifying the watcher processes them correctly. The workflow you built in this lesson is the complete safety gate; L12 connects it to the orchestrator so your employee writes approval requests automatically when it detects a sensitive action.

## Troubleshooting

ProblemCauseFix
Watcher does not detect filesWrong directory pathVerify VAULT path matches your actual vault location with echo $HOME/projects/ai-vault
Permission denied on file moveFile permissionsRun chmod 644 on the approval request file
Log file not createdLogs/ directory missingThe watcher creates it automatically on startup, but verify with ls ~/projects/ai-vault/Logs/
Watcher crashes on startupPython path issueUse python3 explicitly, not python
Files in Pending but never processedFiles must be moved to Approved or RejectedThe watcher only monitors Approved and Rejected, not Pending

## Try With AI

Setup: Have your approval watcher running and your vault folders created.

Prompt 1: Design Your Permission Boundaries

I am building a Personal AI Employee that handles these domains:
- Email (Gmail)
- File management (local vault)
- Calendar (Google Calendar)
- Social media (LinkedIn posts)

Help me design permission boundaries for each domain. For each action
category, define:
1. What should auto-approve (routine, low-risk)
2. What should require my approval (sensitive, irreversible)
3. Where the threshold is and why

Present it as a table I can add to my AGENTS.md file.

What you're learning: Designing permission boundaries requires thinking about YOUR specific risk tolerance. The AI will suggest reasonable defaults, but you will likely want to adjust thresholds based on your domain. Notice where AI's suggestions feel too permissive or too restrictive -- that friction reveals where your context matters most.

Prompt 2: Generate Approval Request Templates

I need approval request templates for three sensitive actions
my AI employee might encounter:

1. Sending an email to someone I have never emailed before
2. Deleting a file that is older than 30 days
3. Posting a reply to a LinkedIn comment

For each, create a complete approval request file with YAML
frontmatter (type, action, target, reason, created, expires, status)
and a human-readable body explaining what the employee wants to do
and why it needs approval. Save them to my /Pending_Approval/ folder.

What you're learning: Approval request templates need enough context for you to make a quick decision. If the request is vague ("wants to send an email"), you have to investigate. If it is specific ("wants to send project proposal to [new-client@example.com](mailto:new-client@example.com) because they requested it via the contact form"), you can approve in seconds. You are learning to design for fast human review, not just any review.

Safety Note: The approval watcher in this lesson logs and archives but does not execute real actions (no actual emails sent, no payments processed). In a production system, the process_approved function would call the appropriate MCP tool. Always test with logging-only mode before connecting to real services.
