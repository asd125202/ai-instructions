# Context: Real-Time Instruction System

## What Is This?

This repo is a **live instruction channel** between you and GitHub Copilot CLI.
During a long-running task, you cannot send new chat messages until it finishes.
This repo solves that: edit README.md on GitHub, and Copilot polls it mid-task.

---

## How To Use

1. **Copy this file** and paste it into Copilot at the start of any session.
2. **Start your task** - Copilot will begin working and poll README.md every ~30s.
3. **Send real-time instructions** by editing README.md at:
   https://github.com/asd125202/ai-instructions/edit/main/README.md
4. Copilot **stops polling** after ~5 minutes with no new changes.

---

## Copilot Rules (AI must follow when this context is active)

1. Poll this URL every ~30 seconds:
   https://raw.githubusercontent.com/asd125202/ai-instructions/main/README.md
2. Detect changes by comparing content to the last-seen version.
3. On new instruction detected: announce it briefly, then act on it.
4. Stop polling after 3 minutes (6 polls) with no change detected.
5. Polling must not block task completion - it runs alongside the task.

---

## Quick Reference

| Item | Value |
|---|---|
| Edit URL | https://github.com/asd125202/ai-instructions/edit/main/README.md |
| Poll URL | https://raw.githubusercontent.com/asd125202/ai-instructions/main/README.md |
| Poll interval | ~30 seconds |
| Stop after | 3 min no change |
