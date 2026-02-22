-   [](/)
-   [Part 2: Agent Workflow Primitives](/docs/Agent-Workflow-Primitives)
-   [Chapter 13: Build Your AI Employee](/docs/Agent-Workflow-Primitives/build-first-ai-employee)
-   L02: Teaching to Write

Updated Feb 21, 2026

[Version history](https://github.com/panaversity/ai-native-software-development/commits/main/apps/learn-app/docs/02-Agent-Workflow-Primitives/13-build-first-ai-employee/02-teaching-your-employee-to-write.md)

# Teaching Your Employee to Write

You send the same types of emails every week. Follow-up messages after meetings. Cold outreach to potential clients. Meeting requests with calendar coordination. Each one takes 5-15 minutes to craft well, and you write dozens of them monthly.

The problem isn't that you can't write emails. The problem is that your email voice lives entirely in your head. Every time you start a new message, you're reconstructing the same decisions: How formal should this be? Should I use a brief opener or get straight to the point? What's my standard sign-off?

What if your email preferences were encoded once and applied consistently forever? That's what you'll build in this lesson: an **email-drafter skill** that understands your tone, follows your communication patterns, and produces drafts that sound like you wrote them. The skill becomes a reusable asset you can invoke with `/email-drafter` anytime you need professional correspondence.

What You'll Build From Scratch

In this lesson, you'll create everything yourself—no templates, no starter code. By the end, you'll have built:

-   A new project directory (`email-assistant/`)
-   Your first skill folder (`.claude/skills/email-drafter/`)
-   A complete `SKILL.md` file with your expertise encoded
-   A `tone-guidelines.md` reference document capturing your voice
-   A project-level `CLAUDE.md` for context

Every file you create in this chapter becomes part of your **Email Digital FTE**—a reusable, portable system you own.

* * *

## Understanding the Skills Architecture

Quick Recap from Chapter 3

You learned about skills in Chapter 3 (Lessons 5-7). This section is a brief refresher before we build our first real skill system. If you're comfortable with the skills architecture, skim this section and jump to "Setting Up Your Project."

Before creating your first skill, you need to understand where skills live and how Claude Code finds them.

**Skills are folders, not files.** This is a critical distinction. While a skill's core instructions live in a single `SKILL.md` file, that file sits inside a folder that can contain supporting materials: templates, reference documents, example outputs, and configuration files.

The folder structure looks like this:

```
email-assistant/├── .claude/│   └── skills/│       └── email-drafter/          # The skill folder│           ├── SKILL.md            # Core instructions (always loaded)│           └── references/         # Supporting files (loaded when needed)│               └── tone-guidelines.md└── CLAUDE.md                       # Project-level context
```

**Why folders instead of single files?** Think of your smartphone. You have 100 apps installed, but your phone doesn't run all of them at once—it would crash. Apps stay closed until you tap them. Skills work the same way: Claude Code knows skills exist from brief descriptions, but only loads full instructions when you invoke a skill.

This three-level architecture protects Claude's working memory:

Level

What's Loaded

When

**Level 1**

Brief description (from YAML frontmatter)

Always (skill discovery)

**Level 2**

Full SKILL.md content

On skill invocation

**Level 3**

Supporting files (references/)

Only if skill needs them

* * *

## Setting Up Your Project

In Chapter 3, you learned about skills architecture and the SKILL.md format. Now you'll apply that knowledge by building a real skill system from the ground up.

**Create the project directory.** Open your terminal and navigate to where you keep your projects:

```
mkdir email-assistantcd email-assistant
```

**Create the Claude Code skill directories:**

```
mkdir -p .claude/skills/email-drafter/references
```

**Output:**

```
(no output - directories created silently)
```

**Verify the structure:**

```
find .claude -type d
```

**Output:**

```
.claude.claude/skills.claude/skills/email-drafter.claude/skills/email-drafter/references
```

You now have the correct directory structure. The `-p` flag creates all parent directories as needed, so `.claude/` and `.claude/skills/` are created automatically.

* * *

## Creating Your SKILL.md File

The `SKILL.md` file is where you encode your expertise. It tells Claude Code:

-   **When** to use this skill (activation triggers)
-   **What** the skill does (capabilities)
-   **How** to apply it (instructions and patterns)

Here's the complete email-drafter skill. Create this file at `.claude/skills/email-drafter/SKILL.md`:

```
---name: email-drafterdescription: This skill should be used when drafting professional emails. Use when the user needs to compose cold outreach, follow-ups, meeting requests, or any professional correspondence. Follows tone guidelines for consistent voice.---# Email Drafter## OverviewDraft professional emails that match your personal tone and communication style.## When to Use This Skill- Composing cold outreach emails- Writing follow-up messages- Drafting meeting requests- Any professional email correspondence## How It Works1. Read tone guidelines from references/tone-guidelines.md2. Understand the email context and purpose3. Draft email matching tone specifications4. Suggest subject line options## Email StructureEvery professional email follows this structure:- **Subject line**: Specific, scannable, action-oriented- **Opening**: Context reminder or value lead- **Body**: Short paragraphs, one idea per paragraph- **Call-to-action**: Specific, easy, with deadline## Tone ApplicationApply tone guidelines for:- Formality level (formal/professional/casual)- Warmth (warm/neutral/direct)- Length (concise/standard/detailed)## Output FormatProvide:1. Subject line options (2-3)2. Email body with proper formatting3. Suggested closingAlways ask for refinement preferences before finalizing.
```

**Key elements in this SKILL.md:**

1.  **YAML frontmatter** (`---` markers): Contains the `name` and `description` fields that Claude Code uses to discover and categorize the skill.
    
2.  **Description format**: Notice it starts with "This skill should be used when..." — this pattern helps Claude Code understand activation triggers.
    
3.  **Structured sections**: Overview, When to Use, How It Works, and Output Format provide clear guidance for consistent behavior.
    

* * *

## Defining Your Tone Guidelines

The tone guidelines document captures your personal communication style. This is the "expertise" you're encoding — the voice that makes emails sound like you.

Create this file at `.claude/skills/email-drafter/references/tone-guidelines.md`:

```
# Tone Guidelines for Email Communication## My Professional Voice### Formality Level: Professional- Use first names after initial contact- Avoid overly formal language ("Dear Sir/Madam")- No slang or very casual expressions### Warmth: Warm-Professional- Brief personal touch when appropriate- Skip small talk in follow-ups- Acknowledge recipient's time### Length: Concise- Maximum 5 sentences for routine messages- 3 paragraphs max for complex topics- Get to the point quickly### Language Patterns- Active voice preferred- Avoid passive constructions- No jargon unless recipient shares context### Signature StyleBest,[Name](No full titles unless formal context requires it)
```

This document captures specific, actionable preferences. "Professional" is vague, but "Use first names after initial contact" is concrete. "Concise" is subjective, but "Maximum 5 sentences for routine messages" is measurable.

* * *

## Creating the Project CLAUDE.md

The project-level `CLAUDE.md` file provides context about your project to Claude Code. Create this at `email-assistant/CLAUDE.md`:

```
# Email Assistant Project## OverviewThis project contains skills and subagents for automated email communication.## Project Structure- `.claude/skills/` - Reusable email skills- `.claude/agents/` - Specialized email processing subagents## Skills Available- `/email-drafter` - Draft professional emails with tone consistency## UsageInvoke skills with `/skill-name` syntax.
```

This file helps Claude Code understand the project context when you open the `email-assistant` folder.

* * *

## Verifying Your Complete Structure

Before testing, verify your project structure is correct:

```
find . -name "*.md" -o -type d | grep -E "(\.claude|CLAUDE)" | sort
```

**Output:**

```
./.claude./.claude/skills./.claude/skills/email-drafter./.claude/skills/email-drafter/references./.claude/skills/email-drafter/references/tone-guidelines.md./.claude/skills/email-drafter/SKILL.md./CLAUDE.md
```

Your complete directory structure should look like this:

```
email-assistant/├── .claude/│   └── skills/│       └── email-drafter/│           ├── SKILL.md│           └── references/│               └── tone-guidelines.md└── CLAUDE.md
```

If any files are missing, create them now using the content provided in the previous sections.

* * *

## Invoking Your First Skill

Now for the moment of truth: using your skill. Open Claude Code in the `email-assistant` directory and invoke the email-drafter:

```
/email-drafterI need to write a follow-up email to someone I met at a conference last week.We discussed a potential collaboration on a data analytics project.Their name is Sarah Chen, and she's the VP of Data at TechCorp.
```

**Expected output:**

Claude Code should:

1.  Load your email-drafter skill
2.  Read your tone guidelines
3.  Draft an email following your specifications

A well-formed response might look like:

```
**Subject Line Options:**1. Following up on our analytics conversation at TechConf2. Data collaboration discussion - next steps3. Great connecting at TechConf, Sarah**Email Draft:**Hi Sarah,Great meeting you at TechConf last week. Our conversation about data analyticscollaboration stuck with me, especially your insights on cross-team data sharing.I'd love to continue that discussion and explore what a partnership might look like.Would you have 30 minutes next week for a call?Best,[Your Name]---**Notes on this draft:**- Used first name per your tone guidelines- Kept to 3 short paragraphs (concise preference)- Warm-professional opening with specific callback- Clear, single call-to-actionWould you like me to adjust the formality, add more detail about thecollaboration scope, or change the call-to-action?
```

* * *

## Refining Through Iteration

The real power emerges when you refine the output. Your first draft is rarely perfect — that's expected.

**You review the draft and notice:**

"The email is good, but I want to mention the specific analytics tool we discussed — Snowflake — to show I was paying attention. Also, make the call-to-action more specific with proposed times."

**Your refinement request:**

```
Add a reference to Snowflake integration we discussed.Also, suggest two specific time slots: Tuesday at 2pm or Thursday at 10am.
```

**Refined output:**

```
Hi Sarah,Great meeting you at TechConf last week. Our conversation about data analyticscollaboration stuck with me, especially your insights on Snowflake integrationfor cross-team data sharing.I'd love to continue that discussion and explore what a partnership might look like.Would Tuesday at 2pm or Thursday at 10am work for a 30-minute call?Best,[Your Name]
```

Notice what happened: you provided domain context (the Snowflake detail) that wasn't in the skill. The AI adapted to your feedback while maintaining the tone you specified. This is the collaborative pattern: **you provide context and corrections, the skill provides consistent structure and voice.**

* * *

## What Made This Skill Work

Your email-drafter skill succeeded because of specific design choices:

Design Element

Why It Works

**Specific activation triggers**

"When drafting cold outreach, follow-ups, meeting requests..." tells Claude exactly when to use this skill

**Concrete tone specifications**

"Maximum 5 sentences" is measurable; "professional" isn't

**Structured output format**

Subject options + body + closing creates predictable output

**Refinement invitation**

"Ask for refinement preferences" builds in the iteration loop

* * *

## Common Mistakes to Avoid

When creating skills, these patterns lead to poor results:

**Mistake 1: Vague descriptions**

```
# WRONGdescription: Helps with emails# CORRECTdescription: This skill should be used when drafting professional emails. Use when the user needs to compose cold outreach, follow-ups, meeting requests, or any professional correspondence.
```

**Mistake 2: Missing references folder**

Skills that need supporting documents (tone guidelines, templates) should store them in `references/`. Putting everything in SKILL.md creates an overloaded file.

**Mistake 3: No output structure**

Without specifying expected output format (subject lines + body + closing), each invocation produces different structures.

* * *

## Try With AI

**Setup:** Open Claude Code in the `email-assistant` directory with the email-drafter skill installed.

**Prompt 1: Test Skill Activation**

```
/email-drafterI need to write a cold outreach email to a potential client. They run a smallmarketing agency (about 15 people) and I want to offer my consulting servicesfor marketing automation. The contact is James Rodriguez, founder of BrightPath Marketing.
```

**What you're learning:** This tests whether your skill correctly activates and applies your tone guidelines to a new context. Review the output: Does it match your specified formality level? Is it the right length? Does it use your signature style?

**Prompt 2: Refine the Voice**

After receiving the first draft, try:

```
The opening feels too generic. Make it more specific by mentioning somethingabout marketing agencies struggling with automation at scale. Also, make thecall-to-action softer - suggest they reply if interested rather than proposing a meeting.
```

**What you're learning:** This demonstrates the collaborative refinement loop. You're teaching the AI about your preferences beyond what's in the tone guidelines. Notice how the skill maintains structure while adapting to your feedback.

**Prompt 3: Adapt to Your Domain**

```
Now I need a completely different email - a meeting request to my team aboutnext quarter's project planning. We need to discuss resource allocation andtimeline adjustments. Keep it internal and brief.
```

**What you're learning:** This tests skill versatility. The email-drafter should handle internal communication differently than cold outreach while maintaining your core voice. Evaluate: Did the formality shift appropriately? Did length stay concise?

**Safety Note:** When drafting emails with AI assistance, always review the final output before sending. AI may include details that don't match your actual situation or make promises you didn't intend to make.

# Lesson 1 Summary: Project Setup & Email Drafter Skill

## Key Concepts

1.  **Skills Architecture**: Skills are reusable expertise packages stored in `.claude/skills/[name]/SKILL.md` that Claude Code loads when invoked with `/skill-name`
    
2.  **SKILL.md Format**: Required YAML frontmatter with `name` and `description` fields, followed by markdown instructions that guide AI behavior
    
3.  **Directory Structure**:
    
    ```
    .claude/skills/email-drafter/
    ├── SKILL.md
    └── references/
        └── tone-guidelines.md
    
    ```
    
4.  **Tone Specification**: Reference files define consistent communication voice (formality, warmth, length, structure)
    
5.  **Skill Invocation**: Use `/email-drafter` syntax to activate skill with context loading
    

## Deliverables

-   Complete project directory structure with `.claude/skills/` and `.claude/agents/` folders
-   Working `/email-drafter` skill with proper YAML frontmatter
-   `references/tone-guidelines.md` with voice specification

## Key Code Snippets

### SKILL.md Structure

```yaml
---
name: email-drafter
description: Draft professional emails with consistent tone and structure
---

# Email Drafter

## Your Role
You are an email composition assistant...

## Workflow
1. Gather context (recipient, purpose, tone)
2. Apply tone guidelines from references/
3. Draft email following structure
4. Offer refinement options

```

### Tone Guidelines Structure

```markdown
# Tone Guidelines

## Formality Level
- Default: Professional but warm
- Adjustments: Formal for executives, casual for peers

## Voice Characteristics
- Active voice preferred
- Concise sentences
- Clear calls to action

```

## Try With AI Prompts

1.  `/email-drafter` → Create cold outreach to Sarah Chen about AI consulting
2.  Test tone consistency by drafting the same email with different recipients
3.  Refine voice by teaching AI your specific preferences

## Skills Practiced

| Skill | Proficiency | Assessment | |-------|-------------|------------| | Skill Directory Structure | A2 | Create proper structure | | SKILL.md Format | A2 | Write valid SKILL.md | | Tone Specification | A2 | Define voice guidelines | | Skill Invocation | A2 | Use `/skill-name` syntax |

## Duration

30 minutes

## Next Lesson

[Lesson 2: Email Templates Skill](./02-email-templates-skill.md) - Build template library with variable substitution

---
Source: https://agentfactory.panaversity.org/docs/Agent-Workflow-Primitives/build-first-ai-employee/teaching-your-employee-to-write