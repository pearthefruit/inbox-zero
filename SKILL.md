---
name: inbox-zero
description: Triage unread emails (last 30 days), draft replies using calendar and notes context, apply labels, and print a triage report. Invoke with /inbox-zero.
version: 2.0.0
---

# Inbox Zero

AI-powered email triage for Gmail. Classifies your unread emails into three buckets, drafts replies for anything that needs a response using your calendar and notes as context, applies labels to organize your inbox, and prints a clean triage report.

## Core Principles

1. **Never send — only draft.** You create drafts. The user reviews and sends manually. This is non-negotiable.
2. **Classify decisively.** Every email gets a category. No "maybe" pile. When in doubt, lean toward `noise`.
3. **Context before keys.** For action-needed emails, gather calendar and notes context *before* writing a single word of the draft. A contextless draft is worse than no draft.
4. **Short is better.** Match the length of the reply to the length of the ask. Never pad. No filler pleasantries.

---

## Phase 0: Detect Mode

Check for a mode signal in the user's invocation:

- **"quick"** or **"scan only"** → **Classify-Only mode**: skip drafting, skip labels, just classify and report.
- **"reconfigure"** or **"reset"** → **Reconfigure mode**: delete `~/.inbox-zero/config.json` and run Phase 1 (Setup Wizard).
- **Anything else (default)** → **Full Triage mode**: classify + draft + label + report.

Then check `~/.inbox-zero/config.json`:
- If the file **does not exist** → run Phase 1 (Setup Wizard), then continue.
- If the file **exists** → skip to Phase 2.

---

## Phase 1: Setup Wizard

Run this when no config exists or the user reconfigures.

### Step 1.1: Introduction

Tell the user:

"Welcome to Inbox Zero. I need to set up three things before we start — this takes about 2 minutes."

### Step 1.2: Ask All Setup Questions

Ask all three questions at once using a single AskUserQuestion call:

**Question 1 — Email Client** (header: "Email"):
Which email client do you use?
Options: Gmail / Outlook *(coming soon)* / Other *(coming soon)*

**Question 2 — Calendar** (header: "Calendar"):
Reference your calendar when drafting replies? Helps suggest real availability for scheduling emails.
Options: Google Calendar *(recommended)* / Skip

**Question 3 — Context Source** (header: "Context"):
Where should I look for context when drafting replies? (notes, files, a database, etc.)
Options: Notes folder *(Obsidian, markdown, etc.)* / Specific file / Notion database / None

For whichever context option they pick, ask a follow-up for the path or credentials. Use the AskUserQuestion call for these too if possible, otherwise ask inline.

### Step 1.3: Verify Connections

**If Gmail:** List Gmail labels to verify the MCP connection. If it fails, read [mcp-setup.md](mcp-setup.md) and walk the user through connecting. Stop and wait for confirmation before proceeding.

**If Google Calendar:** List calendars to verify the connection. If it fails, reference [mcp-setup.md](mcp-setup.md). Continue even if unavailable — just note it in config.

**If Notes folder:** Verify the folder path exists. Ask for a corrected path if it doesn't.

**If Specific file:** Verify the file exists. Ask for a corrected path if it doesn't.

**If Notion:** Save the token and DB ID — no upfront verification needed.

**If Outlook or Other:** Tell them Outlook is coming soon. Gmail is supported today. Stop.

### Step 1.4: Save Config

Create `~/.inbox-zero/` and write `~/.inbox-zero/config.json`:

```bash
mkdir -p ~/.inbox-zero
```

```json
{
  "email": {
    "type": "gmail"
  },
  "calendar": {
    "type": "google_calendar",
    "days_ahead": 7
  },
  "context": {
    "type": "folder",
    "path": "/full/expanded/path/to/notes"
  },
  "processing": {
    "days_back": 30,
    "max_emails": 30
  },
  "onboarding_complete": true
}
```

Tell the user: "All set. Starting triage..."

---

## Phase 2: Fetch Emails

Calculate the date 30 days ago. Search Gmail:
- Query: `is:unread after:YYYY/MM/DD`
- Fetch up to `processing.max_emails` threads (default: 30)

If no unread emails: tell the user "Your inbox is clean — no unread emails in the last 30 days." Stop.

---

## Phase 3: Classify

Read [classification-rules.md](classification-rules.md) for detailed criteria, then classify each thread:

- **action_needed** — requires a response, decision, or act from the user
- **important** — worth reading, no reply needed
- **noise** — safely ignored

Be decisive. Every email gets exactly one category.

---

## Phase 4: Context & Drafts *(Full Triage only — skip in Classify-Only mode)*

For each `action_needed` email:

**4a. Gather calendar context** (skip if `calendar.type` is `"none"`):
Fetch the next `calendar.days_ahead` days of events. Note free slots, busy periods, and anything relevant to the email topic.

**4b. Gather notes context** (skip if `context.type` is `"none"`):
- **folder**: Find 1–3 files in `context.path` most relevant to the email subject or sender. Prioritize `.md` and `.txt`. Cap at ~3000 tokens total.
- **file**: Read the configured file (or a relevant section for large files).
- **notion**: Query the database — see [classification-rules.md](classification-rules.md) for the API call pattern.

**4c. Draft the reply:**
Read [draft-guidelines.md](draft-guidelines.md) for tone, structure, and anti-patterns. Create the draft in Gmail, threaded to the original email.

If you cannot write a useful draft (completely ambiguous request, no actionable content), mark it `draft_skipped` and note the reason.

---

## Phase 5: Apply Labels *(Full Triage only)*

Ensure these labels exist (create if missing):
- `inbox-zero/action-needed` — red `#fb4c2f`
- `inbox-zero/important` — orange `#ffad47`
- `inbox-zero/noise` — gray `#999999`

Apply the appropriate label to each thread.

⚠️ **If label creation fails with "insufficient authentication scopes":** Skip labeling silently and note it in the report with the fix. See [mcp-setup.md](mcp-setup.md) for how to reconnect with write access.

---

## Phase 6: Report & Follow-Up

### Step 6.1: Print the Triage Report

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  INBOX ZERO  |  {today's date}
  {N} emails processed  •  last 30 days
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

ACTION NEEDED  ({N} emails, {N} drafts created)

  1. {Subject}
     From: {Name} <{email}>
     Draft: {one-line summary of the drafted reply}

  2. {Subject}
     From: {Name} <{email}>
     Draft: [skipped — {reason}]

IMPORTANT, NO ACTION  ({N} emails)

  1. {Subject} — {sender}
  2. {Subject} — {sender}

NOISE  ({N} emails)

  Newsletters & Digests
  1. {Subject} — {sender}

  Promotions
  2. {Subject} — {sender}

  Automated Notifications
  3. {Subject} — {sender}

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
{Labels applied. / ⚠️ Labels skipped — see note above.}
{N drafts created. Review Gmail Drafts to send replies.}
```

Group noise by sub-category (Newsletters, Promotions, Automated Notifications) for scannability.

### Step 6.2: Offer Follow-Up

After the report, ask:

"Want to do anything else?
(1) Schedule daily triage — run `/inbox-zero` automatically each morning
(2) Process more emails — fetch the next 30
(3) Fix Gmail write access — reconnect Google with full permissions
(4) Done"

Handle each:
- **(1) Schedule:** Refer the user to `/schedule` to set up a recurring run.
- **(2) More emails:** Re-run Phase 2 with the `nextPageToken` from the first fetch, then Phases 3–6.
- **(3) Fix access:** Read [mcp-setup.md](mcp-setup.md) and walk the user through reconnecting with write scope.
- **(4) Done:** Stop.

---

## ⚠️ Gotchas

| Situation | Symptom | Fix |
|-----------|---------|-----|
| Gmail read-only scope | Label/draft creation fails with "insufficient authentication scopes" | Reconnect Google at claude.ai → Settings → Integrations with all Gmail boxes checked. See [mcp-setup.md](mcp-setup.md). |
| Context folder too large | Slow triage, context window pressure | The skill already caps at 3000 tokens. If still slow, set `context.type` to `"file"` pointing at a focused reference doc. |
| Notion token expired | Curl returns 401 | Create a new token at notion.so/my-integrations, run `reconfigure inbox-zero` to update. |
| No emails found | "Inbox clean" message | The query only fetches unread. Mark emails unread in Gmail if you want them triaged again. |
| Wrong emails classified | Mis-categorized threads | Classification rules are in [classification-rules.md](classification-rules.md) — they can be tuned. |

---

## Supporting Files

| File | Purpose | When to read |
|------|---------|--------------|
| [classification-rules.md](classification-rules.md) | Detailed criteria + examples for all three categories | Phase 3 (classify) |
| [draft-guidelines.md](draft-guidelines.md) | Tone, structure, length, anti-patterns for drafts | Phase 4 (draft) |
| [mcp-setup.md](mcp-setup.md) | Step-by-step Gmail + Calendar MCP connection guide | Phase 1 (setup) + Gotchas |
