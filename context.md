# Context: Real-Time Instruction System

## What Is This?

This repo is a **live instruction channel** between you and GitHub Copilot CLI.
During a long-running task, you cannot send new chat messages until it finishes.
This repo solves that: edit README.md on GitHub, and Copilot polls it mid-task.

---

## How To Use

1. **Copy this file** and paste it into Copilot at the start of any session.
2. **Start your task** - Copilot begins working AND launches the polling loop immediately.
3. **Send real-time instructions** by editing README.md at:
   https://github.com/asd125202/ai-instructions/edit/main/README.md
4. Copilot **stops only** after 3 minutes of no new changes — never self-terminates.

---

## Copilot Rules (AI must follow when this context is active)

1. **NEVER self-terminate.** Only the 3-minute idle timeout may end the session.

2. At the start of every task, launch this polling loop with `mode="async"`, `shellId: poll-git`:
   ```powershell
   $repoUrl = "https://github.com/asd125202/ai-instructions.git"
   $rawUrl  = "https://raw.githubusercontent.com/asd125202/ai-instructions/main/README.md"
   $baselineSha = (git ls-remote $repoUrl HEAD) -split "\s+" | Select-Object -First 1
   Write-Host "[POLL STARTED] Baseline SHA: $baselineSha"
   $noChange = 0; $maxNoChange = 60; $pollNum = 0
   while ($noChange -lt $maxNoChange) {
       Start-Sleep -Seconds 3
       $pollNum++
       $currentSha = (git ls-remote $repoUrl HEAD) -split "\s+" | Select-Object -First 1
       if ($currentSha -ne $baselineSha) {
           $content = (Invoke-WebRequest -Uri $rawUrl -UseBasicParsing).Content
           Write-Host "📥 NEW INSTRUCTION (poll #$pollNum | SHA: $currentSha)"
           Write-Host $content
           $baselineSha = $currentSha; $noChange = 0
       } else {
           $noChange++
           if ($noChange % 10 -eq 0) { Write-Host "[idle $noChange/$maxNoChange] Timeout in ~$(($maxNoChange-$noChange)*3)s." }
       }
   }
   Write-Host "⏹️  3 minutes no change. Polling stopped."
   ```

3. After launching, keep calling `read_powershell` (shellId: poll-git, delay: 10) in a loop — this keeps the turn alive and catches new instructions.

4. On change detected: announce `📥 New instruction: [summary]` and act on it immediately.

5. Stop only when the loop exits (60 no-change polls = 3 min).

---

## Why git ls-remote?

- Only fetches a **single SHA string** per poll (tiny payload, no content comparison issues)
- Full README is only fetched **when a change is confirmed**
- More reliable than HTTP content comparison (no encoding/whitespace bugs)

---

## Quick Reference

| Item | Value |
|---|---|
| Edit URL | https://github.com/asd125202/ai-instructions/edit/main/README.md |
| Repo | https://github.com/asd125202/ai-instructions |
| Poll method | `git ls-remote` (SHA only) |
| Poll interval | 3 seconds |
| Stop after | 3 min no change (60 polls) |
| Self-terminate? | ❌ NEVER |