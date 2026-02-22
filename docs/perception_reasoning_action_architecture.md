# Perception → Reasoning → Action Architecture

The Personal AI Employee architecture is built on a three-layer system that mimics how intelligent systems process information and take action in the real world. Each layer has a distinct role in creating an autonomous digital         
employee.

## Layer 1: Perception (The "Watchers")

The Perception Layer serves as the AI's sensory system, continuously monitoring external sources for events that require attention. Since Claude Code can't constantly listen to external sources, lightweight Python Sentinel Scripts run in the background as "watchers":

- Gmail Watcher: Monitors Gmail API for urgent/important messages
- WhatsApp Watcher: Uses Playwright to monitor WhatsApp Web for specific keywords like "urgent", "invoice", "payment"   
- File Watcher: Monitors designated folders for new files to process

These watchers convert external events into markdown files placed in the /Needs_Action/ directory, effectively "waking up" the reasoning layer when something important happens. This solves the "lazy agent" problem by ensuring the AI is notified of important events rather than waiting passively for user input.

## Layer 2: Reasoning (Claude Code)

The Reasoning Layer is where intelligence and decision-making happen, powered by Claude Code. When a watcher deposits a file in /Needs_Action/, Claude Code processes it through:

1. Read: Analyzes the detected event and relevant context from the Obsidian vault
2. Think: Determines what needs to be done based on business rules and goals
3. Plan: Creates a structured plan with checkboxes in a Plan.md file
4. Write: Drafts responses or creates necessary files
5. Request Approval: For sensitive actions, creates approval request files in /Pending_Approval/

This layer serves as the cognitive engine, interpreting external inputs, applying business logic from files like Company_Handbook.md, and making decisions about the appropriate response.

### Layer 3: Action (MCP Servers)

The Action Layer executes the decisions made by the reasoning layer through Model Context Protocol (MCP) servers that interface with external services:

- Gmail MCP: Send, draft, and search emails
- Browser MCP: Navigate, click, and fill forms
- Calendar MCP: Create and update events
- Filesystem MCP: Read, write, and list files (built-in)

The action layer also implements human-in-the-loop controls for sensitive operations. Actions requiring approval are placed in /Pending_Approval/ where humans can review and either approve (move to /Approved/) or reject (move to /Rejected/) before execution.

### How They Work Together

The three layers create a closed-loop system:

1. Perception detects external events and creates action files
2. Reasoning interprets these events, applies business logic, and decides what to do
3. Action executes the decisions, with safety checks for sensitive operations

This architecture enables a Digital FTE that works 24/7, processes multiple input sources, makes intelligent decisions based on your business rules, and executes actions autonomously while maintaining human oversight for sensitive operations. The system is designed to be both autonomous and trustworthy, with clear audit trails and permission boundaries that define what can be auto-approved versus what requires human approval.

The architecture transforms AI from a reactive chatbot into a proactive business partner capable of managing complex workflows across multiple domains like email, social media, banking, and file management.