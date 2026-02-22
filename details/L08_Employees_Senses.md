# L08: Employee's Senses

In Lesson 7, you assembled a complete email assistant -- skills for drafting, subagents for reasoning, Gmail MCP for action. But your employee has a problem: it only works when you type a command. It cannot notice that three urgent emails arrived while you were in a meeting. It cannot detect that a client dropped a contract PDF in your shared folder. It sits idle until you remember to ask.

This is the lazy agent problem. Your employee has expertise (skills), judgment (subagents), and access (MCP), but no senses. It is a brilliant worker sitting in a dark room with no windows.

Watchers give your employee eyes and ears. They are lightweight Python scripts that run in the background, monitoring data sources for changes. When something happens -- a new file appears, an email arrives -- the watcher creates an action file that wakes your employee up. This lesson builds your first watcher: a filesystem monitor that detects files dropped into a folder and deposits action files into /Needs_Action/ for Claude Code to process.

## The Perception Gap

Your Bronze tier employee has three layers from the L00 architecture, but one is missing:

LayerPurposeBronze Status
PerceptionDetect changes in the worldMissing -- no way to notice events
ReasoningAnalyze and decide (Claude Code + skills + subagents)Working
ActionExecute decisions (Gmail MCP + file operations)Working

Without Perception, the flow breaks at the start:

Event happens → ??? → Reasoning → Action
 ^
 Nobody told Claude Code

Watchers fill that gap. They are the bridge between "something happened in the world" and "Claude Code knows about it":

Event happens → Watcher detects → Action file in /Needs_Action/ → Claude Code reasons → Action

The key insight: Watchers do not process anything. They only detect and deposit. A watcher that spots a new file does not read the file, analyze it, or decide what to do with it. It writes a brief action file describing what it found, and Claude Code handles the rest.

## The Watcher Pattern

Every watcher in the L00 specification follows the same three-method pattern:

MethodPurposeOutput
check_for_updates()Poll or listen for changes at the sourceList of new items detected
create_action_file(item)Convert detected item into a markdown fileFile written to /Needs_Action/
run()Infinite loop: check, process, sleepContinuous background operation

This pattern works for any data source:

Watcher TypeWhat It MonitorsHow It Checks
FilesystemDrop folder for new filesEvent-based (instant detection)
GmailInbox for new messagesPoll-based (check every N seconds)
WhatsAppChat for keywordsPoll-based via browser automation
CalendarUpcoming eventsPoll-based (check every N minutes)

You will build the Filesystem Watcher in this lesson. The Gmail Watcher follows the same pattern but requires Gmail API credentials -- that is a Hackathon deliverable after you have mastered the pattern here.

## Building a Filesystem Watcher

The filesystem watcher monitors a "drop folder" on your computer. When you (or any other program) saves a file there, the watcher detects it and creates an action file in /Needs_Action/.

### Step 1: Install watchdog

The watchdog library provides cross-platform filesystem event monitoring. It handles the operating system details so your code focuses on business logic.

pip install watchdog

Output:

Successfully installed watchdog-4.0.0

If you use uv for package management:

uv pip install watchdog

Output:

Resolved 1 package in 0.5s
Installed 1 package in 0.3s
 + watchdog==4.0.0

Note: watchdog may not detect events reliably on network-mounted filesystems or Windows Subsystem for Linux (WSL). If you are on WSL, test with a local directory first.

### Step 2: Create the Drop Folder and Vault Directories

Before writing the watcher, create the directories it needs:

mkdir -p ~/employee-inbox
mkdir -p ~/projects/ai-vault/Needs_Action

Output:

(no output means success)

Verify they exist:

ls -d ~/employee-inbox ~/projects/ai-vault/Needs_Action

Output:

/Users/yourname/employee-inbox
/Users/yourname/projects/ai-vault/Needs_Action

### Step 3: Write the Watcher Script

Create a file called file_watcher.py in your project directory:

```python
#!/usr/bin/env python3
"""Filesystem Watcher - monitors drop folder for new files."""

import time
import datetime
from pathlib import Path
from watchdog.observers import Observer
from watchdog.events import FileSystemEventHandler

# Configuration
DROP_FOLDER = Path.home() / "employee-inbox"
VAULT_NEEDS_ACTION = Path.home() / "projects" / "ai-vault" / "Needs_Action"

class InboxHandler(FileSystemEventHandler):
 """Handles file creation events in the drop folder."""

 def on_created(self, event):
 if event.is_directory:
 return

 src = Path(event.src_path)
 now = datetime.datetime.now()
 timestamp = now.strftime("%Y-%m-%d_%H-%M-%S")

 # Create action file with metadata frontmatter
 action_filename = f"FILE_{src.stem}_{timestamp}.md"
 action_file = VAULT_NEEDS_ACTION / action_filename

 try:
 file_size = src.stat().st_size
 except OSError:
 file_size = 0

 action_file.write_text(
 f"---\n"
 f"type: file_received\n"
 f"source: {src}\n"
 f"detected: {now.isoformat()}\n"
 f"status: pending\n"
 f"---\n\n"
 f"# New File Received\n\n"
 f"- **File**: {src.name}\n"
 f"- **Size**: {file_size} bytes\n"
 f"- **Location**: {src}\n"
 f"- **Detected**: {now.strftime('%Y-%m-%d %H:%M:%S')}\n\n"
 f"## Action Required\n\n"
 f"Process this file according to vault rules.\n"
 )
 print(f"[WATCHER] Detected: {src.name} -> Created: {action_filename}")

if __name__ == "__main__":
 # Ensure directories exist
 DROP_FOLDER.mkdir(parents=True, exist_ok=True)
 VAULT_NEEDS_ACTION.mkdir(parents=True, exist_ok=True)

 # Set up the observer
 observer = Observer()
 observer.schedule(InboxHandler(), str(DROP_FOLDER), recursive=False)
 observer.start()

 print(f"[WATCHER] Monitoring: {DROP_FOLDER}")
 print(f"[WATCHER] Action files go to: {VAULT_NEEDS_ACTION}")
 print(f"[WATCHER] Press Ctrl+C to stop")

 try:
 while True:
 time.sleep(1)
 except KeyboardInterrupt:
 print("\n[WATCHER] Stopping...")
 observer.stop()
 observer.join()
 print("[WATCHER] Stopped.")
```

### Step 4: Run the Watcher

Open a terminal and start the watcher:

python file_watcher.py

Output:

[WATCHER] Monitoring: /Users/yourname/employee-inbox
[WATCHER] Action files go to: /Users/yourname/projects/ai-vault/Needs_Action
[WATCHER] Press Ctrl+C to stop

The watcher is now running. Leave this terminal open.

Safety Note: Watchers run as background processes with filesystem access. Only monitor directories you control. Never point a watcher at system directories (/, /etc, C:\Windows). The watcher in this lesson writes files but does not delete or modify anything -- it is read-only on the source and write-only on the action folder.

### Step 5: Test It

Open a second terminal and create a test file in the drop folder:

echo "Q4 revenue report draft" > ~/employee-inbox/q4-report.txt

Output in the watcher terminal:

[WATCHER] Detected: q4-report.txt -> Created: FILE_q4-report_2026-02-19_10-30-45.md

### Step 6: Verify the Action File

Check that the action file was created:

ls ~/projects/ai-vault/Needs_Action/

Output:

FILE_q4-report_2026-02-19_10-30-45.md

Read the action file contents:

cat ~/projects/ai-vault/Needs_Action/FILE_q4-report_2026-02-19_10-30-45.md

Output:

---
type: file_received
source: /Users/yourname/employee-inbox/q4-report.txt
detected: 2026-02-19T10:30:45.123456
status: pending
---

# New File Received

- **File**: q4-report.txt
- **Size**: 25 bytes
- **Location**: /Users/yourname/employee-inbox/q4-report.txt
- **Detected**: 2026-02-19 10:30:45

## Action Required

Process this file according to vault rules.

Your watcher detected the file, extracted metadata, and deposited a structured action file. Claude Code can now read this file and decide what to do with it.

### Step 7: Stop the Watcher

Go back to the watcher terminal and press Ctrl+C:

Output:

[WATCHER] Stopping...
[WATCHER] Stopped.

## How This Connects to Your Employee

Here is the complete pipeline from file drop to employee action:

YOU (or any program)
 │
 ▼ Save file to ~/employee-inbox/
┌──────────────────┐
│ file_watcher.py │ ← Perception Layer (this lesson)
│ Detects new file│
└────────┬─────────┘
 │ Creates action file
 ▼
┌──────────────────┐
│ /Needs_Action/ │ ← File-based communication
│ FILE_*.md │
└────────┬─────────┘
 │ Claude Code reads
 ▼
┌──────────────────┐
│ Claude Code │ ← Reasoning Layer (L05-L07)
│ Skills + │
│ Subagents │
└────────┬─────────┘
 │ Takes action
 ▼
┌──────────────────┐
│ Gmail MCP │ ← Action Layer (L06)
│ File operations │
│ Other MCP tools │
└──────────────────┘

The watcher is deliberately simple. It does not decide what to do with the file -- that is Claude Code's job. This separation means you can add new watchers (Gmail, calendar, Slack) without changing your reasoning layer. Each watcher follows the same pattern: detect, create action file, deposit in /Needs_Action/.

## Gmail Watcher Architecture

The Gmail Watcher follows the identical pattern but polls the Gmail API instead of listening for filesystem events:

AspectFilesystem WatcherGmail Watcher
Detection methodEvent-based (instant)Poll-based (every 60 seconds)
LibrarywatchdogGmail API via google-api-python-client
What it monitorsDrop folder for new filesInbox for new messages
Action file contentFile metadata (name, size, path)Email metadata (sender, subject, priority)
Complexity~30 lines~80 lines (API auth + pagination)

You will build the Gmail Watcher in Hackathon 0 (L14). You already have Gmail MCP configured from L06 and the inbox-triager from L05. The watcher connects them by depositing action files that trigger your existing reasoning pipeline. The pattern you learned in this lesson -- detect, create action file, deposit -- is exactly the same.

## Troubleshooting

ProblemCauseFix
ModuleNotFoundError: No module named 'watchdog'Library not installedRun pip install watchdog
Watcher runs but no detectionFile created before watcher startedWatcher only detects files created AFTER it starts running
PermissionError on action file/Needs_Action/ directory missingRun mkdir -p ~/projects/ai-vault/Needs_Action
Watcher detects file twiceSome editors save temp files firstAdd a filter: if src.name.startswith('.') or src.name.endswith('.tmp'): return
Watcher stops when terminal closesScript runs in foregroundLesson 10 covers running watchers as persistent services with PM2

## Try With AI

Setup: Have your file_watcher.py running in one terminal and Claude Code open in another.

Prompt 1: Test the Full Pipeline

I have a file watcher running that deposits action files in ~/projects/ai-vault/Needs_Action/.
I just dropped a file called "invoice-acme-jan.pdf" into ~/employee-inbox/.

Check ~/projects/ai-vault/Needs_Action/ for the action file it created.
Read the action file and tell me:
1. What file was detected?
2. When was it detected?
3. What action would you recommend based on the file name?

What you're learning: This tests the end-to-end pipeline. Your watcher (Perception) creates the action file, and Claude Code (Reasoning) reads it and decides what to do. Notice how Claude Code recommends actions based on the filename metadata -- this is the reasoning layer working with the perception layer's output.

Prompt 2: Design a Filtering Watcher

My current file watcher triggers on ALL new files, including temporary files
from my editor (like .swp files and files starting with ._).

Help me modify the InboxHandler.on_created() method to:
1. Ignore files starting with "." or "_"
2. Ignore files ending with .tmp, .swp, or .bak
3. Only trigger on specific extensions: .pdf, .csv, .txt, .md
4. Log ignored files separately from processed files

Show me the updated on_created() method.

What you're learning: This is iterative refinement of your watcher. You built the basic version; now you are refining it with AI assistance to handle real-world edge cases. The AI suggests filtering patterns you might not have considered, and you decide which filters match your actual workflow.
