# MCP Setup Guide

Step-by-step instructions for connecting Gmail and Google Calendar to Claude Code via claude.ai's built-in Google integration.

---

## How It Works

Inbox Zero uses claude.ai's built-in Google integration, which exposes Gmail and Google Calendar as MCP tools inside Claude Code. No API keys, no OAuth app setup — just connect your Google account once.

---

## Connecting Gmail + Google Calendar

1. Open **[claude.ai](https://claude.ai)** in your browser
2. Go to **Settings** (bottom-left avatar → Settings)
3. Click **Integrations** in the left sidebar
4. Find **Google** and click **Connect**
5. Sign in with the Google account whose Gmail you want to triage
6. On the permissions screen, **check all boxes** — especially:
   - "Read, compose, send, and permanently delete all your email from Gmail"
   - "See, edit, share, and permanently delete all the calendars you can access using Google Calendar"
7. Click **Allow**
8. **Restart Claude Code** — close the app or terminal completely and reopen it

When Claude Code restarts, it picks up the new MCP connections automatically.

---

## Verifying the Connection

After restarting, type `/inbox-zero` and the skill will verify the connection automatically by listing your Gmail labels. If it works, you'll see a confirmation and move straight to setup.

You can also test manually by asking Claude: "List my Gmail labels" — if it returns your labels, you're connected.

---

## ⚠️ Gmail Write Scope (Labels + Drafts)

If you connected Google with limited permissions, inbox-zero can **read** your email but cannot **create labels or drafts**. You'll see this error:

```
Request had insufficient authentication scopes.
```

**To fix:**
1. Go to **claude.ai → Settings → Integrations**
2. Click **Disconnect** next to Google
3. Click **Connect** again
4. On the permissions screen, make sure ALL Gmail boxes are checked — particularly compose and modify permissions
5. Restart Claude Code

The full Gmail scope is needed for label creation, label application, and draft creation. Read-only scope only enables the triage report.

---

## Reconnecting a Different Google Account

If you want to triage a different Gmail account:
1. Disconnect Google in claude.ai → Settings → Integrations
2. Reconnect and sign in with the other account
3. Restart Claude Code
4. Run `reconfigure inbox-zero` to update your config if your context paths differ

---

## Troubleshooting

| Problem | Likely cause | Fix |
|---------|-------------|-----|
| "Gmail MCP unavailable" | Google not connected, or Claude Code not restarted | Connect Google in claude.ai Integrations, restart Claude Code |
| "Insufficient authentication scopes" | Read-only scope | Disconnect and reconnect with all Gmail permissions checked |
| Labels created but not visible in Gmail | Gmail label sync delay | Wait 30 seconds and refresh Gmail |
| Calendar shows no events | Wrong calendar connected | Check that the correct Google account is connected |
| Notion 401 error | Integration token expired | Create a new token at notion.so/my-integrations, run `reconfigure inbox-zero` |

---

## Outlook / Microsoft 365

Not supported. Microsoft 365 requires admin approval to connect as an MCP integration, which makes it impractical for personal use. Only Gmail is supported today.
