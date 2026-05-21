---
name: disable-download-warnings
version: 1.0.0
description: >
  Permanently disable Chrome/Chromium/Edge download safety warnings and Windows
  Attachment Manager "Open File - Security Warning" prompts via registry Group Policy.
  Covers Google Chrome, Playwright Chromium, Puppeteer Chromium, MCP Chrome, and
  Microsoft Edge. Works on Windows 10 and 11. No third-party tools required.
tags:
  - windows
  - chrome
  - chromium
  - edge
  - download
  - security
  - registry
  - group-policy
  - automation
  - playwright
  - puppeteer
  - ai-agent
  - safe-browsing
  - smart-screen
  - attachment-manager
license: MIT
author: Thomaslittlehotpot
---
# Disable Download Warnings

## Overview

This skill permanently eliminates two independent layers of download-blocking prompts on Windows:

1. **Browser-level**: Chrome/Chromium/Edge download warnings (Safe Browsing)
2. **OS-level**: Windows Attachment Manager "Publisher could not be verified" dialog

Both are disabled via **Windows Registry Group Policy** — persistent across reboots, effective for both human users and AI browser automation.

---

## When to Use

Trigger when user reports:

- Chrome/Chromium/Edge shows "This file may be harmful" when downloading .exe/.msi/.bat
- Windows pops "Open File - Security Warning" when launching downloaded installers
- AI automation tools (Playwright, Puppeteer, MCP Chrome) get stuck on download dialogs
- User set Safe Browsing to "No protection" but still sees download prompts
- User wants permanent system-wide download prompt removal

---

## How It Works

### Mechanism 1: Browser Safe Browsing

Chrome blocks "dangerous" file types (.exe, .msi, .bat, .dll) even with Safe Browsing set to "No protection" in browser UI. Only **Windows Registry Group Policy** can override this — it has higher priority than browser settings.

Each browser brand reads from its own policy path:

| Browser | Policy Path |
|---------|-------------|
| Google Chrome | HKCU\SOFTWARE\Policies\Google\Chrome |
| Chromium / Playwright / Puppeteer / MCP Chrome / Electron | HKCU\SOFTWARE\Policies\Chromium |
| Microsoft Edge | HKCU\SOFTWARE\Policies\Microsoft\Edge |

> **Critical**: Playwright and Puppeteer use their own bundled Chromium (e.g. %LOCALAPPDATA%\ms-playwright\chromium-xxxx\chrome-win64\chrome.exe). These read from Policies\Chromium, NOT Policies\Google\Chrome. Always configure both.

### Mechanism 2: Windows Attachment Manager

Windows attaches a **Zone.Identifier** Alternate Data Stream (ADS / Mark of the Web) to downloaded files. Launching a file with this marker triggers the "Publisher could not be verified" dialog.

Fix:
- SaveZoneInformation = 1 → Stop attaching zone markers to new downloads (root cause)
- LowRiskFileTypes → 30+ extensions treated as safe (covers already-marked files)

---

## Implementation

### Step 1: Detect Environment

```
ver         # Confirm Windows 10/11
echo %USERPROFILE%
```

### Step 2: Scan for Browsers

Use PowerShell to find all Chromium-based browsers. Note: paths with spaces may fail in some shells — use short names (C:\PROGRA~1) as fallback.

### Step 3: Apply Browser Policies

Create browser_policies.reg and import with regedit /s:

```
[HKEY_CURRENT_USER\SOFTWARE\Policies\Google\Chrome]
"SafeBrowsingProtectionLevel"=dword:00000000
"DownloadRestrictions"=dword:00000000
"SafeBrowsingEnabled"=dword:00000000

[HKEY_CURRENT_USER\SOFTWARE\Policies\Chromium]
"SafeBrowsingEnabled"=dword:00000000
"SafeBrowsingProtectionLevel"=dword:00000000
"DownloadRestrictions"=dword:00000000

[HKEY_CURRENT_USER\SOFTWARE\Policies\Microsoft\Edge]
"ConfigureDownloadRestrictions"=dword:00000000
"SmartScreenEnabled"=dword:00000000
```

### Step 4: Disable Attachment Manager

Create attachment_fix.reg with SaveZoneInformation=1 and LowRiskFileTypes covering 30+ extensions. Import with regedit /s.

### Step 5: Verify

Query all 5 registry paths to confirm values are set.

### Step 6: Clean Up

Remove temporary .reg and .bat files.

---

## Robustness & Edge Cases

- Path detection failures: Use short 8.3 names (C:\PROGRA~1) when paths with spaces fail
- Registry write failures: Fall back to PowerShell Set-ItemProperty if reg add fails silently
- Old vs new downloads: SaveZoneInformation=1 only affects future downloads; LowRiskFileTypes catches existing files
- AI automation browsers: Always configure both Google\Chrome AND Chromium policy paths
- Headless shell: chrome-headless-shell.exe may not support policies; prefer headed Chromium
- Firefox: Not covered; use about:config to set browser.safebrowsing.downloads.enabled to false
- Enterprise: Add keys under HKLM for system-wide effect

---

## Troubleshooting

| Symptom | Cause | Fix |
|---------|-------|-----|
| Chrome not found by dir | Spaces in path fail | Short names or PowerShell Test-Path |
| Policies not taking effect | Browser was running | Close fully, reopen |
| reg add fails silently | Shell limitation | Use .reg + regedit /s |
| Old files still prompt | Zone.Identifier exists | LowRiskFileTypes covers; reboot |
| Playwright still blocks | Only Chrome policy set | Verify Policies\Chromium |
| Bookmarks visible, logged out | Cookies DB locked | Close Chrome before AI browser |
