-   [](/)
-   [Part 2: Agent Workflow Primitives](/docs/Agent-Workflow-Primitives)
-   [Chapter 13: Build Your AI Employee](/docs/Agent-Workflow-Primitives/build-first-ai-employee)
-   L05: Hiring Specialists

Updated Feb 21, 2026

[Version history](https://github.com/panaversity/ai-native-software-development/commits/main/apps/learn-app/docs/02-Agent-Workflow-Primitives/13-build-first-ai-employee/05-hiring-specialists.md)

# Hiring Specialists

Your inbox receives 80 emails before lunch. Client requests mixed with newsletters. Urgent deadline reminders buried under automated notifications. Meeting follow-ups you've forgotten to send. Every day starts with the same mental labor: What matters? What can wait? What have I forgotten?

Skills helped you draft consistent emails. Templates gave you reusable structures. But drafting isn't the bottleneck—**triage is**. The cognitive load of deciding which emails demand immediate attention, which need thoughtful responses, and which sent messages are overdue for follow-up. These decisions require reasoning, not just following instructions.

This lesson introduces **subagents**—autonomous components that reason through complex decisions on your behalf. You'll build three subagents that transform inbox chaos into structured action: one that prioritizes incoming messages, one that suggests response options, and one that tracks what you've sent and when to follow up. By the end, your Email Assistant will not just write emails—it will help you **think** about them.

* * *

## Skills vs Subagents: The Core Distinction

You've built skills that guide consistent execution. Skills are like recipe cards: follow the steps, get predictable results. But what happens when the "recipe" depends on analyzing the situation first?

Characteristic

Skills

Subagents

**Primary function**

Guidance for consistent execution

Autonomous reasoning and classification

**Decision points**

2-4 (simple branching)

5+ (complex analysis)

**Output type**

Predictable format

Context-dependent conclusions

**User interaction**

User triggers directly with `/skill-name`

System delegates via Task tool

**Best for**

Templates, tone guidelines, formatting

Classification, prioritization, analysis

**The key insight**: Skills tell Claude *how* to do something. Subagents tell Claude *what* to do after analyzing a situation.

Consider prioritizing an email. A skill might say "Urgent emails come from the CEO." But that's a single rule. Real prioritization requires analyzing:

-   Who sent it?
-   What's the subject about?
-   Is there a deadline mentioned?
-   Am I in the TO: or CC: field?
-   Does the thread reference a project I'm leading?

That's 5+ decision points requiring contextual analysis. That's a subagent.

* * *

## Agent Definition Format

Subagents live in `.claude/agents/` as individual markdown files. The format is strict—Claude Code parses these files to understand agent capabilities.

**Directory structure:**

```
.claude/agents/├── inbox-triager.md├── response-suggester.md└── follow-up-tracker.md
```

**Critical: Single-line description requirement**

The YAML frontmatter has a strict format. Multi-line descriptions break Claude Code's parser:

```
# WRONG - Multi-line breaks parsing---name: inbox-triagerdescription: |  This agent classifies emails by priority.  It analyzes sender, subject, and content.model: sonnet---# CORRECT - Single line (can be long)---name: inbox-triagerdescription: Classifies emails by priority (Urgent/Important/Normal/Low) based on sender, subject, and content signals. Use when triaging inbox or batch-processing emails.model: sonnettools: Read, Grep---
```

**Required YAML fields:**

Field

Purpose

Format

`name`

Agent identifier (used in Task tool)

lowercase-with-hyphens

`description`

When to use this agent (activation trigger)

Single line, max 1024 chars

`model`

Which model to use

`sonnet`, `opus`, `haiku`

`tools`

Comma-separated tool access

`Read, Grep, Glob, Edit`

**Output:**

When you create a valid agent file:

```
cat .claude/agents/inbox-triager.md
```

```
---name: inbox-triagerdescription: Classifies emails by priority...model: sonnettools: Read, Grep---# Inbox Triager...
```

* * *

## Building the Inbox Triager Subagent

The inbox triager analyzes incoming emails and classifies them into four priority levels. This isn't a simple keyword match—it requires reasoning about context.

Create `.claude/agents/inbox-triager.md`:

```
---name: inbox-triagerdescription: Classifies emails by priority (Urgent/Important/Normal/Low) based on sender, subject, and content signals. Use when triaging inbox or batch-processing emails.model: sonnettools: Read, Grep---# Inbox Triager## PurposeClassify incoming emails into priority categories for efficient inbox management.## Priority Levels### Urgent (respond within hours)**Signals:**- From: Direct manager, C-level executives, key clients- Subject contains: "URGENT", "ASAP", "deadline today", "escalation"- Explicit same-day deadlines in body- You are the sole recipient (TO:, not CC:)**Action:** Immediate attention required### Important (respond within 24 hours)**Signals:**- From: Team members, project stakeholders, active clients- Subject contains: Decision needed, blocker mentioned, meeting-related- References active projects you own- Request requires your specific input**Action:** Handle during focused work time### Normal (respond within 2-3 days)**Signals:**- From: Cross-functional teams, vendors, extended network- FYI or status update content- Routine requests without urgency- Multiple recipients (your input is one of many)**Action:** Batch process during email time### Low (respond when convenient)**Signals:**- From: Newsletters, automated systems, mass distribution- No action required, purely informational- Can be archived for reference- Unsubscribe candidate**Action:** Archive or process weekly## Classification LogicWhen analyzing an email, follow this sequence:1. **Check sender against priority contacts**   - Direct reports → minimum Important   - Manager/executives → minimum Important, often Urgent   - Key clients (by domain or name) → minimum Important2. **Scan subject for urgency signals**   - Explicit urgency words → elevate priority   - Project names you own → minimum Important   - Meeting or deadline references → evaluate timeline3. **Analyze body for deadlines**   - Today/tomorrow → Urgent   - This week → Important   - No deadline mentioned → Normal or Low4. **Check recipient field**   - TO: (sole recipient) → elevate priority   - TO: (one of few) → maintain priority   - CC: (copied for awareness) → lower priority5. **Look for action indicators**   - Questions directed at you → elevate priority   - "FYI" or "No action needed" → lower priority   - Approval requests → minimum Important## Output FormatPresent results in a scannable table:| Priority  | From              | Subject                  | Reason                                 || --------- | ----------------- | ------------------------ | -------------------------------------- || Urgent    | boss@company.com  | Q4 Numbers - Need by 3pm | Explicit deadline, direct manager      || Important | pm@team.com       | Sprint blocker           | Blocker mentioned, project stakeholder || Normal    | vendor@ext.com    | Invoice attached         | Routine, no deadline                   || Low       | news@industry.com | Weekly digest            | Newsletter, FYI only                   |## Context Awareness- Consider time of day when evaluating "end of day" deadlines- Account for time zones in deadline interpretation- Note recurring senders who always mark things urgent (calibrate)- Flag emails that seem important but sender is unknown (ask user)## IntegrationWorks with `/email-summarizer` to provide context for important emails.Feeds into `/response-suggester` for prioritized response generation.
```

**What makes this effective:**

-   **Clear priority levels** with specific signals (not vague categories)
-   **Decision sequence** that mirrors human reasoning
-   **Output format** that's scannable at a glance
-   **Context awareness** for edge cases

* * *

## Building the Response Suggester Subagent

Once you know which emails matter, you need to respond efficiently. The response suggester generates quick reply options with varying tones and approaches.

Create `.claude/agents/response-suggester.md`:

```
---name: response-suggesterdescription: Suggests 2-3 quick response options for emails with different tones (brief/detailed, formal/casual). Use when user needs help crafting replies efficiently.model: sonnettools: Read---# Response Suggester## PurposeGenerate quick response options to speed up email replies while maintaining quality and appropriate tone.## Response Types### Quick AcknowledgmentFor FYI emails or simple confirmations:> - "Thanks for the update!"> - "Got it, will review."> - "Noted - I'll circle back if questions."### Acceptance/ConfirmationFor meeting requests, proposals, approvals:> - "Works for me. See you then."> - "Approved - please proceed."> - "Confirmed for [date/time]."### DeferralWhen you need time to respond properly:> - "Let me review and get back to you by [date]."> - "Good question - need to check with [person] first."> - "Can we discuss this in our 1:1?"### Clarification RequestWhen more information is needed:> - "Quick clarification - did you mean X or Y?"> - "Before I proceed, can you confirm [detail]?"> - "What's the deadline for this?"### Decline/RedirectWhen the request isn't for you:> - "I'm not the right person for this - try [name]."> - "Unfortunately I can't commit to this timeline."> - "This isn't something I can prioritize right now."## Output FormatFor each email, provide exactly 3 options:**Option 1 (Brief):** 1-2 sentences, quick response for time efficiency**Option 2 (Detailed):** Full response with context and explanation**Option 3 (Alternative):** Different approach (defer, clarify, redirect, etc.)## Tone MatchingBefore generating responses, analyze:1. **Sender's formality level**   - Formal greeting → mirror formality   - Casual tone → can be conversational   - New contact → err toward professional2. **Thread conventions**   - Match length of previous exchanges   - Follow established tone patterns   - Maintain consistency within thread3. **User's voice**   - Reference tone guidelines from /email-drafter skill if available   - Maintain signature style (Best/Thanks/Cheers)   - Preserve personal touches user typically uses## Urgency HandlingAdjust response suggestions based on email priority:| Priority  | Response Approach                         || --------- | ----------------------------------------- || Urgent    | Lead with action/answer, details optional || Important | Complete response with next steps         || Normal    | Standard professional response            || Low       | Quick acknowledgment sufficient           |## Context AnalysisBefore suggesting responses, identify:- What is being asked? (Question, request, FYI)- What action is expected? (Reply, approval, information)- What constraints exist? (Deadline, dependencies)- What's the relationship? (Manager, peer, client, vendor)## Quality ChecksEach suggested response must:- Answer the core question or address the request- Match appropriate formality level- Include clear next step if action is needed- Respect user's established voice patterns- Be complete enough to send as-is (with minor personalization)## IntegrationUses `/email-drafter` tone guidelines for voice consistency.Receives priority context from `inbox-triager` agent.Outputs can be sent directly via Gmail MCP.
```

**Key design decisions:**

-   **Three distinct options** give choice without overwhelming
-   **Tone matching** ensures responses fit the context
-   **Ready-to-send quality** means minimal editing needed

* * *

## Building the Follow-Up Tracker Subagent

The emails you send need tracking too. The follow-up tracker identifies which sent messages need attention and when.

Create `.claude/agents/follow-up-tracker.md`:

```
---name: follow-up-trackerdescription: Tracks sent emails that need follow-up by analyzing implicit and explicit deadlines. Identifies which emails need follow-up and when. Use for inbox zero maintenance.model: sonnettools: Read, Grep---# Follow-Up Tracker## PurposeEnsure no sent emails fall through the cracks by tracking expected response times and alerting when follow-up is needed.## Deadline Detection### Explicit DeadlinesDirect mentions of dates or times:- "Please respond by Friday"- "Need this by EOD"- "Deadline: March 15"- "Before our meeting on Tuesday"### Implicit DeadlinesContext-based urgency without explicit dates:| Email Type          | Implicit Deadline   || ------------------- | ------------------- || Meeting-related     | Before meeting date || Proposal sent       | 5-7 business days   || First outreach      | 7 days              || Second follow-up    | 14 days from first  || Urgent request      | 24-48 hours         || Partnership inquiry | 10-14 days          |### No Follow-Up NeededRecognize when tracking isn't required:- FYI or informational emails- Emails explicitly marked "no reply needed"- Thank you or closing messages- Automated notifications- Emails where you're CC'd## Follow-Up ScheduleRecommended timing based on email type:| Email Type      | First Follow-Up | Second Follow-Up | Final  || --------------- | --------------- | ---------------- | ------ || Cold outreach   | Day 7           | Day 14           | Day 21 || Warm intro      | Day 5           | Day 10           | Day 15 || Proposal        | Day 5           | Day 10           | Day 14 || Meeting request | Day 3           | Day 7            | Day 10 || Urgent request  | Day 2           | Day 4            | Day 7  || Internal team   | Day 3           | Day 7            | -      |## Tracking LogicWhen analyzing sent emails:1. **Identify email type** from subject and content2. **Extract explicit deadlines** if present3. **Calculate implicit deadline** based on email type4. **Check for responses** in inbox5. **Determine follow-up status**:   - Overdue (past deadline, no response)   - Due today (deadline is today)   - Due soon (within 2 days)   - On track (before deadline)   - Resolved (response received)## Output FormatPresent tracking status in actionable format:| Email                          | Sent   | Follow-Up Due | Status                 || ------------------------------ | ------ | ------------- | ---------------------- || Partnership proposal to Marcus | Dec 20 | Dec 27        | ⚠️ Due today           || Meeting request to Sarah       | Dec 23 | Dec 28        | ⏰ Due in 1 day        || Cold outreach to Alex          | Dec 15 | Dec 22        | ❌ Overdue (5 days)    || Quarterly update to team       | Dec 24 | -             | ✅ No follow-up needed |## ActionsFor each tracked email, suggest:- **Overdue**: Generate follow-up email using /email-templates- **Due today**: Prioritize in today's task list- **Due soon**: Schedule for upcoming follow-up- **Resolved**: Mark as closed, remove from tracking## Response DetectionIdentify when email has been answered:- Direct reply in inbox (same thread)- Response from any recipient (not just TO:)- Meeting scheduled (for meeting requests)- Action taken (for approval requests)Mark as "Closed - [response type]" when detected.## Weekly SummaryGenerate weekly overview:> **Follow-Up Summary (Week of Dec 22)**>> Overdue (need immediate attention): 2> Due this week: 5> On track: 8> Closed this week: 12>> **Top priority follow-ups:**>> 1. Partnership proposal to Marcus (overdue 5 days)> 2. Meeting request to Sarah (due tomorrow)## IntegrationUses Gmail MCP to scan sent folder and inbox.Triggers `/email-templates follow-up` for follow-up drafting.Works with `inbox-triager` to correlate sent/received.
```

**Design rationale:**

-   **Implicit deadline detection** handles the common case (no explicit deadline stated)
-   **Status symbols** (⚠️ ❌ ⏰ ✅) enable quick scanning
-   **Action suggestions** turn tracking into doing

* * *

## The Decision Framework: Skills vs Subagents

With three subagents built, you can now apply a clear decision framework for future automation:

Use SKILL when...

Use SUBAGENT when...

Task is content generation

Task requires reasoning/classification

Output format is predictable

Output depends on analysis

User triggers directly (`/skill`)

System triggers automatically

Reusable template applies

Custom logic for each case

2-4 decision points

5+ decision points

Same steps every time

Steps vary by context

**Practical examples:**

Task

Component

Reasoning

Draft email with consistent tone

Skill

Predictable format, user-triggered

Prioritize 50 inbox emails

Subagent

Analysis required, context-dependent output

Apply follow-up template

Skill

Template substitution, predictable steps

Decide which emails need follow-up

Subagent

Deadline detection requires reasoning

Format meeting notes

Skill

Structure is consistent

Classify customer support tickets

Subagent

Category depends on content analysis

* * *

## Testing Your Subagents

With all three agents created, test them using the Task tool:

**Test inbox-triager:**

```
Use the inbox-triager agent to classify these emails:1. From: ceo@company.com   Subject: Board meeting prep - need numbers by 2pm   Body: Please send Q4 projections before board meeting.2. From: newsletter@techsite.com   Subject: This week in AI   Body: Top stories from the AI world...3. From: teammate@company.com   Subject: Sprint planning question   Body: Can you clarify the priority for the API feature?
```

**Expected output:**

```
| Priority | From | Subject | Reason ||----------|------|---------|--------|| Urgent | ceo@company.com | Board meeting prep | C-level sender, explicit 2pm deadline || Low | newsletter@techsite.com | This week in AI | Newsletter, FYI only || Important | teammate@company.com | Sprint planning question | Team member, project-related question |
```

**Test response-suggester:**

```
Use the response-suggester agent for this email:From: client@bigcorp.comSubject: Contract renewal timelineBody: We need to finalize the renewal by end of month. Can youconfirm your team's capacity for the expanded scope?
```

**Expected output:**

```
**Option 1 (Brief):**Thanks for reaching out. Yes, we can handle the expanded scope.I'll send capacity details by Thursday.**Option 2 (Detailed):**Thanks for the renewal discussion. We've reviewed the expanded scopeand can confirm capacity. Our team can absorb the additional workstarting Q1. I'll prepare a detailed capacity breakdown by Thursdayfor your review before month-end signing.**Option 3 (Alternative - Clarification):**Before I confirm capacity, could you share the specific scopeadditions you're considering? That will help me give you anaccurate timeline for the expanded engagement.
```

* * *

## Your Complete Agent Architecture

You now have three specialized subagents:

```
.claude/agents/├── inbox-triager.md      → Prioritizes incoming emails (5+ decision points)├── response-suggester.md → Generates reply options (tone analysis)└── follow-up-tracker.md  → Tracks sent email deadlines (implicit + explicit)
```

Combined with your skills from previous lessons:

```
.claude/skills/├── email-drafter/        → Consistent email voice├── email-templates/      → Reusable email structures└── email-summarizer/     → Thread parsing and action extraction
```

This architecture separates concerns:

-   **Skills** handle consistent execution (templates, formatting, tone)
-   **Subagents** handle reasoning (classification, prioritization, deadline detection)

The orchestration layer (Lesson 6) will combine these into a complete workflow.

* * *

## Try With AI

**Setup:** Ensure you have all three agent files created in `.claude/agents/`

**Prompt 1: Test Priority Classification**

```
I have 5 unread emails. Use the inbox-triager agent to classify them:1. From: direct-manager@company.com   Subject: Quick question about Friday's presentation   Body: Do you have the latest revenue slides?2. From: random-recruiter@linkedin.com   Subject: Exciting opportunity!   Body: Your profile caught my attention...3. From: key-client@bigcorp.com   Subject: URGENT - Production issue   Body: Our API integration is returning errors. Need immediate help.4. From: teammate@company.com   Subject: Coffee chat next week?   Body: Haven't caught up in a while. Free Tuesday?5. From: cfo@company.com (CC: you and 12 others)   Subject: FY24 Planning Update   Body: Attached is the latest planning document for review.
```

**What you're learning:** The inbox-triager applies multi-factor analysis. Notice how it weighs sender importance against content urgency. The CC'd email from CFO is NOT urgent despite the senior sender—being CC'd lowers priority.

**Prompt 2: Compare Response Approaches**

```
Use the response-suggester for this challenging email:From: frustrated-client@company.comSubject: RE: Delayed deliverableBody: This is the third time the deadline has slipped. I need tounderstand what's happening and when we can realistically expectdelivery. My leadership is asking questions.Generate 3 response options that balance empathy with professionalism.
```

**What you're learning:** The response-suggester adapts to emotional context. Each option should address the frustration while providing actionable information. Compare the brief vs detailed approaches—which fits your communication style?

**Prompt 3: Design Your Own Classification Logic**

```
I want to add a new classification to my inbox-triager: "Delegate"for emails that should be forwarded to someone else on my team.Help me define:1. What signals indicate an email should be delegated?2. What information should the output include for delegation?3. How does this interact with the existing priority levels?Update my inbox-triager agent definition with this new category.
```

**What you're learning:** Subagent design is iterative. You're extending the classification logic based on your actual workflow. This is Layer 3 in action—transforming tacit knowledge (you know which emails to delegate) into explicit, reusable intelligence.

**Safety Note:** Subagents can process emails quickly, but always review their classifications before taking action. Priority systems should augment your judgment, not replace it. An email misclassified as "Low" priority could contain something important that the automated analysis missed.

# Lesson 4 Summary: Creating Custom Subagents

## Key Concepts

1.  **Subagent Architecture**: Autonomous agents stored in `.claude/agents/[name].md` that can reason independently and use tools
    
2.  **Agent Definition Format**: YAML frontmatter with:
    
    -   `name`: Agent identifier
    -   `description`: **MUST be single-line** (multi-line breaks parsing)
    -   `tools`: Comma-separated list (Read, Grep, Glob, Edit)
3.  **Task Tool Delegation**: Invoke subagents using the Task tool with `subagent_type` parameter
    
4.  **Skills vs Subagents Decision Framework**: | Use Case | Decision Points | Choose | |----------|-----------------|--------| | Content generation | 2-4 | Skill | | Classification/reasoning | 5+ | Subagent | | External integration | N/A | MCP |
    
5.  **Three Email Subagents**:
    
    -   `inbox-triager`: Priority classification (Urgent/Important/Normal/Low)
    -   `response-suggester`: Generate 2-3 response options with tone variations
    -   `follow-up-tracker`: Identify emails needing follow-up with deadlines

## Deliverables

-   Three working subagents in `.claude/agents/`:
    -   `inbox-triager.md`
    -   `response-suggester.md`
    -   `follow-up-tracker.md`

## Key Code Snippets

### Agent Definition Format

```yaml
---
name: inbox-triager
description: Classify emails into Urgent/Important/Normal/Low priorities with reasoning
tools: Read, Grep, Glob
---

# Inbox Triager Agent

## Classification Criteria

### Urgent (respond within 2 hours)
- Explicit deadline today
- Executive sender
- Customer escalation

### Important (respond within 24 hours)
- Action items assigned to you
- Time-sensitive decisions
- Team dependencies

```

### Task Tool Invocation

```python
# In Claude Code, invoke subagent:
Task(
    subagent_type="inbox-triager",
    prompt="Classify these 5 emails by priority...",
    description="Triage inbox emails"
)

```

### Skills vs Subagents Decision

```markdown
## When to Use Each

**Use SKILL when:**
- Output format is predictable
- Logic has 2-4 decision points
- Guidance needed, not reasoning

**Use SUBAGENT when:**
- Autonomous reasoning required
- 5+ decision points
- Context-dependent responses
- Need to use multiple tools

```

## Try With AI Prompts

1.  Test inbox-triager with 5 sample email subjects/snippets
2.  Generate response options for a client complaint email
3.  Identify follow-up needs in a 2-week-old thread

## Skills Practiced

| Skill | Proficiency | Assessment | |-------|-------------|------------| | Subagent Architecture | B1 | Explain skill vs subagent | | Agent Definition Format | A2 | Create valid agent files | | Classification Logic | B1 | Design priority criteria | | Task Tool Delegation | A2 | Invoke subagents correctly | | Response Patterns | B1 | Generate tone variations | | Deadline Tracking | A2 | Detect implicit deadlines | | Decision Framework | B1 | Choose correct component |

## Duration

35 minutes

## Next Lesson

[Lesson 5: Gmail MCP Integration](./05-gmail-mcp-integration.md) - Connect to real Gmail

---
Source: https://agentfactory.panaversity.org/docs/Agent-Workflow-Primitives/build-first-ai-employee/hiring-specialists