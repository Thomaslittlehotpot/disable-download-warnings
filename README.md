# 🛡️ Disable Download Warnings

> Permanently eliminate Chrome download safety prompts and Windows "Publisher could not be verified" dialogs — for users and AI agents alike.

[**🇨🇳 中文说明**](./README.zh-CN.md)

---

## The Problem

You download a .exe or .msi file. Two things happen:

1. **Chrome blocks it**: "*This file may be harmful*" — even with Safe Browsing set to "No protection"
2. **Windows interrogates you**: "*Open File - Security Warning — Publisher could not be verified. Are you sure?*" — every single time

For **AI automation** (Playwright, Puppeteer, MCP Chrome), these dialogs are catastrophic — the AI thinks the download succeeded but the browser is stuck on a confirmation prompt.

---

## The Fix

Two .reg files. 60 seconds. Permanent.

| File | What it disables |
|------|-----------------|
| browser_policies.reg | Chrome/Chromium/Edge Safe Browsing + download restrictions |
| attachment_fix.reg | Windows Attachment Manager zone markers + "Publisher" warnings |

---

## Quick Start

### 1. Download the .reg files

browser_policies.reg — disables browser download blocking (3 browsers)
attachment_fix.reg — disables Windows "Open File" security warnings

### 2. Double-click each file → Yes → OK

### 3. Restart your browser

That's it. No more prompts.


---

## What's Actually Happening

### Browser Blocking (3 registry paths)

| Registry Path | Browsers Affected |
|---|---|
| HKCU\SOFTWARE\Policies\Google\Chrome | Google Chrome |
| HKCU\SOFTWARE\Policies\Chromium | **Playwright Chromium**, Puppeteer, MCP Chrome, all Chromium-based automation tools |
| HKCU\SOFTWARE\Policies\Microsoft\Edge | Microsoft Edge |

Each gets:
- SafeBrowsingProtectionLevel = 0 → Protection off
- DownloadRestrictions = 0 → No restrictions
- SafeBrowsingEnabled = 0 / SmartScreenEnabled = 0 → SmartScreen off

### Windows Attachment Manager

When a browser downloads a file, Windows secretly attaches a **Zone.Identifier** marker saying "this came from the Internet." Launching the file triggers the warning.

attachment_fix.reg:
- SaveZoneInformation = 1 → **Stop attaching markers** to new downloads (root cause)
- LowRiskFileTypes → 30+ file extensions treated as safe (catches existing marked files)

---

## For AI Agent Users

If you use AI tools that control a browser (Playwright, MCP Chrome, browser-use, etc.):

- The "wrench icon" browser is the automation Chromium → **it reads Policies\Chromium**
- Your regular Chrome profile may be shared (bookmarks visible, but cookies locked if Chrome is running)
- After applying these fixes, AI agents will no longer get stuck on download confirmation dialogs

---

## Requirements

- Windows 10 or Windows 11
- Administrator access (registry writes to HKCU)
- Chrome / Chromium / Edge browser

---

## Reverting

Delete the registry keys to restore defaults:

```cmd
reg delete "HKCU\SOFTWARE\Policies\Google\Chrome" /f
reg delete "HKCU\SOFTWARE\Policies\Chromium" /f
reg delete "HKCU\SOFTWARE\Policies\Microsoft\Edge" /f
reg delete "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Attachments" /f
reg delete "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Associations" /f
```

---

## FAQ

**Q: Is this safe?**  
A: It disables safety warnings for downloaded files. Only do this if you trust what you download. This is equivalent to what developers and power users do on their machines.

**Q: Does it affect Windows Defender?**  
A: No. Windows Defender antivirus continues to run normally. This only disables the "zone check" on file open.

**Q: Will it work for AI automation Chromium?**  
A: Yes — the Policies\Chromium path covers Playwright's bundled Chromium, Puppeteer's Chromium, and MCP Chrome instances.

**Q: I still see warnings for files downloaded before the fix?**  
A: The LowRiskFileTypes list should cover them. If not, restart your computer — some cached zone info requires a reboot to clear.

---

## License

MIT — do whatever you want.
