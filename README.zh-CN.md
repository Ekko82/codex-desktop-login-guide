# Codex 桌面版反复登录、账户无法加载：Windows 手动修复指南

[English](README.md) | 简体中文

整理日期：2026 年 10 月 27 日。验证桌面版本：`26.1021.3705.0`。

这是一份社区指南，记录了一次实际观察到的恢复：**在全新的 Edge InPrivate 会话中，通过官方设备代码登录重新授权，再让桌面版使用新的登录凭据。** 命令行只用于完成授权，恢复后继续使用桌面版。

## 适用情况

本次案例出现了以下症状：

- 网页 ChatGPT 能正常使用，但 Codex 桌面反复要求登录或无法加载账户。
- 桌面提示“账户身份验证不可用”，无法从本地 `app-server` 获取可用的账户身份验证信息。
- 已经重新登录或重装，问题仍然存在；CLI 能正常回复，桌面却无法加载账户。
- 官方签发的凭据缺少 `chatgpt_account_id`、`chatgpt_user_id` 和 `chatgpt_plan_type`。

另外一次带工作区限制的诊断登录出现了以下报错：

```text
Sign-in could not be completed
Error code: workspace_restriction
Login is restricted to a specific workspace, but the token did not include an chatgpt_account_id claim.
```

这条报错来自本次排查过程。执行下面的清单不需要设置工作区限制，也不需要自行查看令牌内容。

下面的步骤面向 Windows 个人账户、桌面与 CLI 共用默认登录存储的环境。如果设置过自定义 `CODEX_HOME`、凭据存储或组织登录策略，先确认桌面与 CLI 使用的是同一套登录存储。

这是在一个案例中验证有效的恢复方法，不是所有登录故障的官方修复方案。如果出现明确的网络超时或证书错误，需要针对这些错误另行排查。

## 手动操作清单

### 1. 保存工作，完全退出 Codex 桌面

- [ ] 保存正在进行的工作。
- [ ] 从应用菜单或系统托盘选择“退出”。

只点击窗口右上角的 `×`，可能仍有后台进程保留旧的登录状态。

### 2. 在网页 ChatGPT 中开启设备代码登录

登录准备使用的账户，进入：

```text
头像菜单 → 设置 → 安全
```

- [ ] 找到包含“设备代码登录”或“Device code”的开关并开启。

不同版本的名称可能略有差异。这是网页账户设置，不是 Windows 设置。组织账户可能需要管理员允许这项功能，见[官方认证说明](https://learn.chatgpt.com/docs/auth#preferred-device-code-authentication-beta)。

### 3. 打开真正全新的隐私窗口

- [ ] 保存已有隐私窗口中的内容，关闭所有 Edge InPrivate 窗口。
- [ ] 在 Edge 中按 `Ctrl + Shift + N`，新开一个 InPrivate 窗口。

已有的多个 InPrivate 窗口可能共享同一登录会话。仅再开一个窗口，不一定能避开旧会话。

### 4. 在普通 PowerShell 中发起登录

使用平时运行桌面的同一个 Windows 用户，不需要管理员权限。打开 PowerShell，执行：

```powershell
codex login --device-auth
```

- [ ] 只运行一个登录流程，并保持这个 PowerShell 窗口打开。

如果提示找不到 `codex`，先按[官方 CLI 安装说明](https://learn.chatgpt.com/docs/codex/cli)准备登录工具，再执行上面的命令。安装这个工具不意味着以后必须改用命令行工作。

完成登录会更新 Codex 使用的凭据。不要同时在桌面、另一个终端或 IDE 中反复发起登录，以免混淆授权流程。

### 5. 从 PowerShell 窗口中找到一次性代码

代码就在这行提示的下方：

```text
Enter this one-time code
```

- [ ] 使用当前这次生成的代码。
- [ ] 不沿用上一轮的旧代码，也不把代码发给别人。

手动操作时，代码直接显示在 PowerShell 窗口里，不需要另外寻找日志文件。如果代码过期，在旧登录窗口按 `Ctrl + C` 结束流程，再执行第 4 步生成新代码。

### 6. 在新开的 InPrivate 窗口中完成授权

在第 3 步打开的隐私窗口中，手动打开终端显示的官方地址，通常是：

[https://auth.openai.com/codex/device](https://auth.openai.com/codex/device)

- [ ] 重新登录自己的正确账户。
- [ ] 输入第 5 步的一次性代码，完成授权。
- [ ] 全程使用这个新的 InPrivate 窗口，不切回普通浏览器窗口继续操作。

如果此时才提示“需要开启设备代码登录”，先按第 2 步开启开关，再结束旧登录流程，重新执行第 4 步生成新代码。

### 7. 等待 PowerShell 确认登录成功

终端中应看到：

```text
Successfully logged in
```

- [ ] 确认终端出现成功提示后，再正常打开 Codex 桌面。

仅看到浏览器里的完成页面还不够，需要等终端确认。

### 8. 检查是否真正恢复

- [ ] 账户页面能正常加载，没有身份验证错误。
- [ ] 在桌面发送一句“你好”，能够收到回复。
- [ ] 完全退出桌面，再打开一次，仍保持登录并能正常聊天。

完成这三项后，才算验证了桌面可用及登录状态能够保留。

## 本次案例验证到了什么

验证环境为 Windows，桌面版本为 `26.1021.3705.0`。

普通浏览器会话下重新授权后，凭据仍缺少 `chatgpt_account_id`、`chatgpt_user_id` 和 `chatgpt_plan_type`。在全新的 Edge InPrivate 会话中登录同一账户后，官方签发的新凭据包含了这些字段。

实际排查先在隔离的 `CODEX_HOME` 中验证新凭据，再复制到桌面使用的登录存储。上面的清单将这一方法整理为默认共享登录存储下的手动步骤，没有单独从头到尾验证这套简化清单。

替换后，桌面成功加载账户、正常回复消息，完全退出再打开也无需再次登录。没有修改桌面程序、JWT 内容或 `.env`。

这次结果支持尝试避开旧浏览器登录会话重新授权，但不能据此确定凭据字段缺失由浏览器 Cookie 引起，也不能保证适用于所有类似错误。

## 如果仍然失败

记录出错的步骤、完整错误文字，以及桌面和 CLI 版本，继续针对具体错误排查。如果 CLI 已提示成功，桌面仍无法加载账户，先确认两者使用同一套登录存储。

不要通过手填账户 ID、修改令牌或清空整个 `.codex` 文件夹来强行消除提示。反馈问题时，不要上传 `auth.json`、访问令牌、刷新令牌或一次性代码。`auth.json` 包含登录凭据，应当像密码一样保管。官方对登录缓存与凭据存储的说明见[认证文档](https://learn.chatgpt.com/docs/auth#credential-storage)。

## 相关报告

- [openai/codex#48571](https://github.com/openai/codex/issues/48571)：Windows 上缺少相同三个账户身份字段的报告。
- [openai/codex#52112](https://github.com/openai/codex/issues/52112)：macOS 上的类似报告。本指南的恢复方法只在 Windows 上验证过。
