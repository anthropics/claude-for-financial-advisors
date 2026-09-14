---
name: estate-and-tax-brief
description: Prepare a meeting-ready estate and tax brief for a household — opens with what was discussed and actioned since the last meeting (Wealthbox/Redtail/Salesforce), pulls the estate plan from Wealth.com (structure, key documents and whether each is signed, balance sheet) plus the prior-year tax look-back and that year's tax constants, and cross-checks it against how accounts are titled and beneficiaried at the custodian and portfolio platforms (Schwab, Orion/Addepar, iCapital) — surfacing unfunded trusts, titling mismatches, and beneficiary gaps as a prioritized flags list. The only write is creating advisor-approved follow-up tasks in the CRM (optional, approved first). Never drafts trust language, gives legal advice, retitles an account, or changes a beneficiary. Triggers on "estate brief", "estate meeting brief", "estate and tax brief", "estate plan review for [client]", "check [client]'s trust funding", "are [client]'s beneficiaries up to date", or "does [client]'s estate plan match their accounts".
---

# Estate & Tax Brief

Give an advisor a clear, meeting-ready read on whether a household's accounts are actually set up the way its estate plan intends — the mismatch between "what the documents say" and "what the accounts show" is where estate plans quietly fail, often unnoticed until it's too late to fix. Opens with what's already been discussed so the meeting doesn't retread old ground, and closes the loop by turning approved flags into tracked CRM follow-ups.

## Inputs

Required: **household name**. If not provided, ask before doing anything else.

**Disambiguation rule:** confirm the household before pulling anything, the same as every other skill in this plugin — if more than one household matches the name, show the candidates and ask which one before proceeding.

Once identity is confirmed, **pull from all connected sources in parallel** (Steps 1-4) — one slow or missing source degrades only its own section of the brief, not the rest. Before starting this multi-source pull, tell the advisor what you're about to gather and why, so a slow pull doesn't look like a silent hang. And once the household is known, don't ask "should I start?" — narrate what you're doing and go; the approval gates in this skill are the CRM writes in Step 8 and any format conversion at the end, not permission to begin.

## Data Gathering — Steps 1 through 4 run at the same time

The four reads below hit different systems, or different reports on one system, and share no state. **Dispatch them as `claude-for-financial-advisors:source-extract` subagents in a single message** — an `Agent(claude-for-financial-advisors:source-extract)` call per read, all in the same response: one for the CRM, one for the Wealth.com estate plan, one for the Wealth.com tax look-back, and one for each connected custodian/portfolio platform in Step 4. Hand each the household identity as you have it, the one system that subagent is to query, the field schema under its step heading, and the window where the read is time-bounded.

Each returns a filled schema block. Step 5's comparison needs both sides as raw normalized lists, so the subagents are told not to interpret, match, or flag anything — that would pre-empt the comparison with a read you can't audit.

**Identity stays with you**, per the disambiguation rule above. `claude-for-financial-advisors:source-extract` never resolves a household: an `IDENTITY MISMATCH` return comes back with candidates — put them to the advisor, and re-dispatch that one read only once they confirm which household is correct. That is the only return a re-dispatch is right for. A `NOT CONNECTED` return is the Connector Placeholder Convention case — you make the manual-fallback offer, not the subagent — and it is never retried by re-dispatching the same extractor: these four reads fire in one message, and a source that wasn't reachable when they fired doesn't become reachable by firing the same read again, so a second dispatch returns `NOT CONNECTED` again at the same cost. If the advisor connects that source later in the conversation, that's a fresh request rather than a retry. The same goes if more than one connected tool could try to solve the same problem for this household on one of the reads below (not necessarily two of the same kind): ask the advisor once which is the book of record, per the Ask-Once, Then Route Convention, rather than guessing, and offer to help them save the choice using the Personalization Convention. This isn't a one-time check at the start — a source the advisor mentions mid-gathering (a CRM note, an outside document) counts too, and gets the same question before it's merged in. And if two systems were pulled anyway and return the same household at figures apart by orders of magnitude, follow the Magnitude-Conflict Convention: name the conflict, and exclude the outlier's figures from every table and total rather than quoting them as evidence.

## Step 1: Since We Last Met — CRM Context

Query the household's CRM for the household's most recent meeting notes and open action items related to the estate plan (trust funding, titling, beneficiary designations). **Query at both grains: the household record's own notes, tasks, and events, and each person contact's.** In Wealthbox a household is itself a contact with its own id — the same list calls run against it — and estate items are routinely linked to the household record rather than either spouse, so a contact-only lookup silently misses them.

For each one, note whether it's been acted on: done, not done, or no update since it was logged — beside the item, in the section itself, so every action item carries its own status. One line at the end saying no updates are available is not that: the advisor reads the list item by item, and "no update" is a status each item gets on its own line. This becomes the opening section of the brief, so the advisor sees what's changed (or hasn't) before diving into the current comparison.

> **Connector Placeholder Convention:** if the CRM's tools aren't available in this session, say "This is where I'd pull [household]'s last meeting notes and action items from [system] once that connector is built," and put that sentence, with a "— pending [system]" marker, where the Since We Last Met content would go. Then carry on to Steps 2–4 and write the brief (Step 7) with the section in that state. Ask the advisor to summarize the last meeting manually *after* the file is written, not before — a brief with one pending section is the deliverable, while a question with no brief behind it is a stall, and in a single exchange the advisor may never see a file at all. If they've already handed you a summary or their own notes, that is the section's source: build the recap from it and write.

## Step 2: The Estate Plan — What It Intends (Wealth.com)

Pull the household's estate plan from Wealth.com:
- Plan structure: trusts (name, type — e.g., revocable living trust, ILIT — and what each is supposed to hold)
- Key documents (will, trust agreements, powers of attorney, healthcare directives) — and for each, **whether it is actually signed and executed, not merely on file**. Take each document's status from the document inventory as the source reports it; never infer execution status from the document's type or its presence in the file.
- Document-internal problems that keep a document from doing its job — not executed, unsigned, missing pages. These are the estate insights Wealth.com records per document; present them as observations to discuss, not as findings or a legal review, and note that they can change as documents are re-examined.
- The plan's balance sheet: intended owner/beneficiary for each major asset or account

> **Connector Placeholder Convention:** if the Wealth.com connector isn't available in this session, say: *"This is where I'd pull [household]'s estate plan from Wealth.com once that connector is built."* Then offer the fallback: the advisor can paste or upload a trust/estate summary (e.g., a trust schedule of assets, attorney letter, or their own notes on what the plan intends). Don't wait for it — write the brief with this section marked "— pending estate documents" rather than guessing at what the plan says, and fold in anything they provide afterwards as a fresh pass.

## Step 3: Prior-Year Tax Look-Back (Wealth.com)

Pull the household's most recent filed tax year and the tax law that applies to it:

- **Look-back report** — the filed year's actuals: filing status, AGI, taxable income, federal and state income tax, capital gains, dividends, and deductions. Report every effective rate together with the tax dollars and the income amount behind it, so the advisor can reproduce the arithmetic — never a bare percentage.
- **Year-specific tax constants** — the brackets, standard deduction, capital-gains breakpoints, and similar figures for the relevant year, to size what the plan's next moves run into.

**This read is time-bounded**, so the dispatch tells its extractor the window as well as the schema: the household's most recent filed tax year, and that same year for the constants. If the advisor named a year, hand that year instead.

**Cite, never compute.** Every figure here is reported by the source system for the applicable tax year — attribute it to the return or report section an advisor would recognize ("the 2024 federal return shows"). Never recompute, estimate, or fill in a constant Claude wasn't given. If a constant for a future year comes back **projected** rather than published, say so explicitly wherever it appears.

The look-back report covers **filed tax returns only** — it does not exist for a trust, will, or any other estate document, so don't ask for one against those.

Keep this section's job narrow: it is context the advisor brings into the estate conversation (what the household's tax picture actually looked like), not a tax plan or a recommendation. Tax strategy is out of scope — see Out of Scope.

> **Connector Placeholder Convention:** if the Wealth.com connector isn't available in this session, say: *"This is where I'd pull [household]'s prior-year tax look-back and the year's tax constants from Wealth.com once that connector is built."* Then offer the fallback: the advisor can paste or upload a prior-year return or tax summary. Don't wait for it — write the brief with this section marked "— pending tax documents" rather than estimating it, and fold in anything they provide afterwards.

## Step 4: The Accounts — What They Actually Show (Schwab, Orion/Addepar, iCapital)

Pull the household's actual titling and beneficiary data from wherever it lives:
- **Schwab** (custodian): each account's actual registration/titling (individual, joint, IRA/Roth, trust — and if trust, which one) and named beneficiary(ies) on file
- **Orion Connect or Addepar** (portfolio platform): consolidated titling across held-away and multi-custodian accounts beyond what Schwab alone shows
- **iCapital**: titling and beneficiary on alternative investment positions — these are often held in a different entity or trust than the liquid book, so don't assume they match

Follow the same **Connector Placeholder Convention** as Step 2 for any that isn't available: say where the data would come from, offer the manual fallback (paste or upload the household's account list/titling), and continue to the brief without waiting — the comparison table carries "— pending [custodian]" for that account's rows.

The custodian, the portfolio platform, and the alts platform can each report titling or a beneficiary for the same account. If two of them disagree with each other about what's actually on file for a given account — not the Step 5 comparison against what the plan intends, but the systems contradicting each other about present-day fact — that's a material conflict: stop and surface it to the advisor as a blocking question rather than picking one silently or footnoting the discrepancy.

Cross-system titling and beneficiary mismatches are **this skill's job** — distinct from the per-document insights in Step 2, which are internal to a single document. Keep the two separate in the brief: a document that is unsigned is a Step 2 observation; an account titled against what the plan intends is a Step 5 comparison.

## Step 5: Compare

Dispatch this to the `claude-for-financial-advisors:titling-compare` subagent — `Agent(claude-for-financial-advisors:titling-compare)` — handing it the Step 2 output as **intended** and the Step 4 output as **actual**. It returns the comparison table, mismatches assigned to the High/Medium/Low tiers by the definitions in Step 6, and — the part that is easy to lose doing this inline — the intended items with no matching account at all, which is how an unfunded trust shows up.

It categorizes but does not order within a tier; Step 6's ranking by dollar size and consequence is yours. It also never states that a mismatch is legally deficient or suggests a fix, which is the same boundary this skill has.

The comparison it runs, and the one to run by hand if the subagent is unavailable — for each account and position, check it against what the estate plan intends:
- **Titling match?** Is the account titled the way the plan calls for (e.g., plan says "held in the Smith Family Trust," but the account is still titled individually)?
- **Beneficiary match?** Does the named beneficiary line up with the plan's intent (e.g., an ex-spouse still listed, a minor named directly with no trust/UTMA wrapper, no beneficiary on file at all where one is expected)?

## Step 6: Flags — Prioritized

Surface what needs the advisor's attention, ranked so the highest-consequence items are seen first — this is the point of the brief:

- **High** — an unfunded trust holding (or meant to hold) a meaningful share of the estate, or a beneficiary conflict that would actively misdirect an asset (an ex-spouse still on file, a minor named with no trust/UTMA wrapper). These fail silently and are the most consequential.
- **Medium** — titling mismatches that don't misdirect an asset outright but keep the plan from functioning as intended (partial trust funding, wrong account type).
- **Low** — missing beneficiary designations where a plan default/contingent structure still applies, or minor documentation gaps.

Use judgment on dollar size and consequence within each tier — a $50k titling gap and a $5M unfunded trust are both "High" by category but the advisor should see the larger one first. If a comparison can't be made because Step 2 or Step 4 data is missing for that account/trust, say so explicitly rather than omitting the item silently.

Every flagged mismatch — whatever its tier — ends by recommending that the advisor involve the household's estate attorney before advising the client on next steps, naming the attorney when a source names one. That sentence is what this skill offers instead of a fix, and it matters most on the High flags, where saying what to do is most tempting. The standing note at the top of the brief (Step 7) does not discharge it: a reader who stops at the flag needs to see the referral on the flag.

## Step 7: Output

Write a meeting-ready brief to a markdown file first — and write it whether or not every source was reachable. A section whose source was missing carries its placeholder sentence and its pending marker in place of content; a missing source is never a reason to hold the file. (An unconfirmed identity, or two systems contradicting each other about present-day fact in Step 4, still is — those are the blocking questions above, and they are asked before the pull, not after the file.) The questions you owe the advisor about what was missing — a manual summary of the last meeting, a document to upload — go in the chat *after* the file exists, not instead of it. The order inside the file:
1. **Since We Last Met** — the Step 1 recap (what was discussed, what's been actioned)
2. **Document status** — key estate documents and whether each is signed/executed, with any document-internal observations from Step 2
3. **Prior-year tax look-back** — the Step 3 figures, each attributed to the return or report it came from, with rates shown alongside their tax dollars and income basis
4. **Comparison table** — one row per account or position, four columns with these headings in this order: **Account / Position**, **Intended per plan**, **Actual titling / beneficiary**, **Match?**. Keep the headings as written and put nothing between them — the advisor reads this table across households, and the same four columns in the same order is what makes it scannable at a glance. Anything extra, a value or a note, goes after Match?.
5. **Flags** — the Step 6 list, sorted by priority
6. A 2-3 sentence plain-English summary of the household's estate-funding picture

**The brief itself must carry the not-legal-advice language** — it leaves this session as a file that other people may read without the surrounding conversation, so the framing has to travel with it. Include a short standing note near the top, in plain English rather than legalese: this is a summary prepared to support the advisor's review, it flags items to discuss with the household's estate attorney, and it is not legal advice or a legal review of the plan. Say the same about the document observations wherever they appear — they are points to raise, not conclusions about whether the plan is sound.

The chat after the file is one short close, and it always carries two things: the format question — would they like it converted to .docx or .pdf (don't create either unless they ask; markdown is the first-pass deliverable, and the heavy formats are materially slower, so they are produced only on request) — and the questions you owe about what was missing, from this step's opening paragraph. Ask both. A close that asks for the manual recap and drops the format question is a miss the advisor won't notice, because they don't know it was owed.

## Step 8: Write Follow-Ups Back to the CRM

Draft a CRM task for each flag the brief surfaced (description, priority, suggested owner — advisor or the household's estate attorney) and show the full draft list to the advisor before creating anything.

Only create the tasks the advisor approves (all, some, or edited) in the CRM identified as the book of record in Step 1. Confirm back with a short summary of what was written.

## Out of Scope (for now)

- **No legal advice or legal conclusions.** This skill flags discrepancies for the advisor (and, where appropriate, the household's estate attorney) to evaluate — it never states that a mismatch is legally deficient or recommends a specific fix.
- **No drafting.** Never draft or amend trust language, beneficiary forms, or other estate documents.
- **No tax advice or tax strategy.** The prior-year look-back is reported context, not a plan — this skill never recommends a conversion, harvest, gifting move, or filing position, and never recomputes a figure the source system reports.
- **No account writes.** This skill never retitles an account or changes a beneficiary designation — any fix happens through the advisor's normal custodian/CRM workflow. The only write this skill performs is creating advisor-approved follow-up tasks in the CRM (Step 8).

## Important Notes

- **Every CRM write pauses for advisor approval.** Never create a follow-up task the advisor hasn't seen and confirmed.
- Never fabricate what an estate plan says, whether a document is signed, what an account's actual titling/beneficiary is, what a prior year's return reported, or what happened in a prior meeting. Missing data is "— pending [connector/documents]," not a guess.
- Recommend the advisor involve the household's estate attorney for any flagged mismatch before advising the client on next steps — Step 6 puts that sentence on each flag.
- Treat estate, beneficiary, and meeting-note data as confidential; only include what's needed for this review.
