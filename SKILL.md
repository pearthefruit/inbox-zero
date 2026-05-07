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
Options: Gmail

**Question 2 — Calendar** (header: "Calendar"):
Reference your calendar when drafting replies? Helps suggest real availability for scheduling emails.
Options: Google Calendar *(recommended)* / Skip

**Question 3 — Context Source** (header: "Context"):
Where should I look for context when drafting replies? (notes, files, a database, etc.)
Options: Notes folder *(Obsidian, markdown, etc.)* / Specific file / Notion database / None

For whichever context option they pick, ask a follow-up for the path or credentials. Use the AskUserQuestion call for these too if possible, otherwise ask inline.

**Question 4 — Categories** (header: "Categories"):
How do you want to sort your inbox? Each category gets a Gmail label and a description that guides how emails are classified.

Show the defaults and ask if they want to keep them or define their own:

Defaults:
1. **Action needed** (`inbox-zero/action-needed`, red) — requires a response, decision, or action *(drafts replies)*
2. **Important** (`inbox-zero/important`, orange) — worth reading, no reply needed
3. **Noise** (`inbox-zero/noise`, gray) — newsletters, promos, alerts — safely ignored

If they want to customize: ask for a list of 2–5 categories. For each, collect:
- A display name (used as the Gmail label, e.g. `inbox-zero/urgent`)
- A one-sentence description of what belongs in it
- Whether to draft replies for emails in this category (yes/no)

Accept blank input as "keep defaults".

### Step 1.3: Verify Connections

**If Gmail:** List Gmail labels to verify the MCP connection. If it fails, read [mcp-setup.md](mcp-setup.md) and walk the user through connecting. Stop and wait for confirmation before proceeding.

**If Google Calendar:** List calendars to verify the connection. If it fails, reference [mcp-setup.md](mcp-setup.md). Continue even if unavailable — just note it in config.

**If Notes folder:** Verify the folder path exists. Ask for a corrected path if it doesn't.

**If Specific file:** Verify the file exists. Ask for a corrected path if it doesn't.

**If Notion:** Save the token and DB ID — no upfront verification needed.

**If anything other than Gmail:** Tell them only Gmail is supported. Stop.

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
  "categories": [
    {
      "id": "action_needed",
      "label": "inbox-zero/action-needed",
      "color": "#fb4c2f",
      "description": "Requires a response, decision, or action from me",
      "draft": true
    },
    {
      "id": "important",
      "label": "inbox-zero/important",
      "color": "#ffad47",
      "description": "Worth reading but no reply needed",
      "draft": false
    },
    {
      "id": "noise",
      "label": "inbox-zero/noise",
      "color": "#999999",
      "description": "Can be safely ignored — newsletters, promos, automated alerts",
      "draft": false
    }
  ],
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

Read the user's categories from `config.categories`. Build the classification criteria dynamically:

```
Classify this email into exactly one of the following categories:
{for each category in config.categories}
- {category.id}: {category.description}
{end}
```

Use [classification-rules.md](classification-rules.md) as supporting guidance for edge cases, but the user's category descriptions are authoritative. Every email gets exactly one category. When in doubt, assign the last category (typically the catch-all / lowest-priority one).

---

## Phase 4: Context & Drafts *(Full Triage only — skip in Classify-Only mode)*

For each email classified into a category where `draft: true` in `config.categories`:

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

For each category in `config.categories`:
- Ensure the label `category.label` exists (create with `category.color` if missing)
- Apply that label to all threads classified into that category

⚠️ **If label creation fails with "insufficient authentication scopes":** Skip labeling silently and note it in the report with the fix. See [mcp-setup.md](mcp-setup.md) for how to reconnect with write access.

---

## Phase 6: Report & Follow-Up

### Step 6.1: Print the Triage Report

Print one section per category, in the order they appear in `config.categories`. For categories with `draft: true`, show draft summaries. For categories with `draft: false`, list subject and sender only.

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  INBOX ZERO  |  {today's date}
  {N} emails processed  •  last 30 days
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

{CATEGORY LABEL — UPPERCASE}  ({N} emails{, N drafts created})

  1. {Subject}
     From: {Name} <{email}>
     Draft: {one-line summary}        ← only for draft:true categories

  2. {Subject}
     From: {Name} <{email}>
     Draft: [skipped — {reason}]

{NEXT CATEGORY}  ({N} emails)

  1. {Subject} — {sender}
  2. {Subject} — {sender}

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
{Labels applied. / ⚠️ Labels skipped — see note above.}
{N drafts created. Review Gmail Drafts to send replies.}
```

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
| Outlook-*.png attachments on draft | Sender uses Outlook — Gmail pulls inline signature images into the reply thread | Delete the attachments before sending. They are not part of your reply. |
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
