-   [](/)
-   [Part 2: Agent Workflow Primitives](/docs/Agent-Workflow-Primitives)
-   [Chapter 13: Build Your AI Employee](/docs/Agent-Workflow-Primitives/build-first-ai-employee)
-   L03: Professional Formats

Updated Feb 21, 2026

[Version history](https://github.com/panaversity/ai-native-software-development/commits/main/apps/learn-app/docs/02-Agent-Workflow-Primitives/13-build-first-ai-employee/03-teaching-professional-formats.md)

# Teaching Professional Formats

You've written the same cold outreach email a hundred times. Each time, you customize the recipient's name, personalize the hook, adjust the value proposition. It takes 10 minutes to get it right. Multiply that across dozens of prospects, and you've spent hours on what should take seconds.

The problem isn't that you lack an AI assistant. Claude can write emails. The problem is that Claude doesn't know YOUR patterns—the specific structure that works for your industry, the tone that matches your brand, the follow-up sequences that actually get responses.

This lesson changes that. You'll build an **email-templates skill** that encodes your email expertise into reusable intelligence. Instead of explaining your preferences every time, you'll invoke a skill that already knows your cold outreach structure, your follow-up timing philosophy, and your meeting request format.

* * *

## Why Template Skills Beat One-Off Prompting

Compare two approaches to sending 20 personalized outreach emails:

Approach

Time per Email

Consistency

Improvement

**Manual prompting**

3-5 minutes (explaining format each time)

Variable (depends on prompt quality)

None (starts fresh each time)

**Template skill**

30 seconds (fill variables, send)

High (same structure every time)

Compounds (refine template once, all emails improve)

The math is clear: 20 emails at 4 minutes each = 80 minutes. With templates: 20 emails at 30 seconds = 10 minutes. That's 70 minutes saved—per batch.

But time savings miss the bigger point. **Templates encode your expertise as reusable intelligence.** When you discover that a particular subject line format doubles your response rate, you update one template. Every future email benefits.

* * *

## Template Design Principles

Effective email templates share three characteristics:

**1\. Clear Placeholders**

Variables identify what changes between emails. Use `{{variable_name}}` syntax for consistency:

```
Good:  {{recipient_name}}, {{company}}, {{hook}}Bad:   [NAME], {company}, <insert hook here>
```

The double-brace syntax `{{...}}` is unambiguous. Claude recognizes it immediately. Your templates become self-documenting.

**2\. Structured Sections**

Each template needs:

-   **Purpose**: When to use this template
-   **Variables**: What information is required
-   **Template**: The actual email structure
-   **Example**: A filled-out version showing the pattern

Structure enables automation. When Claude sees a complete template, it can:

-   Prompt you for missing variables
-   Substitute values correctly
-   Validate the result makes sense

**3\. Anti-Patterns**

Templates should prevent common mistakes, not just provide structure:

```
## Anti-Patterns- Generic openers ("I hope this finds you well")- Wall of text (4-5 short paragraphs max)- Multiple CTAs (one clear ask only)- Obvious template feel
```

Documenting what NOT to do is as valuable as documenting what to do. It prevents regression when you're rushing.

* * *

## The Email Templates Skill Structure

Your skill will follow Claude Code's standard structure:

```
.claude/skills/└── email-templates/    ├── SKILL.md                 # Main skill instructions    └── templates/               # Template library        ├── cold-outreach.md     # First contact        ├── follow-up.md         # Re-engagement        └── meeting-request.md   # Scheduling calls
```

**Why this structure works:**

-   **SKILL.md** loads on-demand when Claude activates the skill (Level 2 loading)
-   **templates/** directory contains supporting files accessed when needed (Level 3 loading)
-   Each template is a separate file for easy editing and version control

* * *

## Building the SKILL.md

The SKILL.md file is the entry point. It tells Claude:

-   What this skill does
-   When to activate it
-   How to use the templates

Here's the complete SKILL.md for your email-templates skill:

```
---name: email-templatesdescription: This skill provides reusable email templates with variable substitution. Use when the user needs to send recurring email types like cold outreach, follow-ups, or meeting requests. Automatically selects appropriate template and fills variables.---# Email Templates## OverviewReusable email templates with variable substitution for consistent, efficient communication.## When to Use This SkillActivate this skill when user needs to:- Send cold outreach to new contacts- Follow up on unanswered emails- Request meetings or calls- Draft any recurring email type## Available Templates### 1. Cold Outreach (`templates/cold-outreach.md`)For first contact with new connections. Focuses on:- Personalized hook (why reaching out to THEM)- Brief credibility (proof you're worth hearing)- Clear value proposition (what's in it for them)- Low-friction CTA (easy next step)### 2. Follow-Up (`templates/follow-up.md`)For re-engaging after no response. Focuses on:- Non-accusatory reference to previous message- New value addition (not just "checking in")- Easier response options### 3. Meeting Request (`templates/meeting-request.md`)For scheduling calls or meetings. Focuses on:- Clear context (why meeting makes sense)- Specific agenda (what you'll discuss)- Multiple time options (respect their calendar)## Variable SyntaxTemplates use `{{variable_name}}` syntax:| Variable             | Description              || -------------------- | ------------------------ || `{{recipient_name}}` | First name of recipient  || `{{company}}`        | Recipient's company name || `{{topic}}`          | Email subject matter     || `{{sender_name}}`    | Your name                |Additional template-specific variables are documented in each template file.## Workflow1. **Identify email type** from user request2. **Load appropriate template** from templates/ directory3. **Gather variable values** from context or ask user4. **Substitute variables** into template5. **Apply tone adjustments** if user has preferences6. **Present filled template** for review and refinement## Integration NotesThis skill works well with:- Gmail MCP for direct sending- Calendar tools for meeting request coordination- CRM data for personalization
```

**What makes this effective:**

-   **Clear activation triggers**: "when user needs to send recurring email types"
-   **Template overview**: Quick reference for what's available
-   **Variable documentation**: Consistent substitution patterns
-   **Workflow steps**: Claude knows the process to follow

* * *

## Creating the Cold Outreach Template

The cold outreach template is your highest-impact email type. First impressions matter. Here's the complete template:

```
# Cold Outreach Template## PurposeFirst contact with someone who doesn't know you. The goal is to earn a response, not close a deal.## Variables| Variable             | Description                           | Example                                               || -------------------- | ------------------------------------- | ----------------------------------------------------- || `{{recipient_name}}` | First name                            | "Sarah"                                               || `{{hook}}`           | Why reaching out to THEM specifically | "Your Kubernetes talk at KubeCon"                     || `{{hook_subject}}`   | Subject line hook                     | "Your Kubernetes talk - question from a practitioner" || `{{credibility}}`    | Brief proof you're worth hearing      | "I've helped 200+ developers adopt similar patterns"  || `{{value_prop}}`     | What's in it for them                 | "Include your approach as a case study"               || `{{cta}}`            | Low-friction next step                | "15-minute call next week"                            || `{{sender_name}}`    | Your name                             | "Alex"                                                |## TemplateSubject: {{hook_subject}}Hi {{recipient_name}},{{hook}}{{credibility}}{{value_prop}}{{cta}}Best,{{sender_name}}## Example Filled**Subject:** Your Kubernetes talk - question from a practitionerHi Sarah,I saw your talk on scaling microservices at KubeCon. Your approach to service mesh configuration solved a problem I've been wrestling with for weeks.I'm building a course on cloud-native development and have helped 200+ developers adopt similar patterns.Would a 15-minute call work next week to discuss including your approach as a case study?Best,Alex## Anti-PatternsThese patterns reduce response rates:| Pattern                      | Problem                | Instead                             || ---------------------------- | ---------------------- | ----------------------------------- || "I hope this finds you well" | Generic, wastes space  | Jump to personalized hook           || Long paragraphs              | Overwhelming on mobile | Keep to 2-3 sentences per paragraph || Multiple asks                | Confuses priority      | One clear CTA only                  || "I'm sure you're busy"       | Apologetic, low status | Confident, value-first framing      || Obvious template feel        | Breaks trust           | Genuine personalization in hook     |## Tone Guidance- **Confident but not arrogant**: You have value to offer- **Specific but not overwhelming**: One clear point- **Personal but not creepy**: Reference public information only- **Short but not abrupt**: 4-5 sentences total
```

**Key design choices:**

-   **Variable table with examples**: Claude knows exactly what to ask for
-   **Complete example**: Shows the pattern in action
-   **Anti-patterns table**: Prevents common mistakes
-   **Tone guidance**: Maintains voice consistency

* * *

## Creating the Follow-Up Template

Follow-ups fail when they sound desperate or accusatory. The key is adding value while making response easier:

```
# Follow-Up Template## PurposeRe-engage after no response without being pushy. Adds value instead of just "checking in."## Variables| Variable             | Description                 | Example                                      || -------------------- | --------------------------- | -------------------------------------------- || `{{recipient_name}}` | First name                  | "Marcus"                                     || `{{original_topic}}` | What first email was about  | "Partnership opportunity"                    || `{{new_value}}`      | Additional value or context | "Your v2.0 release makes this more relevant" || `{{easier_option}}`  | Lower-friction alternative  | "3-minute Loom video instead of call"        || `{{sender_name}}`    | Your name                   | "Alex"                                       |## TemplateSubject: Re: {{original_topic}} - one more thoughtHi {{recipient_name}},Wanted to circle back on my message about {{original_topic}}.{{new_value}}{{easier_option}}Best,{{sender_name}}## Example Filled**Subject:** Re: Partnership opportunity - one more thoughtHi Marcus,Wanted to circle back on my message about the documentation partnership from last week.Since then, I noticed your team released v2.0—congratulations! This actually makes the collaboration even more relevant since the new API patterns match what I'm documenting.Would it be easier to start with a quick async exchange? I could send over our proposed integration in a 3-minute Loom video instead of scheduling a call.Best,Alex## Timing GuidelinesStrategic timing increases response rates:| Follow-up | Timing                 | Approach                           || --------- | ---------------------- | ---------------------------------- || First     | 5-7 business days      | Add new value, maintain confidence || Second    | 10-14 days after first | Offer easier response option       || Final     | 21 days                | Give explicit permission to say no |## Anti-Patterns| Pattern                 | Problem                    | Instead                          || ----------------------- | -------------------------- | -------------------------------- || "Just checking in"      | No value, annoys recipient | Add new information or context   || "Did you see my email?" | Accusatory tone            | Assume they're busy, add value   || "I'll keep this brief"  | Draws attention to length  | Just be brief, don't announce it || Exact same message      | Shows no effort            | Reference something new          |
```

**Why this works:**

-   **New value requirement**: Forces you to earn the follow-up
-   **Easier option**: Reduces friction for busy recipients
-   **Timing guidelines**: Strategic spacing without harassment
-   **Permission to say no**: Final follow-up preserves relationship

* * *

## Creating the Meeting Request Template

Meeting requests fail when they lack context or make scheduling hard. The template solves both:

```
# Meeting Request Template## PurposeSchedule time efficiently by demonstrating value upfront. The recipient should know exactly why meeting is worth their time.## Variables| Variable             | Description                | Example                                            || -------------------- | -------------------------- | -------------------------------------------------- || `{{recipient_name}}` | First name                 | "Priya"                                            || `{{context}}`        | Why meeting makes sense    | "Based on our Slack thread about rate limiting"    || `{{topic}}`          | Meeting subject            | "API integration"                                  || `{{duration}}`       | Time commitment            | "30 min"                                           || `{{agenda_items}}`   | Specific topics (bulleted) | Review limits, discuss patterns, agree on solution || `{{time_options}}`   | 3 specific time slots      | Tuesday 2 PM, Wednesday 10 AM, Thursday 3 PM       || `{{sender_name}}`    | Your name                  | "Alex"                                             |## TemplateSubject: {{duration}} sync on {{topic}} - {{number}} time optionsHi {{recipient_name}},{{context}}Proposed agenda ({{duration}}):{{agenda_items}}Would any of these work?{{time_options}}{{sender_name}}## Example Filled**Subject:** 30-min sync on API integration - 3 time optionsHi Priya,Based on our Slack thread about the rate limiting issue, I think we'd resolve this faster in a quick call.Proposed agenda (30 min):- Review current rate limits (5 min)- Discuss your usage patterns (10 min)- Agree on solution approach (15 min)Would any of these work?- Tuesday 2-2:30 PM EST- Wednesday 10-10:30 AM EST- Thursday 3-3:30 PM ESTAlex## Best Practices| Practice                     | Why It Works                               || ---------------------------- | ------------------------------------------ || Include specific agenda      | Shows you've prepared, respects their time || Offer 3 time options minimum | Increases chance of fit                    || State duration clearly       | Sets expectations, shows respect           || Include time zone            | Prevents confusion, especially remote      || Time-boxed agenda items      | Demonstrates efficiency                    |## Anti-Patterns| Pattern                | Problem                  | Instead                      || ---------------------- | ------------------------ | ---------------------------- || "Let's hop on a call"  | Vague, no agenda         | Specific purpose with agenda || "When are you free?"   | Puts burden on them      | Offer specific times         || "Quick sync" (no time) | Unclear commitment       | State exact duration         || Back-to-back requests  | Assumes first time works | Offer alternatives upfront   |
```

* * *

## Template Selection Logic

Your SKILL.md defines when to use each template. But Claude needs to recognize intent from natural language:

**Cold Outreach indicators:**

-   "I want to reach out to..."
-   "First time contacting..."
-   "Introduce myself to..."
-   "Initial outreach to..."

**Follow-Up indicators:**

-   "Haven't heard back from..."
-   "Follow up on my previous..."
-   "Circle back with..."
-   "Re-engage with..."

**Meeting Request indicators:**

-   "Schedule a call with..."
-   "Set up a meeting..."
-   "Find time to discuss..."
-   "Book time with..."

Add these patterns to your SKILL.md's "When to Use" section to improve activation accuracy.

* * *

## Testing Your Skill

Create the skill structure and test with Claude:

**Step 1: Create the directory structure**

```
mkdir -p .claude/skills/email-templates/templates
```

**Step 2: Create SKILL.md and template files**

Copy the content from this lesson into:

-   `.claude/skills/email-templates/SKILL.md`
-   `.claude/skills/email-templates/templates/cold-outreach.md`
-   `.claude/skills/email-templates/templates/follow-up.md`
-   `.claude/skills/email-templates/templates/meeting-request.md`

**Step 3: Test activation**

Ask Claude:

```
"I need to reach out to Sarah Chen at DataFlow Inc. She gave a great talkon data pipelines at the recent conference. I want to explore a potentialpartnership for our developer education content."
```

**Expected behavior:**

1.  Claude recognizes this as cold outreach
2.  Loads the cold-outreach template
3.  Identifies variables: `{{recipient_name}}` = Sarah, `{{company}}` = DataFlow Inc
4.  Asks for remaining variables or infers from context
5.  Produces filled template for review

**Step 4: Refine through iteration**

Claude's first attempt may not match your voice perfectly. This is where AI collaboration shines:

**You**: "This sounds too formal. I want it warmer, more conversational."

**Claude adapts**: Revises tone while maintaining structure.

**You**: "The CTA is too aggressive. Soften it."

**Claude refines**: Adjusts call-to-action while keeping it clear.

Each refinement teaches you something about your preferences. Update the template's tone guidance based on what you learn.

* * *

## Extending Your Template Library

The three core templates handle most professional email needs. But you can extend for your domain:

Domain

Additional Templates

**Sales**

Demo request, pricing follow-up, contract renewal

**Recruiting**

Candidate outreach, interview scheduling, offer letter

**Investor Relations**

Intro request, update email, pitch deck follow-up

**Customer Success**

Onboarding check-in, renewal reminder, upsell intro

Each new template follows the same structure:

1.  Purpose
2.  Variables table
3.  Template
4.  Example filled
5.  Anti-patterns
6.  Tone guidance

The pattern is the reusable intelligence. Templates are just instantiations.

* * *

## What You Built

You now have a production-ready email templates skill:

Component

Purpose

**SKILL.md**

Entry point with activation triggers and workflow

**cold-outreach.md**

First contact template with personalization

**follow-up.md**

Re-engagement template with value-add approach

**meeting-request.md**

Scheduling template with agenda and options

This skill represents **Layer 3 intelligence**—tacit knowledge (your email patterns) transformed into explicit, reusable assets. Every email you send using these templates reinforces the pattern. Every refinement improves all future emails.

* * *

## Try With AI

**Create Your Cold Outreach Template:**

```
"Help me create a cold outreach email template for [your specific use case].I typically reach out to [target audience] about [your offering]. My usualstructure is [describe your current approach]. What variables should wedefine, and what anti-patterns should I avoid?"
```

**What you're learning:** How to translate your intuitive email patterns into explicit template structure. Notice how Claude asks clarifying questions—that's the variable identification process.

**Test Variable Substitution:**

```
"Using the cold outreach template structure, create an email for:- Recipient: [name] at [company]- Hook: [something specific about them]- My credibility: [your relevant experience]- Value prop: [what you're offering]- CTA: [your ask]Show me the filled template and explain which variables you substituted."
```

**What you're learning:** The mechanical process of variable substitution. This is what your skill automates—Claude gathers variables and produces consistent output.

**Refine Tone Through Iteration:**

```
"This template sounds too [formal/casual/aggressive/passive]. I want myemails to sound more [your desired tone]. Revise the template andupdate the tone guidance section to reflect this preference."
```

**What you're learning:** How bidirectional refinement works. You're teaching Claude your preferences; Claude is suggesting improvements you hadn't considered. The result is better than either starting point.

**Remember:** Always review AI-generated emails before sending. Templates provide structure; your judgment provides appropriateness for each specific recipient and context.

# Lesson 2 Summary: Email Templates Skill

## Key Concepts

1.  **Template Design**: Create email templates with `{{variable_name}}` placeholders for personalization
    
2.  **Variable Substitution**: Replace placeholders with actual values at generation time (recipient name, company, value proposition)
    
3.  **Template Library**: Organize templates in `templates/` directory with clear naming conventions
    
4.  **Template Selection Logic**: Define when-to-use criteria for each template type
    
5.  **References Directory**: Store supporting files that enhance skill capabilities
    

## Deliverables

-   Working `/email-templates` skill with proper SKILL.md
-   Three template files:
    -   `templates/cold-outreach.md`
    -   `templates/follow-up.md`
    -   `templates/meeting-request.md`

## Key Code Snippets

### Template Structure

```markdown
# Cold Outreach Template

Subject: {{subject_hook}}

Hi {{recipient_name}},

{{personalized_opener}}

I'm reaching out because {{value_proposition}}.

{{specific_ask}}

{{sign_off}}

```

### SKILL.md Workflow

```markdown
## Template Application Workflow

1. Identify template type from context
2. Read template from templates/
3. Collect variable values from user
4. Substitute all {{placeholders}}
5. Apply tone guidelines
6. Return completed email

```

### Variable Collection Pattern

```markdown
## Required Variables by Template

### Cold Outreach
- recipient_name: First name of recipient
- company: Their organization
- value_proposition: What you offer
- specific_ask: Clear next step

```

## Try With AI Prompts

1.  `/email-templates cold-outreach` → Sarah Chen at TechCorp about AI consulting
2.  Test variable substitution with multiple recipients
3.  Create a new template for investor updates

## Skills Practiced

| Skill | Proficiency | Assessment | |-------|-------------|------------| | Email Template Design | A2 | Create templates with placeholders | | Skill Creation | A2 | Write valid SKILL.md | | Template Library Organization | A2 | Organize templates logically | | Variable Substitution Patterns | A2 | Implement {{variable}} syntax | | Template Selection Logic | A2 | Define when-to-use criteria |

## Duration

25 minutes

## Next Lesson

[Lesson 3: Email Summarizer Skill](./03-email-summarizer-skill.md) - Parse threads and extract action items

---
Source: https://agentfactory.panaversity.org/docs/Agent-Workflow-Primitives/build-first-ai-employee/teaching-professional-formats