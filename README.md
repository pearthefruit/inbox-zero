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
- A [claude.ai](https://claude.ai) account with Google connected via **claude.ai Settings → Connectors** (not the desktop app)

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

The skill uses Claude's built-in Google connector. **You must connect via the claude.ai web app** — the desktop app does not expose the full permission scope list.

1. Open **[claude.ai](https://claude.ai)** in a browser → **Settings** → **Connectors**
2. Click **Connect** next to Gmail
3. On the Google permissions screen, **check every box** — especially:
   - "Read, compose, send, and permanently delete all your email from Gmail"
4. Repeat for **Google Calendar** if you want availability context in drafts
5. **Restart Claude Code**

> **Why every box?** Read-only scope lets the skill fetch and classify emails. The write scope is required to create drafts and apply labels. If you skip the write boxes, drafts and labels will silently fail.

No API keys. No OAuth app to configure. That's it.

---

## First Run

Open Claude Code and type:

```
/inbox-zero
```

On first run, a 2-minute setup wizard asks:
- Your email client (Gmail)
- Whether to reference Google Calendar when drafting (recommended)
- Where to pull context from when drafting replies (notes folder, file, Notion, or none)

Your config is saved to `~/.inbox-zero/config.json` and reused on every subsequent run.

---

## Modes

| Invocation | Mode | What happens |
|---|---|---|
| `/inbox-zero` | Full triage | Classify + draft + label + report |
| `/inbox-zero quick` | Scan only | Classify + report, no drafts or labels |
| `/inbox-zero reconfigure` | Setup wizard | Re-run onboarding, update config |

---

## Categories

Categories define how your inbox gets sorted. Each one gets a Gmail label, a color, a description that guides classification, and a flag for whether to draft replies.

The defaults work for most people:

| id | label | description | draft |
|----|-------|-------------|-------|
| `action_needed` | `inbox-zero/action-needed` | Requires a response, decision, or action | yes |
| `important` | `inbox-zero/important` | Worth reading but no reply needed | no |
| `noise` | `inbox-zero/noise` | Safely ignored — newsletters, promos, alerts | no |

**To use your own:** the setup wizard will ask on first run. You can define 2–5 categories with any names and descriptions. You can also edit `~/.inbox-zero/config.json` directly or run `reconfigure inbox-zero` at any time.

Example custom setup for a founder:
```json
"categories": [
  { "id": "urgent", "label": "triage/urgent", "color": "#fb4c2f",
    "description": "Needs a reply today — customer issues, investor asks, anything time-sensitive", "draft": true },
  { "id": "investors", "label": "triage/investors", "color": "#a67cc5",
    "description": "From investors, advisors, or board members", "draft": true },
  { "id": "later", "label": "triage/later", "color": "#ffad47",
    "description": "Worth reading but not urgent", "draft": false },
  { "id": "ignore", "label": "triage/ignore", "color": "#999999",
    "description": "Automated, promotional, or low-signal", "draft": false }
]
```

---

## Context Sources

The skill pulls in external context before writing any draft reply:

| Type | What it does |
|------|-------------|
| `folder` | Reads relevant `.md`/`.txt` files from a notes folder (Obsidian, etc.) |
| `file` | Reads a single reference file — resume, bio, project doc, etc. |
| `notion` | Queries a Notion database via the API |
| `none` | Drafts without external context |

See [examples/](examples/) for sample configs.

---

## Reconfiguration

To change your email, calendar, or context source:

```
reconfigure inbox-zero
```

---

## Example Triage Report

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  INBOX ZERO  |  2026-05-07
  18 emails processed  •  last 30 days
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

ACTION NEEDED  (2 emails, 2 drafts created)

  1. Re: Q2 Budget Review
     From: Sarah Chen <sarah@company.com>
     Draft: Confirmed availability for May 8 review; flagged headcount line item

  2. Interview scheduling — Software Engineer Role
     From: recruiting@example.com
     Draft: Proposed Tue May 12 2pm or Thu May 14 10am

IMPORTANT, NO ACTION  (7 emails)

  1. Merge request approved — gitlab@company.com
  2. Your flight confirmation ORD→JFK May 15 — delta@email.delta.com
  ...

NOISE  (9 emails)

  Newsletters & Digests
  1. Weekly digest: Engineering — noreply@weekly.io

  Promotions
  2. 50% off weekend sale — bananarepublic@email.br.com

  Automated Notifications
  3. New connection on LinkedIn — notifications-noreply@linkedin.com

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Labels applied. 2 drafts created. Review Gmail Drafts to send replies.
```

---

## Troubleshooting

| Problem | Fix |
|---------|-----|
| Drafts or labels fail with "insufficient authentication scopes" | Reconnect Google via **claude.ai web** (not desktop app), check **all** Gmail permission boxes, restart Claude Code |
| "This app is blocked" when connecting | You're signing into a managed Google Workspace account. Use a personal Gmail instead. |
| No emails found | The skill only fetches unread threads. Mark emails unread in Gmail if you want them triaged again. |
| Wrong emails classified | Classification rules are in [classification-rules.md](classification-rules.md) — edit to tune behavior. |

---

## Roadmap

- [ ] Configurable per-sender rules ("always mark X as noise")
- [ ] Scheduled daily runs via `/schedule`
- [ ] Push notification on triage complete

---

## License

MIT
