# Context: Real-Time Instruction System

## What Is This?

This repo is a **live instruction channel** between you and GitHub Copilot CLI.
During a long-running task, you cannot send new chat messages until it finishes.
This repo solves that: edit README.md on GitHub, and Copilot polls it mid-task.

---

## How To Use

1. **Copy this file** and paste it into Copilot at the start of any session.
2. **Start your task** - Copilot will begin working AND start a polling loop immediately.
3. **Send real-time instructions** by editing README.md at:
   https://github.com/asd125202/ai-instructions/edit/main/README.md
4. Copilot **stops polling** only after 3 minutes of no new changes — never before.

---

## Copilot Rules (AI must follow when this context is active)

1. **NEVER self-terminate a task/response.** Only the 3-minute timeout may end the session.
2. Immediately upon starting any task, launch a polling loop (PowerShell async) that:
   - Fetches https://raw.githubusercontent.com/asd125202/ai-instructions/main/README.md every 30 seconds
   - Compares content to the last-seen version (baseline captured at session start)
   - On change detected: announce "📥 New instruction: [summary]" and act on it
   - Resets the 3-min idle timer on every change
3. Stop the loop ONLY when 6 consecutive polls (3 minutes) show no change.
4. While the loop runs, continue doing the actual task work in parallel.
5. When the loop finally exits, then (and only then) declare the task complete.

---

## Quick Reference

| Item | Value |
|---|---|
| Edit URL | https://github.com/asd125202/ai-instructions/edit/main/README.md |
| Poll URL | https://raw.githubusercontent.com/asd125202/ai-instructions/main/README.md |
| Poll interval | ~30 seconds |
| Stop after | 3 min no change (6 polls) |
| Self-terminate? | ❌ NEVER |