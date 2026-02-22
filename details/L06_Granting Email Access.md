-   [](/)
-   [Part 2: Agent Workflow Primitives](/docs/Agent-Workflow-Primitives)
-   [Chapter 13: Build Your AI Employee](/docs/Agent-Workflow-Primitives/build-first-ai-employee)
-   L06: Granting Email Access

Updated Feb 21, 2026

[Version history](https://github.com/panaversity/ai-native-software-development/commits/main/apps/learn-app/docs/02-Agent-Workflow-Primitives/13-build-first-ai-employee/06-granting-email-access.md)

# Granting Email Access

You've built skills that draft professional emails. You've created subagents that triage inbox priorities and suggest responses. But there's a gap: your Email Assistant can't actually read or send emails. It's like having a brilliant assistant who can write perfect letters but has no mailbox.

The Gmail MCP server bridges this gap. It gives Claude Code direct, authenticated access to your Gmail inbox — not simulated emails, but your actual messages. Search for emails from your boss. Draft responses that appear in your Gmail Drafts folder. Send messages when you're confident they're ready. With 19 specialized tools, Gmail MCP transforms your Email Assistant from a writing helper into a complete communication system.

This lesson walks you through two authentication paths: the quick SMTP method (2 minutes with an App Password) and the comprehensive OAuth method (10 minutes with full API access). You'll test the connection, explore the available tools, and establish safety protocols that protect you from automation mistakes.

* * *

## Gmail MCP Architecture

The Gmail MCP server acts as a secure bridge between Claude Code and your Gmail account. Here's how the components connect:

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐│   Claude Code   │────▶│   Gmail MCP     │────▶│   Gmail API     ││   (Your Agent)  │     │   (Bridge)      │     │   (Your Inbox)  │└─────────────────┘     └─────────────────┘     └─────────────────┘        │                       │                       │   "Search for                  │                 Authenticated   emails from                  │                 API requests   boss@company.com"      19 specialized                          tools available
```

**The key insight**: You're not giving Claude Code your Gmail password. Instead, Gmail MCP uses either an App Password (limited access token) or OAuth credentials (Google's official authorization system) to make authenticated requests on your behalf.

* * *

## The 19 Gmail MCP Tools

Gmail MCP provides a comprehensive toolkit organized into four categories:

### Email Operations (Core)

Tool

Purpose

Example Use

`send_email`

Send email immediately

Routine messages after review

`draft_email`

Create draft for review

Important messages requiring approval

`read_email`

Get full email content

Understanding thread context

`search_emails`

Find emails by query

"Emails from boss this week"

`modify_email`

Change labels/read status

Mark as read, archive

`delete_email`

Remove email permanently

Cleanup old messages

`batch_modify_emails`

Bulk label operations

Archive all newsletters

`batch_delete_emails`

Bulk deletion

Clear spam folder

`download_attachment`

Save file from email

Extract report attachments

### Label Management

Tool

Purpose

Example Use

`list_email_labels`

Show all labels

See inbox structure

`create_gmail_label`

Create new label

Organize by project

`update_gmail_label`

Modify label properties

Change label color

`delete_gmail_label`

Remove label

Cleanup unused labels

`get_or_create_gmail_label`

Ensure label exists

Idempotent setup

### Filter Management

Tool

Purpose

Example Use

`create_gmail_filter`

Set up auto-rules

Auto-archive newsletters

`list_gmail_filters`

Show existing filters

Audit current automation

`get_gmail_filter`

View filter details

Debug filter behavior

`delete_gmail_filter`

Remove filter

Disable auto-processing

`create_filter_from_template`

Quick filter setup

Common patterns

* * *

## Authentication: Two Paths

Gmail MCP supports two authentication methods. Choose based on your needs:

Method

Setup Time

Access Level

Best For

**SMTP + App Password**

~2 minutes

Basic send/receive

Quick testing, simple automation

**OAuth**

~10 minutes

Full API access

Production use, all 19 tools

**Recommendation**: Start with SMTP for this lesson. Move to OAuth if you need label management, filters, or batch operations.

* * *

## Path A: SMTP Authentication (2 Minutes)

This method uses Google's App Passwords — special 16-character passwords that grant limited access without exposing your main account credentials.

### Step 1: Generate an App Password

**Requirements**: Your Google account must have 2-Factor Authentication enabled.

1.  Open [https://myaccount.google.com/apppasswords](https://myaccount.google.com/apppasswords)
2.  Sign in if prompted
3.  Under "App name", type: `Gmail MCP`
4.  Click **Create**
5.  Copy the 16-character password (format: `xxxx xxxx xxxx xxxx`)

**Important**: Save this password immediately — Google only shows it once. Store it in a password manager, not a plain text file.

### Step 2: Add Gmail MCP to Claude Code

Choose your scope:

Scope

Flag

Use When

🌍 **User Scope** (Recommended)

`--scope user`

Available in all your projects

📁 **Project Scope**

`--scope project`

Only in current project

**Project scope** (current project only):

```
claude mcp add gmail --transport http \  https://deep-red-marten.fastmcp.app/mcp \  --header "X-Gmail-Email: your-email@gmail.com" \  --header "X-Gmail-Password: xxxx-xxxx-xxxx-xxxx"
```

**User scope** (available in all projects - recommended):

```
claude mcp add gmail --transport http --scope user \  https://deep-red-marten.fastmcp.app/mcp \  --header "X-Gmail-Email: your-email@gmail.com" \  --header "X-Gmail-Password: xxxx-xxxx-xxxx-xxxx"
```

**Replace these values:**

-   `your-email@gmail.com` → Your actual Gmail address
-   `xxxx-xxxx-xxxx-xxxx` → The 16-character App Password you generated

**Output:**

```
Added gmail to user settings
```

.mcp.json Conflict

If you have a `.mcp.json` file in your project with a `gmail` server configured, it may override your `claude mcp add` config. Either:

-   Delete the conflicting entry from `.mcp.json`, or
-   Use a different server name (e.g., `gmail-smtp`)

### Step 3: Verify Connection

Test that Gmail MCP is working:

```
claude mcp list
```

**Output:**

```
gmail: connected  Transport: HTTP (remote)  Endpoint: https://deep-red-marten.fastmcp.app/mcp  Tools: 19
```

If you see `Tools: 19`, your connection is working.

* * *

## Path B: OAuth Authentication (10 Minutes)

OAuth provides full API access and is required for label management, filters, and some advanced operations. This method is more complex but more powerful.

### Step 1: Create Google Cloud Project

1.  Open [Google Cloud Console](https://console.cloud.google.com/)
2.  Click the project dropdown (top left, near the Google Cloud logo)
3.  Click **New Project**
4.  Enter project name: `gmail-mcp-integration`
5.  Click **Create**
6.  Wait for project creation (30-60 seconds)

### Step 2: Enable Gmail API

1.  In your new project, go to **APIs & Services** > **Library**
2.  Search for "Gmail API"
3.  Click **Gmail API** in results
4.  Click **Enable**

**Output confirmation**: You'll see "Gmail API" appear in your Enabled APIs list.

### Step 3: Configure OAuth Consent Screen

1.  Go to [Google Auth Platform](https://console.cloud.google.com/auth/overview) (or **APIs & Services** > **OAuth consent screen**)
2.  If prompted, click **Get Started** or **Configure Consent Screen**
3.  Select **External** → Click **Create**

**Configure Branding** (left sidebar):

1.  App name: `Gmail MCP`
2.  User support email: Your email
3.  Developer contact: Your email
4.  Click **Save**

**Configure Data Access** (left sidebar):

1.  Click **Add or Remove Scopes**
2.  In the filter, search for `gmail`
3.  Select these scopes:
    -   `https://www.googleapis.com/auth/gmail.modify`
    -   `https://www.googleapis.com/auth/gmail.settings.basic`
4.  Click **Update** → **Save**

**Configure Audience** (left sidebar):

1.  Click **Add Users**
2.  Add your Gmail address
3.  Click **Save**

**Publish Your App** (Summary in left sidebar):

1.  Click **Publish App** to move from "Testing" to "In Production"
2.  This allows your app to request full permissions needed for all Gmail MCP tools

Why Publish?

While in "Testing" mode, tokens expire after 7 days. Publishing removes this limitation for your own use.

### Step 4: Create OAuth Credentials

1.  Go to **Clients** (left sidebar) or **APIs & Services** > **Credentials**
2.  Click **Create Client** or **Create Credentials** > **OAuth client ID**
3.  Application type: **Web application**
4.  Name: `Gmail MCP`
5.  Under **Authorized redirect URIs**, click **Add URI** and enter:
    
    ```
    https://developers.google.com/oauthplayground
    ```
    
6.  Click **Create**
7.  Copy your **Client ID** and **Client Secret** (save them securely!)

### Step 5: Get Refresh Token (OAuth Playground)

1.  Go to [Google OAuth Playground](https://developers.google.com/oauthplayground)
2.  Click ⚙️ **Settings** (gear icon, top right)
3.  Check ✅ **Use your own OAuth credentials**
4.  Enter your **Client ID** and **Client Secret**
5.  Close settings
6.  In left panel, find **Gmail API v1** and select:
    -   `https://www.googleapis.com/auth/gmail.modify`
    -   `https://www.googleapis.com/auth/gmail.settings.basic`
7.  Click **Authorize APIs**
8.  Sign in with your Google account and click **Allow**
9.  Click **Exchange authorization code for tokens**
10.  Copy the **Refresh Token** from the response

### Step 6: Add Gmail MCP with OAuth

Choose your scope:

Scope

Flag

Use When

🌍 **User Scope** (Recommended)

`--scope user`

Available in all your projects

📁 **Project Scope**

(default)

Only in current project

**Project scope** (current project only):

```
claude mcp add gmail --transport http \  https://deep-red-marten.fastmcp.app/mcp \  --header "X-Gmail-Client-Id: YOUR_CLIENT_ID" \  --header "X-Gmail-Client-Secret: YOUR_CLIENT_SECRET" \  --header "X-Gmail-Refresh-Token: YOUR_REFRESH_TOKEN"
```

**User scope** (available in all projects - recommended):

```
claude mcp add gmail --transport http --scope user \  https://deep-red-marten.fastmcp.app/mcp \  --header "X-Gmail-Client-Id: YOUR_CLIENT_ID" \  --header "X-Gmail-Client-Secret: YOUR_CLIENT_SECRET" \  --header "X-Gmail-Refresh-Token: YOUR_REFRESH_TOKEN"
```

**Replace the placeholder values with your actual credentials.**

**Output:**

```
Added gmail to user settings
```

.mcp.json Conflict

If you have a `.mcp.json` file in your project with a `gmail` server configured for SMTP, it may override your OAuth config. Either:

-   Update `.mcp.json` to use OAuth headers, or
-   Delete the `gmail` entry from `.mcp.json`, or
-   Use a different server name (e.g., `gmail-oauth`)

**Available Tools with OAuth:**

Category

Tools

**Email Operations**

`send_email`, `draft_email`, `read_email`, `search_emails`, `modify_email`, `delete_email`

**Label Management**

`list_email_labels`, `create_gmail_label`, `delete_gmail_label`

**Filter Management**

`create_gmail_filter`, `list_gmail_filters`, `delete_gmail_filter`

* * *

## Testing Your Connection

Once authenticated (either method), test that Gmail MCP works correctly.

### Test 1: List Labels

Open Claude Code and ask:

```
List my Gmail labels
```

**Expected output:**

```
You: List my Gmail labelsClaude: [Uses list_email_labels tool]Found 12 labels:- INBOX (42 unread)- SENT- DRAFTS (3)- STARRED- IMPORTANT- TRASH- SPAM- Category/Primary- Category/Social- Category/Updates- Projects/Q4-Planning- Clients/Acme-Corp
```

If you see your labels, the connection works.

### Test 2: Search Emails

```
Search for emails from my boss in the last week
```

Claude interprets this and constructs the Gmail search query:

```
You: Search for emails from my boss in the last weekClaude: [Uses search_emails tool with query "from:boss@company.com after:2025/12/25"]Found 3 emails:1. "Q4 Review Meeting" - Dec 30   From: boss@company.com   Preview: "Following up on our discussion about..."2. "Budget Approval Needed" - Dec 28   From: boss@company.com   Preview: "Please review the attached budget..."3. "Team Update" - Dec 27   From: boss@company.com   Preview: "Great work on the project milestone..."
```

### Test 3: Create a Draft

```
Draft an email to john@example.com about rescheduling our meeting to next Tuesday at 3pm
```

**Expected output:**

```
You: Draft an email to john@example.com about rescheduling our meetingClaude: [Uses draft_email tool]Draft created successfully!To: john@example.comSubject: Rescheduling Our MeetingHi John,I hope this message finds you well. I need to reschedule our meetingto next Tuesday at 3pm. Would that time work for you?Please let me know if you need to adjust the time.Best regards---Draft saved to Gmail. View in Gmail: [link to Drafts folder]
```

The draft appears in your actual Gmail Drafts folder. You can review and edit it in Gmail before sending.

* * *

## Safety Protocols

Email automation is powerful but risky. A typo in a recipient address or an AI-generated phrase that doesn't match your intent can cause real problems. These protocols protect you.

### The Draft-First Rule

**For important emails, always use `draft_email` before `send_email`.**

Message Type

Recommended Approach

Cold outreach to clients

Draft → Review in Gmail → Send manually

Follow-ups with colleagues

Draft → Quick review → Send via Claude

Internal team updates

Send directly (lower risk)

Anything with attachments

Draft → Verify attachment → Send manually

Emails to executives

Draft → Review tone → Send manually

Defining "Known Contacts"

The L00 specification says email replies to "known contacts" can be auto-approved. Define this explicitly for your employee: known contacts are people you've exchanged **3+ emails with** in the past 90 days AND whose email address is in your contacts or CRM. Everyone else is a "new contact" requiring draft review. Encode this definition in your `Company_Handbook.md` or `AGENTS.md` so your employee applies it consistently.

**Implementation**: Tell Claude your preference:

```
Always create drafts for emails to external recipients.Only use send_email for internal team messages after I confirm.
```

### Sensitive Data Handling

**Never include in your prompts:**

Data Category

Why It's Risky

Passwords

Could be logged or cached

Financial account numbers

PII exposure

Personal health information

Privacy regulations (HIPAA)

Social Security Numbers

Identity theft risk

API keys or tokens

Security breach potential

**Safe alternative**: Reference information by description, not value:

```
# WRONG - exposes sensitive data"Send payment confirmation with account number 1234-5678-9012"# CORRECT - references without exposing"Send payment confirmation referencing the account on file"
```

### Email Safety Checklist

Before any automated send operation:

-    **Verify recipient address** — Is this the right person?
-    **Review subject line** — Does it accurately describe the content?
-    **Check for AI-generated errors** — Names spelled correctly? Dates accurate?
-    **Confirm attachments** — Are the right files attached?
-    **Test with yourself first** — Send to your own address before external recipients

### Recovery Procedures

**If you send an email in error:**

1.  **Within 30 seconds**: Use Gmail's "Undo Send" feature (if enabled)
2.  **After 30 seconds**: Send a follow-up correction email immediately
3.  **For serious errors**: Contact recipient directly (phone if necessary)

**Enable Undo Send**: Gmail Settings > General > "Undo Send" > Set to 30 seconds

* * *

## Integrating Gmail MCP with Your Skills

Gmail MCP becomes most powerful when combined with the skills you've already built.

### Connection Pattern

```
┌─────────────────────────────────────────────────────────────┐│                    Email Assistant System                    │├─────────────────────────────────────────────────────────────┤│                                                             ││   Skills (Expertise)          MCP (Access)                  ││   ┌───────────────────┐      ┌───────────────────┐         ││   │ /email-drafter    │──────▶│ draft_email      │         ││   │ Tone guidelines   │      │ send_email       │         ││   └───────────────────┘      └───────────────────┘         ││                                                             ││   ┌───────────────────┐      ┌───────────────────┐         ││   │ /email-summarizer │◀─────│ read_email       │         ││   │ Thread parsing    │      │ search_emails    │         ││   └───────────────────┘      └───────────────────┘         ││                                                             ││   Subagents (Reasoning)                                     ││   ┌───────────────────┐      ┌───────────────────┐         ││   │ inbox-triager     │◀────▶│ list_email_labels │         ││   │ Priority rules    │      │ modify_email     │         ││   └───────────────────┘      └───────────────────┘         ││                                                             │└─────────────────────────────────────────────────────────────┘
```

### Example: Complete Workflow

Here's how the components work together:

```
You: Triage my inbox and draft responses to urgent emailsClaude:1. [Uses search_emails] Scanning inbox...   Found 23 unread messages2. [Uses inbox-triager subagent] Prioritizing...   - 3 urgent (client emails awaiting response)   - 8 important (team updates)   - 12 routine (newsletters, notifications)3. [Uses read_email] Reading urgent emails...   Email 1: Client asking for project status   Email 2: Vendor invoice question   Email 3: Partner meeting request4. [Uses /email-drafter skill] Drafting responses...   Draft 1: Project status update to client   Draft 2: Invoice clarification to vendor   Draft 3: Meeting confirmation to partner5. [Uses draft_email] Creating drafts in Gmail...   All 3 drafts saved. Ready for your review.
```

* * *

## Troubleshooting Common Issues

### "Authentication Failed" Error

**Symptoms**: Gmail MCP can't connect after setup.

**Solutions**:

1.  **App Password method**: Regenerate the App Password and try again
2.  **OAuth method**: Check that refresh token hasn't expired; re-authorize if needed
3.  **Both methods**: Verify 2FA is enabled on your Google account

### "Insufficient Permission" Error

**Symptoms**: Some tools work but others fail.

**Solution**: This usually means OAuth scopes are incomplete. Re-run the OAuth Playground step and ensure you select all required scopes.

### "Rate Limited" Error

**Symptoms**: Tools work initially but fail after many operations.

**Solution**: Gmail API has rate limits. Wait 60 seconds and try again. For production use, implement exponential backoff.

### Drafts Not Appearing

**Symptoms**: `draft_email` returns success but you don't see the draft.

**Solutions**:

1.  Check the Gmail Drafts folder (not Inbox)
2.  Refresh your Gmail page
3.  Check if drafts are synced across devices (may take 30-60 seconds)

* * *

## Try With AI

**Setup:** Open Claude Code with Gmail MCP configured (either SMTP or OAuth method).

**Prompt 1: Explore Your Inbox**

```
Use Gmail MCP to give me an overview of my inbox. Show me:1. How many unread emails I have2. My top 3 most active labels3. Any emails from the last 24 hours that might need urgent attentionAfter showing results, suggest which email I should respond to first and why.
```

**What you're learning:** This tests the search and analysis capabilities of Gmail MCP. You're seeing how Claude interprets "urgent" based on sender, subject, and timing. Notice whether the AI's prioritization matches your own judgment — this reveals where you might want to add custom priority rules to your skills.

**Prompt 2: Draft with Safety**

```
Draft a follow-up email to a client about our project status. The client isMaria Santos at Acme Corp. The project is on track for the January 15 deadline.Before creating the draft:1. Search for any recent emails from Maria to understand our latest context2. Apply my email-drafter tone guidelines3. Create the draft for my review (don't send)Show me the draft and explain what context you used from previous emails.
```

**What you're learning:** This demonstrates the integration between Gmail MCP (search, draft) and your skills (tone guidelines). The search-before-draft pattern ensures your response builds on existing conversation context rather than starting fresh. Evaluate: Did Claude find relevant context? Did the tone match your guidelines?

**Prompt 3: Establish Your Safety Workflow**

```
Help me set up a safe email automation workflow. Based on my actual inbox:1. Identify which senders I email most frequently2. Categorize them as "safe for auto-send" vs "always draft-first"3. Create a simple rule document I can referenceShow me the category list and ask for my feedback before finalizing.
```

**What you're learning:** This builds your personal safety protocol based on real data from your inbox. The collaborative aspect — Claude proposes, you refine — ensures the final rules match your actual risk tolerance. This becomes a reusable reference for future automation.

**Safety Note:** When testing Gmail MCP, start with read-only operations (search, list labels) before attempting write operations (draft, send). Always verify drafts in Gmail before sending. For your first send test, use your own email address as the recipient.

# Lesson 5 Summary: Gmail MCP Integration

## Key Concepts

1.  **Gmail MCP Server**: External tool server providing 19 Gmail operations to Claude Code
    
2.  **Authentication Methods**:
    
    -   **SMTP + App Password** (2 min): Quick setup for basic send/receive
    -   **OAuth** (10 min): Full API access with granular permissions
3.  **Gmail MCP Tools** (19 total): | Category | Tools | |----------|-------| | Reading | `read_email`, `search_emails`, `get_email_thread` | | Sending | `send_email`, `draft_email`, `reply_to_email` | | Organization | `list_email_labels`, `create_label`, `apply_label` | | Management | `archive_email`, `delete_email`, `mark_as_read` |
    
4.  **Safety Protocols**:
    
    -   Draft-first workflow: Always create drafts before sending
    -   Sensitive data handling: Never include passwords/tokens in prompts
    -   Confirmation gates: Require explicit approval for destructive actions
5.  **MCP + Skills Integration**: Combine Gmail MCP tools with email skills for complete automation
    

## Deliverables

-   Working Gmail MCP connection (SMTP or OAuth)
-   Tested Gmail operations (list labels, search, draft)
-   Documented safety protocols

## Key Code Snippets

### SMTP Setup (2 minutes)

```bash
# Add Gmail MCP with App Password
claude mcp add gmail --scope user -- npx mcp-remote \
  https://deep-red-marten.fastmcp.app/mcp \
  --header "X-Gmail-Email: your-email@gmail.com" \
  --header "X-Gmail-Password: your-app-password"

```

### OAuth Setup (10 minutes)

```bash
# 1. Create Google Cloud project
# 2. Enable Gmail API
# 3. Create OAuth credentials
# 4. Configure consent screen
# 5. Add to Claude Code with OAuth token

```

### Test Gmail Connection

```
# In Claude Code:
"List all my Gmail labels"
→ Gmail MCP tool: list_email_labels

"Search for emails from sarah@techcorp.com this week"
→ Gmail MCP tool: search_emails

```

### Safety Protocol: Draft-First

```markdown
## Draft-First Workflow

1. User requests email send
2. Claude drafts to Gmail Drafts folder
3. User reviews draft in Gmail
4. User confirms send
5. Claude executes send_email

NEVER: Skip draft review for external recipients

```

## Try With AI Prompts

1.  "List all my Gmail labels" - Test basic connection
2.  "Search for emails from \[person\] about \[topic\]" - Test search
3.  "Draft a response to the latest email from \[person\]" - Test draft creation

## Skills Practiced

| Skill | Proficiency | Assessment | |-------|-------------|------------| | Gmail MCP Configuration | B1 | Configure with SMTP or OAuth | | App Password Creation | A2 | Generate Google App Password | | OAuth Credential Setup | B1 | Create Cloud project and credentials | | Gmail MCP Tool Usage | B1 | Invoke search, draft, send tools | | Email Safety Protocols | B1 | Apply draft-first workflow |

## Duration

30 minutes

## Next Lesson

[Lesson 6: Orchestrating Complete System](./06-orchestrating-complete-system.md) - Combine everything into Email Digital FTE

---
Source: https://agentfactory.panaversity.org/docs/Agent-Workflow-Primitives/build-first-ai-employee/granting-email-access