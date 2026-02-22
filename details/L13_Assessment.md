# L13: Assessment - Chapter 13 Quiz

Test your understanding of the concepts from Lessons 1 through 12. This assessment covers vault setup, skills, subagents, MCP, watchers, HITL, scheduling, error recovery, and audit logging across all three tiers.

## Bronze Tier Assessment (L01-L07)

### Question 1: Vault Structure
What are the three core governance files that define your AI employee's behavior, and what is the purpose of each?

<details>
<summary>Click to reveal answer</summary>

- **AGENTS.md**: Governance rules, permission boundaries, and behavioral constraints
- **CLAUDE.md**: Context about your business, projects, and working style
- **Company_Handbook.md**: Standard operating procedures and domain-specific rules

</details>

### Question 2: Skills vs Subagents
What is the fundamental difference between a Skill and a Subagent? When would you use each?

<details>
<summary>Click to reveal answer</summary>

**Skills** provide expertise and templates (content generation with known patterns). Use for: email drafting, template application, summarization.

**Subagents** provide autonomous reasoning and classification (judgment calls). Use for: priority triage, response suggestions, deadline tracking.

Key difference: Skills = guidance, Subagents = reasoning.

</details>

### Question 3: MCP Purpose
What does MCP (Model Context Protocol) provide that skills alone cannot?

<details>
<summary>Click to reveal answer</summary>

MCP provides **external system integration** -- real operations on real systems. Skills work with data you provide; MCP can fetch from Gmail, write to databases, control browsers, etc. MCP transforms your employee from a consultant (advises) to a worker (acts).

</details>

### Question 4: Graceful Degradation
Your Gmail MCP connection fails. According to L07, how should the email-assistant skill respond?

<details>
<summary>Click to reveal answer</summary>

1. Continue working with local skills (email-drafter, email-templates still function)
2. Accept pasted email content for subagent analysis
3. Output email content to clipboard instead of creating Gmail drafts
4. Clearly notify user: "Gmail MCP offline. Email copied to clipboard for manual sending."

</details>

## Silver Tier Assessment (L08-L11)

### Question 5: Watcher Pattern
What are the three methods every watcher follows, and what is the key insight about what watchers do NOT do?

<details>
<summary>Click to reveal answer</summary>

**Three methods:**
1. `check_for_updates()` - Poll or listen for changes
2. `create_action_file(item)` - Convert detected item to markdown
3. `run()` - Infinite loop: check, process, sleep

**Key insight:** Watchers do NOT process, analyze, or decide. They only detect and deposit. Claude Code handles all reasoning.

</details>

### Question 6: HITL Workflow
Describe the four-folder approval workflow. What happens when a file is moved to each folder?

<details>
<summary>Click to reveal answer</summary>

| Folder | Purpose | Result |
|--------|---------|--------|
| `/Pending_Approval/` | Employee writes requests here | Awaiting human decision |
| `/Approved/` | Human moves to approve | Watcher logs + executes + archives to Done |
| `/Rejected/` | Human moves to reject | Watcher logs (no action) + archives with REJECTED_ prefix |
| `/Done/` | Final archive | Audit trail complete |

</details>

### Question 7: PM2 vs Cron
What is the difference between PM2 and cron? Give an example use case for each.

<details>
<summary>Click to reveal answer</summary>

**PM2:** Keeps long-running processes alive forever. Restarts crashes. Survives reboots.
- Example: File watcher monitoring a drop folder 24/7

**Cron:** Runs tasks at specific times, then exits.
- Example: CEO Briefing every Sunday at 11 PM

PM2 = continuous. Cron = scheduled.

</details>

### Question 8: Stop Hook Exit Codes
What exit code tells Claude Code to continue processing? What exit code tells it to stop?

<details>
<summary>Click to reveal answer</summary>

- **Exit 2** = Keep going (tasks remain)
- **Exit 0** = Stop (all tasks complete)

Exit 1 is a non-blocking error (Claude ignores it and stops anyway).

</details>

### Question 9: CEO Briefing Data Sources
What three data sources does the CEO Briefing skill read, and what insight does each provide?

<details>
<summary>Click to reveal answer</summary>

| Source | Provides |
|--------|----------|
| `Business_Goals.md` | Revenue targets, KPIs, subscription audit rules |
| `/Done/*.md` | Completed tasks with duration and revenue |
| `/Logs/*.json` | Activity history for the reporting period |

Cross-referencing these three sources produces insights like "Client B proposal took 3x expected duration" or "Notion subscription inactive for 47 days."

</details>

## Gold Tier Assessment (L12)

### Question 10: Error Categories
Match each error type to its recovery strategy:

| Error | Category | Recovery |
|-------|----------|----------|
| Network timeout | ? | ? |
| OAuth token expired | ? | ? |
| Corrupted markdown file | ? | ? |
| Process crash | ? | ? |

<details>
<summary>Click to reveal answer</summary>

| Error | Category | Recovery |
|-------|----------|----------|
| Network timeout | Transient | Exponential backoff retry (up to 3 attempts) |
| OAuth token expired | Authentication | Pause operations, write alert to vault, human re-authenticates |
| Corrupted markdown file | Data | Quarantine to /Errors/, alert human |
| Process crash | System | PM2 auto-restart, watchdog alert |

</details>

### Question 11: Audit Log Required Fields
List the six required fields for every audit log entry.

<details>
<summary>Click to reveal answer</summary>

1. `timestamp` - When (UTC ISO 8601)
2. `action_type` - What category (email_send, file_move, triage)
3. `actor` - Who (claude_code, gmail_watcher, human)
4. `target` - What was acted upon (email, file path)
5. `approval_status` - HITL status (auto_approved, approved, rejected, not_required)
6. `result` - Outcome (success, failed, retry_scheduled)

</details>

### Question 12: End-to-End Verification
When testing the full invoice flow, what five verification checks confirm the pipeline works?

<details>
<summary>Click to reveal answer</summary>

1. Original file moved to `/Done/`
2. Approval file cleaned up from `/Approved/`
3. Log entry written to `/Logs/`
4. Dashboard.md updated with new activity row
5. Git commit recorded with descriptive message

All five must pass for Gold Tier certification.

</details>

## Practical Exercises

### Exercise 1: Design Permission Boundaries

Design permission boundaries for a new domain: **Calendar Management**. Create a table with:
- 3 auto-approve actions (low risk)
- 3 require-approval actions (sensitive)
- Clear thresholds for each

### Exercise 2: Build a Watcher

Write a Python script that monitors a specific folder for `.pdf` files only. When detected, create an action file that includes:
- File name and size
- Detection timestamp
- Suggested action based on filename patterns (e.g., "invoice" → process payment)

### Exercise 3: Debug a Stop Hook

Given this broken hook:

```bash
#!/bin/bash
TASK_DIR="$HOME/projects/ai-vault/Needs_Action"
if ls "$TASK_DIR"/*.md 1>/dev/null 2>&1; then
  echo "Tasks remaining"
  exit 1
fi
exit 0
```

Identify the bug and fix it. Explain why the original doesn't work.

### Exercise 4: Write Audit Queries

Write shell commands to answer these questions from your logs:
1. How many approval requests were rejected this week?
2. What is the most common error type?
3. Which action type has the highest failure rate?

## Submission Guidelines

To complete Chapter 13:

1. **Complete all 12 quiz questions** - Write your answers without looking, then check
2. **Finish all 4 practical exercises** - Implement and test your solutions
3. **Build your Gold Tier employee** - Verify all 10 checklist items pass
4. **Document your system** - README.md that explains your implementation

## Try With AI

### Prompt 1: Generate Your Own Review Questions

I just completed Chapter 13 (Build Your First AI Employee).
Read my vault's AGENTS.md and CLAUDE.md, then generate 5 new
quiz questions about MY specific implementation — not generic
questions, but ones that test whether I truly understand how
my own employee is configured.

What you're practicing: Turning AI into a personalized assessor. Generic quizzes test textbook knowledge; AI-generated questions based on your actual vault test applied understanding.

### Prompt 2: Identify Your Weakest Concept

Here are the Chapter 13 topics I got wrong on the assessment:
[paste your wrong answers]. For each one, explain the concept
in a different way than the textbook did, and give me a
concrete scenario where getting it wrong would cause a real
problem in my AI employee.

What you're practicing: Using AI as a targeted tutor. Instead of re-reading entire lessons, you direct AI to the exact gaps in your understanding and ask for alternative explanations with real-world consequences.

---

**Next:** After completing this assessment, proceed to **L14: Hackathon 0** where you will build a complete Personal AI Employee project from scratch and submit it as a GitHub repository.
