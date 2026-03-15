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

2. Immediately upon starting any task, launch a PowerShell async polling loop (shellId: poll-fast):
   ```powershell
   $pollUrl = "https://raw.githubusercontent.com/asd125202/ai-instructions/main/README.md"
   $baseline = (Invoke-WebRequest -Uri "$pollUrl?cb=$(Get-Random)" -UseBasicParsing).Content
   $noChange = 0; $maxNoChange = 60; $pollNum = 0  # 60 x 3s = 3 min
   while ($noChange -lt $maxNoChange) {
       Start-Sleep -Seconds 3
       $pollNum++
       $current = (Invoke-WebRequest -Uri "$pollUrl?cb=$(Get-Random)" -UseBasicParsing).Content
       if ($current -ne $baseline) {
           Write-Host "📥 NEW INSTRUCTION DETECTED (poll #$pollNum)"; Write-Host $current
           $baseline = $current; $noChange = 0
       } else {
           $noChange++
           if ($noChange % 10 -eq 0) { Write-Host "[idle $noChange/60] Timeout in ~$(($maxNoChange-$noChange)*3)s." }
       }
   }
   Write-Host "⏹️  3 minutes no change. Polling stopped."
   ```

3. Keep calling `read_powershell` (shellId: poll-fast) with delay:5 in a loop to stay alive and catch changes.

4. On change detected: announce "📥 New instruction: [summary]" and act on it immediately.

5. Stop only when the loop exits (60 consecutive no-change polls = 3 minutes).

---

## Quick Reference

| Item | Value |
|---|---|
| Edit URL | https://github.com/asd125202/ai-instructions/edit/main/README.md |
| Poll URL | https://raw.githubusercontent.com/asd125202/ai-instructions/main/README.md |
| Poll interval | **3 seconds** |
| Stop after | 3 min no change (60 polls) |
| Self-terminate? | ❌ NEVER |