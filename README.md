# Inbox Zero — Claude Code Skill

A Claude Code skill that triages your unread emails, drafts replies with context from your calendar and notes, applies labels to organize your inbox, and prints a triage report — all in one `/inbox-zero` command.

**You keep full control.** The skill only creates drafts. You review and send.

---

## What It Does

1. **Fetches** unread emails from the last 30 days (up to 30 threads)
2. **Classifies** each email into:
   - `action needed` — requires a response or decision
   - `important` — worth reading, no reply needed
   - `noise` — newsletters, promos, automated alerts
3. **Drafts replies** for action-needed emails, using your calendar availability and notes as context
4. **Applies Gmail labels** (`inbox-zero/action-needed`, `inbox-zero/important`, `inbox-zero/noise`)
5. **Prints a triage report** summarizing every email and every draft

---

## Requirements

- [Claude Code](https://claude.ai/code) (desktop app or CLI)
- A [claude.ai](https://claude.ai) account with Google connected

---

## Installation

```bash
# Clone the repo
git clone https://github.com/YOUR_USERNAME/inbox-zero.git

# Copy the skill to your Claude skills directory
# macOS / Linux:
cp -r inbox-zero ~/.claude/skills/inbox-zero

# Windows (PowerShell):
Copy-Item -Recurse inbox-zero "$env:USERPROFILE\.claude\skills\inbox-zero"
```

---

## Connecting Gmail & Google Calendar

The skill uses Claude's built-in Google integration. To enable it:

1. Go to **[claude.ai](https://claude.ai)** → **Settings** → **Integrations**
2. Click **Connect** next to Google
3. Grant access to Gmail and Google Calendar
4. Restart Claude Code

That's it — no API keys, no OAuth apps to configure.

---

## First Run

Open Claude Code and type:

```
/inbox-zero
```

On first run, the skill will walk you through a 2-minute setup:
- Confirm your email client (Gmail)
- Confirm your calendar (Google Calendar or skip)
- Choose a context source for drafting replies (notes folder, specific file, Notion database, or none)

Your config is saved to `~/.inbox-zero/config.json` and reused on every subsequent run.

---

## Context Sources

The skill can reference external context when drafting replies:

| Type | What it does |
|------|-------------|
| `folder` | Reads relevant markdown/text files from a notes folder (Obsidian, etc.) |
| `file` | Reads a single file — great for a resume, bio, or project reference doc |
| `notion` | Queries a Notion database via the API |
| `none` | Drafts without external context |

See [examples/](examples/) for sample configs.

---

## Reconfiguration

To change your email, calendar, or context source:

```
reconfigure inbox-zero
```

To see your current settings:

```
show my inbox-zero config
```

---

## Example Triage Report

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  INBOX ZERO  |  2026-05-06
  18 emails processed  •  last 30 days
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

ACTION NEEDED  (5 emails, 4 drafts created)

  1. Re: Q2 Budget Review
     From: Sarah Chen <sarah@company.com>
     Draft: Confirmed availability for May 8 review; flagged headcount line item

  2. Interview scheduling — Software Engineer Role
     From: recruiting@example.com
     Draft: Proposed Tue May 12 2pm or Thu May 14 10am

  3. Invoice #4821 — Due May 10
     From: contractor@studio.io
     Draft: [skipped — payment approval requires your decision]

IMPORTANT, NO ACTION  (7 emails)

  1. Merge request approved — gitlab@company.com
  2. Your flight confirmation ORD→JFK May 15 — delta@email.delta.com
  3. Weekly digest: Engineering team — noreply@weekly.io
  ...

NOISE  (6 emails)

  1. 50% off weekend sale — bananarepublic@email.br.com
  2. New connection on LinkedIn — notifications-noreply@linkedin.com
  ...

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Labels applied. Review your Gmail Drafts to send replies.
```

---

## Roadmap

- [ ] Outlook support
- [ ] Outlook Calendar support
- [ ] Configurable classification rules (e.g. "always mark X sender as noise")
- [ ] Scheduled runs via cron
- [ ] Summary email / push notification delivery

---

## License

MIT
