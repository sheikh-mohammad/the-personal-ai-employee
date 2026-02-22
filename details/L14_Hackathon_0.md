# L14: Hackathon 0 - Build Your Personal AI Employee

**Note:** This hackathon lesson is referenced throughout Chapter 13 as the capstone project. The specific URL for this lesson may not be available on the Agent Factory website, but the requirements are documented across the previous lessons.

## Overview

Hackathon 0 is your opportunity to build a complete Personal AI Employee project from scratch and submit it as a GitHub repository. Everything you practiced in L01-L13 -- pipeline integration, error recovery, audit logging, documentation -- becomes the quality bar for your hackathon submission.

## Project Requirements

Your submission must demonstrate all three tiers of achievement:

### Bronze Tier Requirements (Working Email Assistant)

- [ ] Obsidian vault with AGENTS.md, CLAUDE.md, Company_Handbook.md
- [ ] Four email skills: email-drafter, email-templates, email-summarizer, email-assistant (orchestrator)
- [ ] Three subagents: inbox-triager, response-suggester, follow-up-tracker
- [ ] Gmail MCP integration with working authentication
- [ ] Master orchestrator skill that coordinates all components
- [ ] Graceful degradation when MCP is offline

### Silver Tier Requirements (Proactive Assistant)

- [ ] File watcher using watchdog library
- [ ] Action file deposition to /Needs_Action/
- [ ] HITL approval workflow with four folders (Pending_Approval, Approved, Rejected, Done)
- [ ] Approval watcher script
- [ ] PM2 configuration for continuous operation
- [ ] Cron job for scheduled tasks (e.g., daily audit)
- [ ] Stop hook for persistent multi-turn work
- [ ] CEO Briefing skill with Business_Goals.md integration
- [ ] Weekly briefing generation with revenue tracking, bottleneck detection, subscription audit

### Gold Tier Requirements (Autonomous Employee)

- [ ] Dashboard.md with real-time status tracking
- [ ] End-to-end pipeline: event → watcher → orchestrator → HITL → action → log
- [ ] Error recovery with retry wrapper (exponential backoff)
- [ ] Authentication failure handling with alert generation
- [ ] Structured audit logging (JSON format, all required fields)
- [ ] README.md with complete architecture documentation
- [ ] Git version control with meaningful commits
- [ ] All 10 Gold Tier verification checks passing

## Deliverables

### 1. GitHub Repository

Create a public repository with this structure:

```
personal-ai-employee/
├── README.md              # Architecture documentation
├── LICENSE                # Open source license
├── .gitignore
├── vault/
│   ├── AGENTS.md
│   ├── CLAUDE.md
│   ├── Company_Handbook.md
│   ├── Business_Goals.md
│   ├── Dashboard.md
│   ├── .claude/
│   │   ├── skills/
│   │   └── agents/
│   ├── Needs_Action/
│   ├── Pending_Approval/
│   ├── Approved/
│   ├── Rejected/
│   ├── Done/
│   ├── Logs/
│   └── Briefings/
├── watchers/
│   ├── file_watcher.py
│   └── approval_watcher.py
├── scripts/
│   ├── error_recovery.py
│   └── logger.py
├── .mcp.json
└── pm2.config.js (optional)
```

### 2. Video Demo (Optional but Recommended)

Record a 5-10 minute video demonstrating:
1. File drop → watcher detection → action file creation
2. Orchestrator processing the request
3. HITL approval workflow
4. Email sending via Gmail MCP
5. Dashboard update
6. Audit log query
7. Error recovery demonstration

### 3. Written Report

Include in your README or as a separate document:
- Architecture decisions and trade-offs
- Challenges encountered and how you solved them
- What you would improve with more time
- Lessons learned about building autonomous systems

## Evaluation Criteria

Your submission will be evaluated on:

| Criterion | Weight | What Evaluators Look For |
|-----------|--------|-------------------------|
| **Completeness** | 25% | All Bronze, Silver, Gold requirements met |
| **Functionality** | 25% | System works end-to-end without manual intervention |
| **Safety** | 20% | HITL properly implemented, sensitive actions gated |
| **Resilience** | 15% | Error recovery works, graceful degradation |
| **Documentation** | 15% | README explains system clearly, code is commented |

## Submission Process

1. **Create your repository** on GitHub
2. **Test thoroughly** using the Gold Tier verification checklist
3. **Write your README** with setup instructions
4. **Submit** via the Agent Factory submission form or share on Discord

## Tips for Success

### Start Small, Iterate
Don't try to build everything at once. Get Bronze working first, then add Silver features, then Gold. Each tier should be functional before moving to the next.

### Test Each Component
Before integrating, verify each piece works independently:
- Test watchers without the orchestrator
- Test skills without MCP
- Test HITL workflow manually

### Document As You Go
Don't wait until the end to write documentation. Comment your code and update your README as you build. Future-you (and evaluators) will thank you.

### Use AI Pair Programming
Throughout the hackathon, use AI to:
- Debug error messages
- Generate boilerplate code
- Review your architecture decisions
- Write test cases

### Security First
Never commit:
- API keys or credentials
- Personal email content
- Real client information

Use environment variables and `.env` files (gitignored) for secrets.

## Common Pitfalls

| Pitfall | How to Avoid |
|---------|--------------|
| Watcher doesn't detect files | Test with a file created AFTER watcher starts |
| Cron job silently fails | Check macOS Full Disk Access for cron |
| PM2 processes lost after reboot | Run both `pm2 save` AND `pm2 startup` |
| Stop hook doesn't trigger | Verify exit code is 2 (not 1) for continue |
| Logs don't include all fields | Use the provided `log_action()` function |
| Dashboard doesn't update | Remember: manual updates for now, automation is a stretch goal |

## Stretch Goals

If you complete all requirements and have time, consider:

1. **Gmail Watcher** - Poll Gmail API for new messages (not just file watcher)
2. **Multi-domain support** - Add calendar or social media integration
3. **Automated Dashboard** - Scripts that update Dashboard.md automatically
4. **Metrics visualization** - Charts showing weekly trends
5. **Mobile notifications** - Push alerts for pending approvals
6. **Natural language queries** - "Show me all rejected payments this month"

## Resources

- **L01-L07**: Bronze tier foundation (skills, subagents, MCP, orchestration)
- **L08-L11**: Silver tier capabilities (watchers, HITL, cron, CEO Briefing)
- **L12**: Gold tier integration (error recovery, audit logging, documentation)
- **L13**: Assessment (test your knowledge before submitting)

## Next Steps

After completing Hackathon 0:

1. Share your repository on the Agent Factory Discord
2. Provide feedback on other students' projects
3. Consider contributing improvements back to the curriculum
4. Start planning Hackathon 1 (advanced topics)

---

**Remember:** The goal is not perfection. The goal is learning. Your first AI employee won't be flawless, but it will be yours -- and you'll understand every line of code, every design decision, every trade-off. That understanding is what matters.

Good luck! 🚀
