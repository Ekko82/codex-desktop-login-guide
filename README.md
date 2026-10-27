# Codex desktop login loop on Windows: account unavailable / missing `chatgpt_account_id`

English | [简体中文](README.zh-CN.md)

Documented on October 27, 2026. Tested with Windows desktop version `26.1021.3705.0`.

This community guide records one observed recovery: authorize the same account through the official device login flow in a **fresh Edge InPrivate session**, then let the desktop app use the new credentials. The CLI is only used to complete login; you can continue working in the desktop app afterward.

## When to try this

The symptoms in this case included:

- ChatGPT worked in the browser, but Codex desktop repeatedly requested login or could not load the account.
- The desktop reported unavailable account authentication from its local `app-server`.
- Reinstalling and signing in again did not resolve the problem. The CLI could respond while the desktop still failed.
- The issued credentials lacked `chatgpt_account_id`, `chatgpt_user_id`, and `chatgpt_plan_type`.

A separate diagnostic attempt with a workspace restriction produced this error:

```text
Sign-in could not be completed
Error code: workspace_restriction
Login is restricted to a specific workspace, but the token did not include an chatgpt_account_id claim.
```

That error was part of this investigation. You do not need to configure a workspace restriction or inspect tokens to follow this guide.

The checklist below is for a personal account on Windows, with Codex desktop and the CLI sharing their default credential storage. If you use a custom `CODEX_HOME`, credential store, or organization login policy, first confirm which login store each application uses.

This is a workaround tested in one case, not an official fix for every login failure. If you see explicit connection timeouts or certificate errors, investigate those errors separately.

## Manual checklist

### 1. Save your work and fully quit Codex desktop

- [ ] Save any work in progress.
- [ ] Choose **Quit** or **Exit** from the application menu or system tray.

Closing the window with `X` may leave a background process running with the old login state.

### 2. Enable device code login in ChatGPT on the web

Sign in to the account you want to use, then open:

```text
Profile menu → Settings → Security
```

- [ ] Find the setting for device code login and enable it.

The wording may vary. This is a ChatGPT account setting in the browser. Organization accounts may require an administrator to allow this feature. See the [official authentication documentation](https://learn.chatgpt.com/docs/auth#preferred-device-code-authentication-beta).

### 3. Start a fresh Edge InPrivate session

- [ ] Save anything you need from existing InPrivate windows, then close **all** of them.
- [ ] Press `Ctrl + Shift + N` in Edge to open a new InPrivate window.

Multiple InPrivate windows can share the same browser session. Opening another window while an older one remains open may keep the previous session.

### 4. Start device login in a normal PowerShell window

Use the same Windows user account that normally runs Codex desktop. Administrator privileges are not required. Open PowerShell and run:

```powershell
codex login --device-auth
```

- [ ] Run only one login attempt at a time and leave this PowerShell window open.

If PowerShell cannot find `codex`, follow the [official CLI installation instructions](https://learn.chatgpt.com/docs/codex/cli), then run the command again. The CLI provides the login command; you can keep using the desktop app for your work.

Completing login updates the credentials Codex uses. Avoid starting other login attempts in the desktop app, another terminal, or your IDE while this one is pending.

### 5. Find the one-time code in PowerShell

The code appears below this prompt in the terminal:

```text
Enter this one-time code
```

- [ ] Use the code from the current attempt.
- [ ] Keep the code private and do not reuse a code from an earlier attempt.

For this manual procedure, the code appears directly in PowerShell. You do not need to find a separate log file. If the code expires, press `Ctrl + C` in that terminal and repeat step 4 to generate a new one.

### 6. Complete authorization in the new InPrivate window

In the InPrivate window from step 3, manually open the official URL shown in the terminal. It is normally:

[https://auth.openai.com/codex/device](https://auth.openai.com/codex/device)

- [ ] Sign in to your intended account again.
- [ ] Enter the current one-time code and complete authorization.
- [ ] Stay in this new InPrivate session throughout the process.

If the page asks you to enable device code login, complete step 2 first. Then stop the old terminal login attempt and repeat step 4 for a new code.

### 7. Wait for PowerShell to confirm success

Wait until the terminal displays:

```text
Successfully logged in
```

- [ ] Reopen Codex desktop after the terminal confirms success.

A completion page in the browser alone does not confirm that the CLI has finished saving the login.

### 8. Verify the desktop app and login persistence

- [ ] Check that the account loads without an authentication error.
- [ ] Send a short message in the desktop app and confirm that it receives a response.
- [ ] Fully quit and reopen the desktop app. Confirm that it remains signed in and can still respond.

## What was verified in this case

The tested desktop version was `26.1021.3705.0` on Windows. Reauthorizing in the ordinary browser session still produced credentials without `chatgpt_account_id`, `chatgpt_user_id`, and `chatgpt_plan_type`. Authorizing the same account in a fresh Edge InPrivate session produced official credentials containing those fields.

During the investigation, the new credentials were first tested in an isolated `CODEX_HOME`, then copied into the credential store used by the desktop app. The manual checklist above adapts that method for the default shared credential store; this exact simplified sequence was not separately tested from beginning to end.

After the credential replacement, the desktop loaded the account and replied to messages. It also remained signed in after a full quit and reopen. The desktop program, JWT contents, and `.env` were not modified.

This result supports trying authorization in a fresh browser session. It does not establish that browser cookies caused the missing claims, or that the method will resolve every similar error.

## If it still fails

Record the step that failed, the error text, and the desktop and CLI versions. If the CLI reports success but desktop still cannot load the account, confirm that both use the same credential store before repeating login.

Do not fill in account IDs by hand, edit tokens, or clear the entire `.codex` directory to suppress the error. Keep `auth.json`, access tokens, refresh tokens, and one-time codes out of screenshots, issue comments, and uploaded logs. Treat `auth.json` like a password. See the [official credential storage documentation](https://learn.chatgpt.com/docs/auth#credential-storage).

## Related reports

- [openai/codex#48571](https://github.com/openai/codex/issues/48571): a Windows report with the same three missing account fields.
- [openai/codex#52112](https://github.com/openai/codex/issues/52112): a similar report on macOS. The recovery documented here was tested on Windows only.
