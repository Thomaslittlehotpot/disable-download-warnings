# 🛡️ 关闭下载安全警告

> 永久消除 Chrome 下载拦截提示和 Windows「无法验证发布者」弹窗——同时解放人类用户和 AI 自动化工具。

[**🇬🇧 English**](./README.md)

---

## 你遇到的问题

下载一个 .exe 或 .msi 文件，你面对的是：

1. **Chrome 拦截**：「此文件可能有害」——即使已经把 Safe Browsing 调到"不保护"
2. **Windows 盘问**：「打开文件 - 安全警告 — 无法验证发布者，你确定要运行吗？」——每次都弹

对 **AI 自动化**（Playwright、Puppeteer、MCP Chrome）来说，这些弹窗是灾难——AI 以为下载成功了，实际浏览器卡在确认对话框上。

---

## 解决办法

两个 .reg 文件。60 秒。永久生效。

| 文件 | 关闭什么 |
|------|---------|
| browser_policies.reg | Chrome/Chromium/Edge 安全浏览 + 下载限制 |
| attachment_fix.reg | Windows 附件管理器区域标记 +「无法验证发布者」弹窗 |

---

## 快速使用

### 1. 下载 .reg 文件

browser_policies.reg  — 关闭浏览器下载拦截（覆盖 3 种浏览器）
attachment_fix.reg    — 关闭 Windows「打开文件」安全警告

### 2. 双击每个文件 → 是 → 确定

### 3. 重启浏览器

搞定。不再有弹窗。

---

## 原理说明

### 浏览器拦截（3 条注册表路径）

| 注册表路径 | 影响的浏览器 |
|---|---|
| HKCU\\SOFTWARE\\Policies\\Google\\Chrome | Google Chrome |
| HKCU\\SOFTWARE\\Policies\\Chromium | Playwright Chromium、Puppeteer、MCP Chrome、所有 Chromium 内核自动化浏览器 |
| HKCU\\SOFTWARE\\Policies\\Microsoft\\Edge | Microsoft Edge |

每条路径设置：
- SafeBrowsingProtectionLevel = 0 → 关闭安全浏览
- DownloadRestrictions = 0 → 不限制下载
- SafeBrowsingEnabled = 0 / SmartScreenEnabled = 0 → 关闭 SmartScreen

### Windows 附件管理器

浏览器下载文件时，Windows 会偷偷给文件附加一个 Zone.Identifier 标记，注明「此文件来自 Internet」。运行文件时触发警告。

attachment_fix.reg 做了：
- SaveZoneInformation = 1 → 不再给新下载的文件打标记（治本）
- LowRiskFileTypes → 30+ 种文件类型视为安全（兼容已有标记的老文件）

---

## 给 AI 用户

如果你用 AI 工具操控浏览器（Playwright、MCP Chrome、browser-use 等）：

- 「带扳手图标」的浏览器是自动化 Chromium → 它读的是 Policies\\Chromium
- 可能会共享你真实 Chrome 的书签（能看到收藏夹），但 Cookie 可能不共享（原 Chrome 运行时会锁住 Cookie 数据库）
- 应用这些修复后，AI 不会再被下载确认弹窗卡住

---

## 系统要求

- Windows 10 或 Windows 11
- 管理员权限（写入 HKCU 注册表）
- Chrome / Chromium / Edge 浏览器

---

## 如何还原

删除注册表键值即可恢复默认：

```cmd
reg delete "HKCU\\SOFTWARE\\Policies\\Google\\Chrome" /f
reg delete "HKCU\\SOFTWARE\\Policies\\Chromium" /f
reg delete "HKCU\\SOFTWARE\\Policies\\Microsoft\\Edge" /f
reg delete "HKCU\\Software\\Microsoft\\Windows\\CurrentVersion\\Policies\\Attachments" /f
reg delete "HKCU\\Software\\Microsoft\\Windows\\CurrentVersion\\Policies\\Associations" /f
```

---

## 常见问题

**Q: 这样安全吗？**  
A: 它关闭了下载文件的安全警告。只在你信任下载来源时使用。开发者和高级用户经常这样设置。

**Q: 会影响 Windows Defender 吗？**  
A: 不会。Windows Defender 杀毒软件继续正常运行。只是关闭了打开文件时的「区域检查」。

**Q: AI 自动化 Chromium 能生效吗？**  
A: 能——Policies\\Chromium 路径覆盖 Playwright 捆绑的 Chromium、Puppeteer 的 Chromium、以及 MCP Chrome 实例。

**Q: 修复前下载的老文件还是弹窗？**  
A: LowRiskFileTypes 列表应该能覆盖。如果不能，重启电脑——有些缓存的区域信息需要重启才能清除。

**Q: 为什么我的 Chrome 在 C:\\Program Files\\Google\\Chrome\\Application 但 AI 搜不到？**  
A: 带空格的路径在某些 shell 环境中需要特殊处理。本项目的 SKILL.md 包含了绕过的办法。
