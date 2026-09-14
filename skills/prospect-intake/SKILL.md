---
name: prospect-intake
description: Start a new prospect's intake, with or without their files in hand — a request naming a new prospect and attaching no documents is a normal starting point, because Step 1 searches the connected tools or asks the advisor to upload. Cleans and normalizes their account files (statements, exports, screenshots) into a consolidated view, then hands back a plain-English prospect summary, a detailed analyst handoff the full proposal gets built from, and a friendly "what to expect" memo, offered as a Gmail draft on approval. Does not produce the proposal, recommend an allocation, or quote fees. Triggers on "prospect intake", "/prospect-intake", "new prospect intake", "clean up [prospect]'s statements", "process [prospect]'s files", "what to expect note for [prospect]" — and on how an advisor raises a new prospect before any file exists: "I met with a potential client/customer", "[name] is interested in becoming a client", "I have a new prospect", "how should I proceed with [name]".
---

# Prospect Intake

Turn the pile of statements a prospect hands over into something usable — a clean summary they can read, a clean handoff the analyst team can build a full proposal from, and a warm note that keeps the prospect engaged while that happens.

This skill is the **intake step**, not the proposal itself. The full investment proposal (proposed allocation, expected outcomes, fees, transition plan) is built by the firm's analysts from the detailed handoff this skill produces — it is not generated here.

## Inputs

Required: the **prospect's name**. Their files are needed too — either found in a connected tool or uploaded — but a request that arrives without them is a normal starting point, not a reason to hold off: **Step 1: Find the Files** below is how they get found, and it is where this skill begins. Never invent holdings.

## Step 1: Find the Files

Ask the advisor a single question up front: *do you want me to look for [prospect]'s files in your connected tools, or would you rather upload them here?* Get the prospect's name either way — it's needed for both paths.

**A. Look in connected tools**

Check which systems are actually connected — call `ListConnectors` (load via `ToolSearch` if it isn't already available) — for a CRM (Redtail, Salesforce, Wealthbox), Google Drive, Gmail, and Microsoft 365. Don't guess at a tool-name prefix; for any system whose tools are loaded, use `ToolSearch` with that system's name to find its actual tools.

**Look for the tools before you trust the registry.** `ToolSearch` by the system's own name is the check that decides: if its tools come back, that system is connected and callable — use them. `ListConnectors` can answer "No installed connectors found" even in a session with several live, working connectors, and in some clients it renders a user-facing card rather than returning data at all. So it explains a gap, it never establishes one, and an empty result means **unknown**, never "nothing is connected". When it does return entries, read `enabledInChat`, not `connected`: `connected: true` with `enabledInChat: false` is authenticated but switched off for this chat, so tell the advisor they can enable it here rather than reporting it as unconnected; a missing or `null` `connected` is unknown, not disconnected.

Search all connected systems **in parallel** — one system coming back empty doesn't block reading the others. If this pull is likely to take a moment (several systems connected), say so up front; that's a heads-up, not another "should I start?" gate — the upload-vs-search question above is the only required input before proceeding.

Show the advisor what matched (file names, email subjects, CRM record) before reading any of it — confirm which items are actually the prospect's files before pulling content from them. **Disambiguate first:** if a CRM search turns up more than one matching contact or opportunity, don't pull from or attach to any of them until the advisor confirms which one is correct — same privacy bar as a household match in other skills: never proceed on a name match alone.

If Zocks is also connected, check it for a prior conversation with this prospect and pull whatever facts it already captured (goals, life details, account mentions). Reuse those instead of re-asking the advisor or the prospect for them, and cite them as coming from that conversation rather than treating them as missing in Step 2.

If none of CRM/Drive/Gmail/M365 are connected, say so plainly and fall back to upload.

**B. Upload here**

Wait for the advisor to provide files — PDF/CSV/screenshot, or pasted text. Don't proceed on a promise of files to come; wait for them to actually arrive. Never invent holdings to fill a gap while waiting.

## Step 2: Clean the Files

**Dispatch one `claude-for-financial-advisors:statement-extract` subagent per file, all in a single message so the pile is read concurrently** — one `Agent(claude-for-financial-advisors:statement-extract)` call per file, in the same response. Each gets one file path and the prospect's name, and returns that file's accounts as a normalized record with data-quality flags and its inferred-vs-read marking already done. Reading a stack of statements one after another is the slowest part of this skill, and it gets slower the bigger the pile — which is exactly when the advisor is waiting longest.

Consolidation stays with you: the subagents each see one file and cannot tell that an account in file 3 is the same account as one in file 7. Duplicate detection across the set, and the single consolidated view, are yours to assemble from their records.

The extraction schema each subagent works to, which is also what to do by hand if the subagent is unavailable — per account:
- Account type/registration (401(k), IRA, brokerage, etc.) and custodian/provider if shown
- Holdings: ticker/fund name, shares or units, current value, cost basis if present
- Fees visible on the statement
- Statement/export date
- Any restrictions noted (e.g., employer plan in-service withdrawal rules, vesting)

Normalize everything into one consolidated view across all files/accounts. Flag data-quality problems as you go rather than silently working around them:
- Illegible or partial scans
- Missing cost basis
- Stale statement dates
- Accounts that appear duplicated across files

Never fabricate a figure that isn't legible or present — mark it "— not shown on file provided," unless it's a fact Zocks already captured in a prior conversation (Step 1), in which case use that instead of flagging it missing.

## Step 3: The Three Outputs

Use the templates in this skill's `templates/` folder as the skeleton for each. Fill every field; where data wasn't shown on any file provided, mark it "— not shown on file provided" rather than leaving it blank silently.

**Inferred vs. Read applies to every figure in all three outputs**, not just the Analyst Handoff below — any number in the Prospect Summary or What-to-Expect memo that was computed or assumed rather than read directly off a file needs the same flag.

### A. Prospect Summary
Template: `templates/prospect-summary-template.md`. Plain-English, prospect-facing. Sections:
1. **Accounts in Scope** — account types/registrations reviewed (no account numbers, no custodian internals beyond what's needed to identify the account type)
2. **Holdings Snapshot** — a high-level view of what's held, by account
3. **Total Assets Under Review** — the sum across all accounts

**No asset mix/allocation commentary, no fees, no performance or return claims** — this confirms "here's what we received and understood," nothing more.

### B. Analyst Handoff
Template: `templates/analyst-handoff-template.md`. Internal, for the firm's analyst team — this one stays internal (see Step 4). Sections:
1. **Per-Account Line Items** — the full consolidated detail from Step 2: account type/custodian, each holding with shares/units, value, cost basis, fees, statement date, restrictions
2. **Data Quality Flags** — illegible/partial scans, missing cost basis, stale statement dates, duplicate accounts across files, called out per account/line item
3. **Inferred vs. Read** — for every figure that required inference (e.g., computed from other numbers, assumed from context) rather than being read directly off a file, mark it as inferred; everything else is read-off-file by default

### C. What-to-Expect Memo
Template: `templates/what-to-expect-template.md`. Brief, friendly, prospect-facing. Sections:
1. **What We Received** — the files/accounts reviewed
2. **What Happens Next** — the firm's analysts are building the full proposal from this intake
3. **Who You'll Hear From, and When** — ask the advisor for the actual next point of contact and timeline; never invent a turnaround time, a name, or promise a specific outcome. If the advisor hasn't given you these yet, ask before finalizing this section rather than leaving a guess in place.

## Step 4: Compliance

Run **/compliance** on outputs **A (Prospect Summary)** and **C (What-to-Expect Memo)** — both are prospect-facing and fall under the same marketing/antifraud rules as client communications.

**Finish the review before you hand anything back.** /compliance writes two files per document it reviews — `<slug>-compliance-analysis.md` and `<slug>-compliance-redraft.md`. Until both exist for **both** A and C, the review has not happened yet and the intake is not done. A clean review still writes a redraft, so "nothing needed changing" is not a reason for the file to be missing. Nothing here runs in the background: if you are waiting on a scan, it has already come back and the remaining markup is yours to write.

**From here on, A and C mean the redrafts.** When you hand the documents back, list them for attaching, or offer to send them, name `<slug>-compliance-redraft.md` — never the original draft. If you show the advisor a list of files, say which one goes to the prospect; an undifferentiated list with the original and the redraft side by side is how the pre-compliance draft gets sent by mistake.

**Output B (Analyst Handoff) stays internal and does not go through /compliance** — it's not prospect-facing.

## Step 5: CRM

After the three outputs are ready, show the advisor one summary of what you're about to do — attach A, B, and C to the existing contact/opportunity confirmed in Step 1, or create a new contact and attach them there — and get a single confirmation on that whole batch, not a separate approval per document. Only write after they say yes; never attach or create anything before that.

## Step 6: Email the What-to-Expect Memo

If Gmail is connected, offer to send the compliance-reviewed version of C to the prospect: draft the message (subject and body) and show the advisor the full draft before anything goes out. Only send after they explicitly approve it — never send automatically, and never send the pre-compliance draft.

If Gmail isn't connected, or the advisor would rather send it themselves, hand back the markdown file from Output instead — this step is optional, not required.

## Output

Write all three as markdown files first, clearly labeled: the prospect summary (A), the analyst handoff (B), and the what-to-expect memo (C). Remind the advisor that A and C need compliance review before going out (Step 4), that B stays internal, and that the full proposal itself is a separate next step the analyst team builds from B.

Then ask the advisor whether they'd like the prospect-facing pieces (A and C) converted to .docx or .pdf — don't create either format unless they ask. B stays internal and markdown is fine for it either way.

## Out of Scope (for now)

- **No proposed allocation, fee schedule, or expected-outcomes modeling** — that's the full proposal the analysts build from this skill's handoff.
- **No transition/ACAT paperwork.**
- **No performance projections or return assumptions.**

## Important Notes

- Never fabricate holdings, values, or cost basis from illegible or partial files — mark it as not shown rather than estimating.
- Treat the prospect's files as confidential, whether pulled from a connected tool or uploaded.
- The prospect isn't yet a client — don't create or modify any CRM record without the advisor's explicit go-ahead (see Step 5).
- Same rule for email: never send the Step 6 draft without the advisor's explicit approval. Claude drafts, the advisor decides.
