---
name: post-meeting
description: Turn a client meeting into next-best-action follow-up for a financial advisor — no drafting required. Given a meeting (via Zocks, Wealthbox, or a pasted transcript), finds the right household, drafts a CRM note plus the action items and opportunities it heard, shows them all in one batch for approval, then writes what's approved and offers to file the record in Drive. Triggers on "log the meeting", "log my meeting with [client]", "meeting follow-up", "post meeting", "/post-meeting", or "follow up on my meeting with [client]".
---

# Post Meeting

Turn a client meeting into a CRM note plus a confirmed set of follow-ups and opportunities — the write-up, to-do chasing, and opportunity-spotting advisors would otherwise do by hand, done for them, with the advisor approving every write before it happens. This is next-best-action: doing the follow-up, not drafting an email for the advisor to send. If the advisor wants something sent to the client, that goes through `/compliance` — this skill's default path is the CRM, not a client-facing message.

## Inputs

Required: **the meeting itself** — identified by household/client name, or by whatever meeting metadata a connected source supplies. Pull the meeting's content in this order, stopping at the first that's available:

1. **Zocks AI results** — the meeting's AI-generated summary and extracted items, if Zocks is connected. Fall back to the raw transcript (also from Zocks) for direct quotes when the summary is ambiguous about wording.
2. **Wealthbox meeting summary** — when Zocks isn't connected, Wealthbox's notetaker exposes a summary of the meeting (not a raw transcript). Use it as-is.
3. **Pasted transcript** — if neither connector is available, ask the advisor to paste the transcript.

Zoom is not a supported source yet — do not offer it or imply it's coming.

Infer the household and the meeting date rather than asking by default: from the transcript itself, from Zocks meeting metadata (date and meeting ID, when present), or from the advisor's connected calendar. Only ask the advisor to identify the household or supply a date if none of those resolve it — never show a date picker as a first resort. Once you've inferred a date, put it on the note preview in Step 2 so the advisor can correct it if it's wrong.

Once the household and meeting are identified, don't ask "should I start?" — pull the meeting content per the waterfall above and go. The approval gate in this skill is on the write batch in Step 4, not on starting the work.

## Workflow

### 1. Identify the household

Query the household's CRM by the client/household name. **If more than one connected tool could try to solve this for the household**, ask the advisor once which is the book of record — per the Ask-Once, Then Route Convention — rather than guessing, and use that one for the rest of this workflow, including the Step 5 write. Offer to help the advisor save the choice using the Personalization Convention so they aren't asked again next session. If the advisor mentions a second CRM or notes source later in the workflow, that's the same question again before anything from it gets merged in — and if two sources ever disagree on a material fact, stop and ask rather than writing a footnoted or averaged version back to the CRM.

**Disambiguation rule:** never proceed on a name match alone. If more than one household matches, show the advisor the candidates (name + household + masked email or last-activity date) and ask which one is correct before touching any record — logging a meeting against the wrong household is a privacy incident, not a minor mistake. Confirm identity even on a single match if anything about the context (household, recent activity) doesn't line up with what the advisor said.

Follow the **Connector Placeholder Convention**: if the CRM's tools aren't available in this session, say "This is where I'd search [system] for this household once that connector is built," then ask the advisor to confirm the household manually and continue.

Whatever the CRM record turns up here — prior notes, open tasks, past meetings — is auxiliary context for identifying the household and spotting duplicates in Step 3. It is never the source of *this* meeting's content; that always comes from the waterfall in Inputs above, with Zocks AI results as the primary source when connected.

### 2. Draft the meeting note

Draft the note (the meeting content pulled via the Inputs waterfall, plus the inferred meeting date/type) and show the advisor what will be written, with the inferred date called out so it's easy to correct. Getting this right before writing matters more here than for most writes — CRM notes are never edited in place, only replaced by deleting and recreating them.

### 3. Identify action items and opportunities

Read the meeting content for two distinct things and list them back to the advisor separately:

- **Action items** — commitments the advisor made or next steps that were agreed to (e.g., "I'll send the updated plan," "let's set up a call about the 529").
- **Opportunities** — things mentioned in passing that could grow or protect the relationship (e.g., "a CD maturing next month," "mentioned an inheritance," "unhappy with a held-away account"). These are signals from *this meeting only*, not a full CRM opportunity review — call out that a deeper look (a dedicated opportunity-review pass) is a separate step if the advisor wants one.

Only include things actually said in the meeting content — never infer a commitment or opportunity that isn't there, and label anything uncertain as a possibility rather than a fact.

**Dup-check before proposing writes:** Zocks (the product, separately from its MCP) may already export its own summaries, tasks, and opportunities into the CRM. Check the CRM for tasks/opportunities already logged against this household around the meeting date — **at both grains, the household record's own items and each person contact's** (household-linked items are invisible to contact-only lookups) — before drafting new ones, and drop anything that's already there rather than proposing a duplicate.

### 4. Show the batch for approval

Present everything from Steps 2–3 as **one schema-shaped table**, not prose, and ask for a single confirmation on the whole batch rather than approving items one by one:

| Type | What | When | Who | Notes |
|---|---|---|---|---|
| Note | (note preview, with inferred date) | meeting date | household | — |
| Task | action item text | due/target date if known | owner (usually the advisor) | — |
| Opportunity | opportunity description | — | — | any metadata the connected CRM exposes (e.g. estimated size, stage) |

The advisor can approve all, some, or edited versions of any row.

### 5. Write what's approved

For whichever rows the advisor approves, create the corresponding CRM record using that CRM's own entities — Wealthbox's Note/Task/Opportunity objects, or Redtail's note/activity/opportunity calls — rather than assuming a generic mapping (e.g. Salesforce custom fields) that may not exist for the connected system. Confirm back with a short summary of what was logged.

### 6. Offer to file the record in Drive

After the CRM writes are done, if Drive is connected, offer to file the meeting record (as markdown) there too. If Drive isn't connected, skip this step silently — never hold up the CRM writes on it.

## Output

- The meeting note, action items, and opportunities are written directly to the CRM per the approved batch — that's this skill's primary output, not a document.
- If filing the record to Drive, file it as markdown. Don't create a `.docx`/`.pdf`/`.xlsx` version unless the advisor asks for one.

## Out of Scope (for now)

- **Creating a new household/contact** when no match is found — flag it to the advisor instead.
- **Editing any existing contact field** (name, address, etc.) noticed in the meeting content — flag it instead.
- **A full opportunity-analysis pass** against existing CRM records — this skill only surfaces what's in this meeting.
- **Sending anything to the client.** If the advisor wants to send a follow-up message, route it through `/compliance` — this skill's default path is the CRM, not a client email.
- **Trade execution.**

## Important Notes

- **Every write pauses for advisor approval**, gated through the single batch in Step 4. Never create anything the advisor hasn't seen and confirmed.
- **Never fabricate** meeting content, action items, opportunities, or client details. If the meeting content is ambiguous about whether something was a firm commitment or a real opportunity, ask rather than assume.
- **Treat the meeting content as confidential client data** — it may contain financial, health, or family details beyond the meeting's stated purpose.
