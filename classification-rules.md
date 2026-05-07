# Classification Rules

Use these criteria when classifying emails in Phase 3. Every email gets exactly one category. When in doubt, lean toward `noise`.

---

## action_needed

The user must respond, decide, or act. A reply is expected or a task is created.

**Classify as action_needed if:**

| Signal | Examples |
|--------|---------|
| Direct question addressed to the user | "Can you send me the report by Friday?", "Are you available Thursday?" |
| Request for a decision | "Please confirm your attendance", "Do you approve this?" |
| Meeting invite or scheduling request | Calendar invitations, "When are you free to chat?" |
| Document requiring review or signature | Contracts, onboarding forms, NDAs |
| Job application or interview scheduling | "We'd like to schedule a call", "Please complete this assessment" |
| Deadline with action required | "Invoice due May 10", "RSVP by tomorrow" |
| Delegated task or assignment | "Can you take the lead on X?", "I've assigned you to this ticket" |
| Direct message from a real person | Not automated — someone actually wrote to the user |

**Do NOT classify as action_needed if:**
- The email is automated (noreply@, system alert, no reply address)
- The action has already been completed (e.g., a registration confirmation where the user already registered)
- The "ask" is a marketing CTA ("shop now", "upgrade today") — that's noise

---

## important

Worth reading or saving. No reply needed, no task created.

**Classify as important if:**

| Signal | Examples |
|--------|---------|
| Financial statement or account notice | Bank statements, HSA/401k summaries, tax documents |
| Booking or order confirmation | Flight confirmation, hotel receipt, package shipped |
| Upcoming event confirmation | Event registration confirmed, appointment reminder |
| Status update from a project or system the user cares about | PR merged, deploy succeeded, build failed |
| Personal message from a known contact | Friend or family update that requires no reply |
| Informational notice with future relevance | Policy change, service update, legal notice |
| Job offer, rejection, or application status update | Even if no reply is needed right now |

**Edge cases:**
- Automated balance alerts: `important` (financial awareness, no reply needed)
- LinkedIn "someone viewed your profile": `noise` (social notification with no action)
- LinkedIn recruiter reaching out directly: `action_needed` (a real person is asking)
- Newsletter from someone the user explicitly follows: `important` (if it appears to be genuinely read) or `noise` (if mass-sent and generic)

---

## noise

Safely ignored. The user loses nothing by never opening this.

**Classify as noise if:**

| Signal | Examples |
|--------|---------|
| Newsletter or digest | TLDR, Substack, weekly roundups, blog emails |
| Promotional or marketing email | "50% off", "flash sale", "your exclusive offer" |
| Social media notification | LinkedIn likes/comments/follows, Twitter notifications |
| Automated job alert | "New jobs matching your search", "Apply to saved jobs" |
| Apartment/housing listing alert | Automated listings from Zillow, StreetEasy, Apartment List |
| Platform onboarding email | "Get started with X", "Here's what you can do in Y" |
| Mass-sent newsletter from a service | Related Life newsletter, Meetup group suggestions |
| Anything sent to an undisclosed list | BCC campaigns, bulk mailers |

---

## Notion API Pattern

When `context.type` is `"notion"`, use this curl pattern to query the database:

```bash
curl -s "https://api.notion.com/v1/databases/{db_id}/query" \
  -H "Authorization: Bearer {token}" \
  -H "Notion-Version: 2022-06-28" \
  -H "Content-Type: application/json" \
  -d '{"page_size": 20}'
```

Extract page titles and relevant text properties from the response. Use as background context when drafting — don't quote it verbatim in drafts.
