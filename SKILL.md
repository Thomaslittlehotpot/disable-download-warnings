---
name: disable-download-warnings
description: >
  Disable Chrome/Chromium download safety warnings and Windows Attachment Manager
  "Open File - Security Warning" prompts. Covers Google Chrome, Chromium (Playwright,
  Puppeteer, MCP Chrome), and Microsoft Edge. Works via Windows Registry Group Policy
  for permanent system-level effect. Use when AI agents get blocked by download
  interception dialogs, or when users are tired of confirming every downloaded .exe/.msi.
---

# Disable Download Warnings

## Overview

This skill eliminates two layers of download-blocking prompts on Windows:

1. **Browser-level**: Chrome/Chromium/Edge "This file may be harmful" / Safe Browsing download warnings
2. **OS-level**: Windows Attachment Manager "Open File - Security Warning — Publisher could not be verified"

Both are disabled via Windows Registry Group Policy — no third-party tools needed, and settings persist across reboots.

---

## When to Use

Trigger this skill when the user experiences any of:

- Chrome shows "This file may be harmful" when downloading .exe/.msi files
- Windows pops up "Open File - Security Warning" when launching downloaded installers
- AI automation tools (Playwright, Puppeteer, MCP Chrome) get stuck on download interception dialogs
- User wants to permanently disable download safety prompts system-wide
- User has already set Safe Browsing to "No protection" but still sees download prompts

---

## How It Works

There are **two independent blocking mechanisms** on Windows:

### Mechanism 1: Browser Safe Browsing / Download Restrictions

Chrome and Chromium-based browsers block or warn about "dangerous" file types. Even with Safe Browsing set to "No protection" in browser settings, Chrome still intercepts certain downloads. The only way to fully disable this is via **Windows Registry Group Policy**, which has higher priority than browser UI settings.

### Mechanism 2: Windows Attachment Manager

When a browser downloads a file, Windows attaches a **Zone.Identifier** Alternate Data Stream (ADS) marking it as "from the Internet." When you open the file, Windows checks this marker and shows the "Publisher could not be verified" dialog. Disabling this requires setting SaveZoneInformation=1 to stop the marker from being attached.

---

## Implementation

### Step 1: Detect the environment

Run ver or echo %OS% to confirm Windows. Check user profile path with echo %USERPROFILE%.

Search for installed Chromium-based browsers:
```powershell
dir /s /b "%LOCALAPPDATA%\ms-playwright" 2>nul
dir /b "C:\Program Files\Google\Chrome\Application\chrome.exe" 2>nul
dir /b "C:\Program Files (x86)\Microsoft\Edge\Application\msedge.exe" 2>nul
```

> **Note**: C:\Program Files paths with spaces may fail with cmd /c. Use PowerShell Test-Path or short names like C:\PROGRA~1 instead.

### Step 2: Create browser registry policies

Create a .reg file and import it silently:

```reg
Windows Registry Editor Version 5.00

[HKEY_CURRENT_USER\SOFTWARE\Policies\Google\Chrome]
"SafeBrowsingProtectionLevel"=dword:00000000
"DownloadRestrictions"=dword:00000000
"SafeBrowsingEnabled"=dword:00000000

[HKEY_CURRENT_USER\SOFTWARE\Policies\Chromium]
"SafeBrowsingEnabled"=dword:00000000
"SafeBrowsingProtectionLevel"=dword:00000000
"DownloadRestrictions"=dword:00000000
"AllowDeletingBrowserHistory"=dword:00000001

[HKEY_CURRENT_USER\SOFTWARE\Policies\Microsoft\Edge]
"ConfigureDownloadRestrictions"=dword:00000000
"SmartScreenEnabled"=dword:00000000
"SmartScreenPuaEnabled"=dword:00000000
"PreventSmartScreenPromptOverride"=dword:00000000
```

Import with:
```cmd
regedit /s browser_policies.reg
```

**What each path covers:**

| Registry Path | Browsers Covered |
|---|---|
| Policies\Google\Chrome | Google Chrome (system-installed) |
| Policies\Chromium | Playwright Chromium, Puppeteer Chromium, MCP Chrome, all Chromium-based automation browsers |
| Policies\Microsoft\Edge | Microsoft Edge |

**Key value meanings:**
- SafeBrowsingProtectionLevel=0 → No protection (0=disabled, 1=standard, 2=enhanced)
- DownloadRestrictions=0 → No download restrictions
- SafeBrowsingEnabled=0 → Safe Browsing completely off
- SmartScreenEnabled=0 → Microsoft Defender SmartScreen off

### Step 3: Disable Windows Attachment Manager

Create another .reg file:

```reg
Windows Registry Editor Version 5.00

[HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Policies\Attachments]
"SaveZoneInformation"=dword:00000001

[HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Policies\Associations]
"LowRiskFileTypes"=".exe;.msi;.bat;.cmd;.ps1;.vbs;.reg;.msu;.msix;.appx;.jar;.com;.scr;.msc;.cpl;.hta;.chm;.tmp;.zip;.rar;.7z;.iso;.dmg;.pkg;.apk;.py;.js;.vbe;.wsf;.wsh"
```

Import with:
```cmd
regedit /s attachment_fix.reg
```

**What this does:**
- SaveZoneInformation=1 → Stop attaching "from Internet" zone markers to new downloads (root cause fix)
- LowRiskFileTypes → List of 30+ file extensions treated as safe; covers existing files that already have zone markers

### Step 4: Verify

Query all registry paths to confirm values are set:

```cmd
reg query "HKCU\SOFTWARE\Policies\Google\Chrome"
reg query "HKCU\SOFTWARE\Policies\Chromium"
reg query "HKCU\SOFTWARE\Policies\Microsoft\Edge"
reg query "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Attachments"
reg query "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Associations"
```

> **Important**: When using remote/local system tools, prefer PowerShell Set-ItemProperty over reg add for registry writes, as reg add may silently fail in certain execution environments.

### Step 5: Clean up

Remove temporary .reg and .bat files created during the process.

---

## Caveats & Notes

1. **Existing vs. new downloads**: SaveZoneInformation=1 only affects files downloaded AFTER the setting is applied. Files already on disk keep their Zone.Identifier markers — that's why LowRiskFileTypes is also set as a safety net.

2. **AI automation browsers**: Playwright and Puppeteer use their own bundled Chromium (e.g., %LOCALAPPDATA%\ms-playwright\chromium-xxxx\chrome-win64\chrome.exe). These read HKCU\SOFTWARE\Policies\Chromium, not Google\Chrome. Make sure both are configured.

3. **Bookmarks vs. cookies in AI browsers**: When AI tools launch Chrome pointing to your real profile (--user-data-dir), they can read Bookmarks (plain JSON file) but NOT Cookies (SQLite database locked by the running Chrome instance). This is why AI browsers show your bookmarks but require re-login. Close your regular Chrome before using AI browsers if you want cookie sharing.

4. **Headless shell**: Playwright's headless Chromium (chrome-headless-shell.exe) may not implement full policy support. For headless downloads, consider using the headed Chromium instead.

5. **Windows 11**: All registry paths are identical on Windows 11. No changes needed.

---

## Troubleshooting

| Symptom | Cause | Fix |
|---------|-------|-----|
| reg add fails silently | Shell execution environment limitation | Use PowerShell Set-ItemProperty or .reg file import |
| Policies not taking effect | Chrome/Chromium was already running | Restart the browser completely (check taskbar tray) |
| Still seeing prompts for old files | Zone.Identifier already attached | LowRiskFileTypes covers them; or manually delete the ADS with streams.exe -d file.exe |
| Playwright Chromium still blocks | Chromium policy path missing | Verify HKCU\SOFTWARE\Policies\Chromium exists with all values |
| Short 8.3 names don't resolve | NTFS 8.3 name generation disabled | Use PowerShell Get-ChildItem instead of dir /x |
