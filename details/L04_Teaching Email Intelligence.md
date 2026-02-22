-   [](/)
-   [Part 2: Agent Workflow Primitives](/docs/Agent-Workflow-Primitives)
-   [Chapter 13: Build Your AI Employee](/docs/Agent-Workflow-Primitives/build-first-ai-employee)
-   L04: Email Intelligence

Updated Feb 21, 2026

[Version history](https://github.com/panaversity/ai-native-software-development/commits/main/apps/learn-app/docs/02-Agent-Workflow-Primitives/13-build-first-ai-employee/04-teaching-email-intelligence.md)

# Teaching Email Intelligence

Your inbox shows 47 unread messages. Buried in that count is a 23-message thread about the Q4 budget that you need to respond to before 3 PM. But you can't just read the latest message—critical context is scattered across replies from five different stakeholders over two weeks. Someone made a decision about headcount. Someone assigned you an action item. Someone asked a question that's still unanswered.

You could spend 20 minutes re-reading the entire thread. Or you could invoke a skill that does it in seconds.

This lesson builds an **email-summarizer skill** that transforms long email threads into actionable intelligence: decisions made, tasks assigned, questions pending. More importantly, you'll learn how skills chain together—how the summarizer's output becomes the drafter's input, creating an automated read-to-respond workflow.

* * *

## The Information Extraction Challenge

Long email threads bury critical information in conversational noise. Consider a typical 15-message thread:

Message Type

Percentage

Value

Greetings and sign-offs

25%

Zero

Context repetition

20%

Low (already known)

Discussion and exploration

30%

Medium (background)

Decisions and action items

15%

High (actionable)

Open questions

10%

Critical (blocks progress)

Only 25% of thread content matters for your response. The email-summarizer skill extracts that 25% while discarding the rest.

**Why this matters beyond time savings:**

-   **Accuracy**: Manual scanning misses items. Systematic extraction catches everything.
-   **Consistency**: Same extraction pattern every time, not dependent on your attention span.
-   **Handoff**: Clear summary enables delegation. Anyone can pick up the thread.
-   **Memory**: Summaries become searchable records of decisions made.

* * *

## Understanding Thread Structure

Before extracting information, you need to understand how email threads are structured. Different email clients format threads differently, but common patterns exist.

**Thread Markers to Recognize:**

```
On Mon, Jan 15, 2024 at 2:30 PM Sarah Chen <sarah@company.com> wrote:
```

```
From: Marcus Thompson <marcus@company.com>Sent: Tuesday, January 16, 2024 9:15 AMTo: Team <team@company.com>Subject: Re: Q4 Budget Discussion
```

```
---------- Forwarded message ---------From: Finance Team <finance@company.com>Date: Wed, Jan 17, 2024 at 11:00 AM
```

**What to Extract from Structure:**

Element

What It Tells You

Sender name

Who said what

Timestamp

When (chronological reconstruction)

Subject changes

Topic pivots or forwards

Reply depth

Conversation branches

CC additions

Stakeholder expansion

**Message Boundary Detection:**

Threads nest replies inside replies. The skill needs to identify where one message ends and another begins. Key signals:

-   Timestamp patterns (`On [date], [name] wrote:`)
-   Quote markers (`>` at line start)
-   Horizontal rules or separator lines
-   Header blocks (From/To/Subject/Date)

* * *

## Extraction Targets: What to Pull from Threads

Your skill needs to extract three categories of information, each requiring different recognition patterns.

### 1\. Decisions Made

Explicit agreements or conclusions that close discussion:

**Signal Patterns:**

-   "We've decided to..."
-   "Let's go with..."
-   "Agreed—we'll..."
-   "Final decision:"
-   "Approved by..."
-   "Confirmed:"
-   "Sign-off received"

**What to Capture:**

-   The decision itself
-   Who made or approved it
-   When it was made
-   Any conditions attached

**Example Extraction:**

```
Thread excerpt:"After reviewing the proposals, let's go with Vendor B for thecloud migration. Sarah approved the budget yesterday."Extracted:DECISION: Select Vendor B for cloud migrationWHO: Team decision, budget approved by SarahWHEN: [date of message]CONDITIONS: None stated
```

### 2\. Action Items

Tasks assigned to specific people with ownership:

**Signal Patterns:**

-   "Can you please..."
-   "\[Name\] will..."
-   "I'll take care of..."
-   "Action item:"
-   "TODO:"
-   "Next step:"
-   "By \[date\], we need..."
-   "Your action:"

**What to Capture:**

-   The task description
-   Who is responsible (owner)
-   Due date (if stated)
-   Dependencies (if mentioned)

**Example Extraction:**

```
Thread excerpt:"Marcus, can you pull the usage metrics from Q3?We need them before Friday's meeting."Extracted:ACTION: Pull Q3 usage metricsOWNER: MarcusDUE: Before Friday meetingDEPENDENCY: Needed for Friday discussion
```

### 3\. Open Questions

Unresolved items waiting for response:

**Signal Patterns:**

-   Lines ending with `?`
-   "What do you think about..."
-   "Need input on..."
-   "Waiting for..."
-   "TBD:"
-   "Open question:"
-   "Has anyone..."
-   "Can someone clarify..."

**What to Capture:**

-   The question itself
-   Who asked it
-   Who should answer (if specified)
-   How long it's been open

**Example Extraction:**

```
Thread excerpt (from 3 days ago):"Does the new pricing apply to existing customers,or just new signups? Need to know before we update the FAQ."Extracted:QUESTION: New pricing scope (existing vs new customers)ASKED BY: [sender]ANSWER NEEDED FROM: [not specified - likely product/sales]OPEN FOR: 3 daysBLOCKS: FAQ update
```

* * *

## The Email Summarizer Skill Structure

Your skill follows the same pattern as email-drafter and email-templates:

```
.claude/skills/└── email-summarizer/    ├── SKILL.md                    # Main skill instructions    └── references/        └── extraction-patterns.md  # Pattern recognition library
```

* * *

## Building the SKILL.md

Here's the complete SKILL.md for your email-summarizer skill. Create this at `.claude/skills/email-summarizer/SKILL.md`:

```
---name: email-summarizerdescription: This skill summarizes email threads and extracts key information. Use when the user has a long email thread to understand, needs to identify action items, or wants context before replying. Extracts decisions made, action items, and open questions.---# Email Summarizer## OverviewTransform long email threads into actionable summaries with extracted decisions, action items, and context.## When to Use This Skill- Email thread has 5+ messages- Need to quickly understand thread state- Preparing to reply to a thread- Extracting action items from discussions- Creating meeting notes from email exchanges- Briefing someone on thread status## Extraction Targets### 1. Decisions MadeExplicit agreements or conclusions:- Who decided what- When it was decided- Any conditions attached### 2. Action ItemsTasks assigned to specific people:- WHO is responsible- WHAT they need to do- WHEN it's due (if stated)### 3. Open QuestionsUnresolved items needing response:- Questions asked but not answered- Items waiting for input- Blockers mentioned### 4. Key ContextBackground for response:- Current state of discussion- Positions of key stakeholders- Recent developments## Output Formats### Executive Summary (Default)Quick overview for busy professionals:- 3-5 sentence summary- Top 3 action items- Immediate next step### Detailed BreakdownFor complex threads:- Decision log with timestamps- Full action item list with owners- Open questions with context- Stakeholder positions### Response ContextWhen preparing to reply:- What the thread is asking of you- Your pending action items- Key points to address## Workflow1. Parse thread structure (identify messages and senders)2. Identify participants and their roles3. Extract decisions chronologically4. Extract action items with ownership5. Identify open questions6. Format based on user need7. Offer to chain with /email-drafter for response## Integration with Other Skills- Chains with `/email-drafter` for response generation- Uses tone from `email-drafter/references/tone-guidelines.md`- Can feed `/email-templates` for structured replies
```

**Key design elements:**

-   **Clear activation triggers**: "5+ messages", "action items", "preparing to reply"
-   **Three extraction targets**: Decisions, actions, questions (DAQ pattern)
-   **Multiple output formats**: Different needs require different summaries
-   **Skill chaining**: Explicit integration with drafter and templates

* * *

## Creating the Extraction Patterns Reference

The extraction patterns file gives Claude a recognition library. Create this at `.claude/skills/email-summarizer/references/extraction-patterns.md`:

```
# Extraction Patterns for Email Analysis## Decision SignalsLook for these patterns indicating a decision:- "We've decided to..."- "Let's go with..."- "Agreed - we'll..."- "Final decision:"- "Confirmed:"- "Approved by..."- "Sign-off received"- "Moving forward with..."- "After discussion, we'll..."- "Consensus is..."## Action Item SignalsPatterns indicating tasks:- "Can you please..."- "I'll take care of..."- "[Name] will..."- "Action item:"- "TODO:"- "Next step:"- "By [date], we need..."- "Your action:"- "Please send..."- "Make sure to..."- "Don't forget to..."## Question PatternsUnresolved items:- Lines ending with "?"- "What do you think about..."- "Need input on..."- "Waiting for..."- "TBD:"- "Open question:"- "Has anyone..."- "Can someone clarify..."- "Does anyone know..."- "Should we..."## Thread Structure MarkersIdentify message boundaries:- "On [date], [name] wrote:"- "From: / To: / Subject:"- "---------- Forwarded message"- "Begin forwarded message:"- Timestamp patterns (various formats)- Quote markers (> at line start)- Horizontal separator lines## Participant RolesIdentify key players:- **Original sender** (initiator) - started the thread- **Decision makers** (approvers) - can authorize- **Subject matter experts** (info providers) - provide data- **Blockers** (waiting on input) - thread stalls on them- **FYI recipients** (CC'd) - informed but not active## Urgency IndicatorsSignals requiring immediate attention:- "URGENT:" or "ASAP"- "EOD" or "End of day"- "Before [meeting/deadline]"- "Blocking [something]"- Exclamation marks (multiple)- ALL CAPS sections- "Critical" or "Priority"## De-Prioritization SignalsSafe to defer:- "When you get a chance..."- "No rush, but..."- "Low priority"- "FYI only"- "For your awareness"
```

**Why pattern libraries work:**

-   **Consistency**: Same patterns recognized every time
-   **Coverage**: Captures variations you might miss
-   **Improvement**: Add patterns as you encounter new ones
-   **Transfer**: Patterns work across domains, not just email

* * *

## Output Format Examples

Different situations require different summary formats. Here's how each works:

### Executive Summary Format

For quick decisions about whether to engage deeply:

```
## Thread Summary: Q4 Budget Discussion**Status**: Active discussion, awaiting final approval**Key Points** (3):1. Vendor B selected for cloud migration (approved by Sarah, Jan 15)2. Headcount frozen at current levels for Q43. Marketing budget reduced 15% per CFO guidance**Your Action Items**:- Pull Q3 usage metrics before Friday- Review revised timeline from Marcus**Immediate Next Step**: Respond to open question about contractor budget
```

### Detailed Breakdown Format

For complex threads requiring full context:

```
## Thread Analysis: Q4 Budget Discussion**Messages**: 23 | **Participants**: 5 | **Duration**: 12 days### Decisions Log| Date | Decision | Decided By | Conditions ||------|----------|------------|------------|| Jan 15 | Vendor B for migration | Sarah (budget), Team (technical) | Subject to contract review || Jan 18 | Headcount freeze | CFO directive | Through Q4 only || Jan 22 | Marketing -15% | CFO + CMO | Reallocate to digital |### Action Items| Task | Owner | Due | Status ||------|-------|-----|--------|| Q3 metrics | Marcus | Friday | Pending || Timeline revision | Marcus | Next week | In progress || Contract review | Legal | Jan 30 | Not started || FAQ update | You | After pricing clarity | Blocked |### Open Questions1. **Contractor budget scope** (Jan 20, asked by PM)   - Does freeze apply to existing contractors?   - No response yet (3 days open)   - Blocks: Resource planning2. **New pricing for existing customers** (Jan 19, asked by Sales)   - Grandfather existing or apply to all?   - Blocks: FAQ update, customer comms### Stakeholder Positions- **CFO**: Focused on cost reduction, firm on headcount- **CMO**: Accepted marketing cut, pushing for digital reallocation- **PM**: Concerned about timeline with frozen resources- **Sales**: Waiting on pricing clarity for customer communication
```

### Response Context Format

When you're about to reply:

```
## Response Context: Q4 Budget Thread**What the thread is asking of you**:1. Pull Q3 usage metrics (explicit request from Marcus)2. Clarify contractor budget question (you're likely the right person)3. Confirm timeline feasibility with reduced resources**Your pending action items**:- Q3 metrics: Due Friday (2 days)- FAQ update: Blocked on pricing question**Key points to address in your reply**:- Confirm you'll deliver metrics by Friday- Provide contractor budget guidance (or defer to CFO)- Flag timeline risk if resources stay frozen**Tone note**: CFO is in thread—be concise, data-focused
```

* * *

## Skill Chaining: Summarizer to Drafter

The real power emerges when skills work together. Your email-summarizer produces context that your email-drafter consumes:

**The Chain:**

```
[Long Thread] → /email-summarizer → [Context] → /email-drafter → [Reply Draft]
```

**How It Works:**

1.  Invoke summarizer on the thread
2.  Summarizer extracts action items, decisions, questions
3.  Ask to draft a response
4.  Drafter uses summary as context
5.  Draft addresses extracted items with your tone

**Example Flow:**

```
User: Summarize this thread and then help me respond.[Claude invokes /email-summarizer]Summary:- Decision: Vendor B selected- Action for you: Provide Q3 metrics by Friday- Open question: Contractor budget (needs your input)[Claude recognizes response intent, invokes /email-drafter]Using your summary, here's a draft response:Hi team,Quick updates on the open items:**Q3 Metrics**: I'll have these ready by Thursday EOD,giving us a day buffer before Friday's meeting.**Contractor Budget**: For existing contractors, I recommendwe honor current agreements through Q4. New contractorrequests should route through the approval exception process.Let me know if you need anything else before Friday.Best,[Your name]
```

**Why chaining matters:**

-   **Context preservation**: Summary context flows into draft
-   **Efficiency**: Single workflow, not separate prompts
-   **Consistency**: Same tone guidelines apply
-   **Completeness**: Draft addresses all extracted items

* * *

## Testing Your Skill

Let's verify the skill works correctly.

**Step 1: Create the directory structure**

```
mkdir -p .claude/skills/email-summarizer/references
```

**Output:**

```
(no output - directories created silently)
```

**Step 2: Verify your structure**

```
find .claude/skills -type d | sort
```

**Output:**

```
.claude/skills.claude/skills/email-drafter.claude/skills/email-drafter/references.claude/skills/email-summarizer.claude/skills/email-summarizer/references.claude/skills/email-templates.claude/skills/email-templates/templates
```

**Step 3: Create the files**

Copy the SKILL.md content to `.claude/skills/email-summarizer/SKILL.md` and the extraction patterns to `.claude/skills/email-summarizer/references/extraction-patterns.md`.

**Step 4: Test with a sample thread**

Provide Claude with a multi-message email thread and invoke the skill:

```
/email-summarizer[Paste your email thread here]Give me an executive summary and list my action items.
```

**Expected Behavior:**

1.  Claude recognizes thread parsing is needed
2.  Identifies message boundaries and senders
3.  Extracts decisions, actions, questions
4.  Formats as executive summary
5.  Highlights your specific action items

* * *

## Common Extraction Mistakes

When building extraction logic, these patterns cause problems:

**Mistake 1: Missing implied decisions**

```
Thread: "I don't hear any objections, so we'll proceed with Plan A."WRONG: No decision extracted (no explicit "decided" word)RIGHT: Decision extracted (consensus by silence)
```

**Mistake 2: Confusing suggestions with action items**

```
Thread: "We could also consider reaching out to Partner X."WRONG: Action item - reach out to Partner XRIGHT: Suggestion only (no ownership, no commitment)
```

**Mistake 3: Missing conditional action items**

```
Thread: "If the budget clears, Sarah will start procurement."WRONG: Action item - Sarah starts procurementRIGHT: Conditional action - Sarah starts IF budget clears
```

**Mistake 4: Ignoring thread chronology**

```
Early message: "Let's use Vendor A"Later message: "After review, switching to Vendor B"WRONG: Decision - use Vendor ARIGHT: Decision - use Vendor B (supersedes earlier)
```

* * *

## Extending the Skill

Once the basic skill works, consider these enhancements:

Extension

Value

Complexity

Sentiment detection

Identify tension or urgency

Medium

Stakeholder mapping

Auto-identify roles

Low

Timeline extraction

Build project timeline from thread

Medium

Commitment tracking

Match promises to follow-through

High

**Sentiment example addition to SKILL.md:**

```
## Sentiment Indicators### Urgency Signals- Multiple exclamation marks- "URGENT", "ASAP", "Critical"- Short response times (< 1 hour between messages)### Tension Signals- Defensive language ("As I mentioned...")- Escalation (new executives added to CC)- Formal tone shift (first names → full names)
```

* * *

## Try With AI

**Setup:** Open Claude Code in the `email-assistant` directory with the email-summarizer skill installed.

**Prompt 1: Basic Thread Extraction**

```
/email-summarizerHere's a thread I need to understand quickly:[Paste a real or sample email thread with 5+ messagesthat contains at least one decision, one action item,and one open question]Give me the executive summary format.
```

**What you're learning:** This tests the core extraction workflow. Review the output: Did Claude correctly identify message boundaries? Are the extraction targets (decisions, actions, questions) accurate? Did anything get missed or miscategorized?

**Prompt 2: Response Context Generation**

```
Now give me the response context format for that same thread.I need to understand what the thread is asking of ME specificallyand what I should address in my reply.
```

**What you're learning:** This tests output format flexibility. The same thread should produce different summaries for different purposes. Compare the executive summary to the response context—same source data, different focus.

**Prompt 3: Chain to Email Drafter**

```
Based on that summary, help me draft a response.Use /email-drafter to compose a reply that addressesall the action items assigned to me and answersthe open questions I can answer.
```

**What you're learning:** This demonstrates skill chaining. The summarizer's output becomes the drafter's context. Notice how the draft addresses the specific items extracted from the summary. This is workflow automation—multiple skills coordinating automatically.

**Safety Note:** When summarizing threads containing sensitive information, be mindful of what you share with AI systems. Summarization is powerful but should be used appropriately for the content's sensitivity level.

# Lesson 3 Summary: Email Summarizer Skill

## Key Concepts

1.  **Thread Parsing**: Identify message boundaries, sender attribution, and chronological order in email threads
    
2.  **Extraction Targets**: Three categories to extract:
    
    -   **Decisions**: Conclusions reached in the thread
    -   **Action Items**: Tasks assigned with owners and deadlines
    -   **Open Questions**: Unanswered queries requiring response
3.  **Output Formatting**: Different formats for different use cases:
    
    -   Executive Summary: Brief overview for quick decisions
    -   Detailed Breakdown: Complete analysis for response planning
4.  **Skill Chaining**: Connect summarizer output to drafter input for automated response generation
    
5.  **Reference Patterns**: Create extraction patterns reference file for consistent analysis
    

## Deliverables

-   Working `/email-summarizer` skill with SKILL.md
-   `references/extraction-patterns.md` with extraction criteria
-   Skill chaining demonstration with email-drafter

## Key Code Snippets

### Extraction Patterns Reference

```markdown
# Extraction Patterns

## Decision Indicators
- "We've decided to..."
- "The final approach is..."
- "Going with option X"

## Action Item Markers
- "Please [verb]..."
- "[Name] will..."
- "By [date]..."

## Open Question Signals
- "What about...?"
- "Has anyone...?"
- "Still need to determine..."

```

### Output Format: Executive Summary

```markdown
## Thread Summary: {{subject}}

**Key Decision**: {{main_decision}}

**Your Action Items**:
1. {{action_1}} (due: {{deadline_1}})
2. {{action_2}} (due: {{deadline_2}})

**Open Questions**:
- {{question_1}}

```

### Skill Chaining Pattern

```markdown
## Workflow: Summarize → Draft Response

1. Invoke /email-summarizer on thread
2. Extract action items assigned to user
3. Identify open questions requiring response
4. Pass context to /email-drafter
5. Generate response addressing all items

```

## Try With AI Prompts

1.  `/email-summarizer` → Paste a long email thread for analysis
2.  Test extraction accuracy with threads containing implicit deadlines
3.  Chain summarizer output to drafter for response generation

## Skills Practiced

| Skill | Proficiency | Assessment | |-------|-------------|------------| | Thread Parsing | A2 | Identify messages and senders | | Information Extraction | A2 | Extract decisions, actions, questions | | Output Format Design | A2 | Create appropriate formats | | Skill Chaining | A2 | Connect skill outputs to inputs | | Skill Reference Files | A2 | Create extraction patterns |

## Duration

25 minutes

## Next Lesson

[Lesson 4: Creating Custom Subagents](./04-creating-custom-subagents.md) - Build specialized email processing agents

---
Source: https://agentfactory.panaversity.org/docs/Agent-Workflow-Primitives/build-first-ai-employee/teaching-email-intelligence