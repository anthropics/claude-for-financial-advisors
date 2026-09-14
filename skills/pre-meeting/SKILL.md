---
name: pre-meeting
description: Prepare a complete client review meeting prep document for a financial advisor, for whatever timeframe the review covers (quarterly, annual, ad hoc). Given a household name or email and a timeframe, pulls CRM context (Salesforce/Redtail/Wealthbox), portfolio data (Orion/Addepar/Envestnet-Tamarac), estate & tax status (Wealth.com), meeting notes (Zocks or CRM), recent correspondence (Gmail/Outlook), and the household's plan snapshot (MoneyGuide), then assembles a meeting-ready prep doc with agenda, talking points, and action items. Triggers on "pre meeting", "/pre-meeting", "prep for [client]", "quarterly review prep", "get me ready for my meeting with [client]", or "client meeting tomorrow".
---

# Pre Meeting

Build a financial review prep document an advisor can read in 10 minutes and walk into the meeting confident.

## Inputs

Required: **household name and/or email address**, and the **timeframe** the review covers (quarterly, annual, or ad hoc). If either is missing, ask for both before doing anything else — default the timeframe to quarterly if the advisor doesn't care which.

Optional (ask only if ambiguous): meeting date/time, anything specific the advisor wants to cover.

## Data Gathering

Follow the **Connector Placeholder Convention**: before assuming a system below isn't connected, check `ListConnectors` (load via `ToolSearch` if it isn't already available) — don't guess at a tool-name prefix like `mcp__orion__*`, since real connector tool names use opaque prefixes. If it's connected, use `ToolSearch` (by system name) to find its actual tools. If it's not connected, say "This is where I'd make a call out to [system] to pull [data] once that connector is built," then offer the manual fallback (paste, upload an export, or skip) and keep going. Apply this to every source below, not just the first one that's missing — and if more than one connected tool could try to solve the same problem for this household (not necessarily two of the same kind), ask the advisor once which is the book of record, per the Ask-Once, Then Route Convention, rather than guessing, and offer to help them save the choice using the Personalization Convention. This gathering pass touches several systems in sequence, so the check isn't a one-time thing at the top: re-run it whenever a later source could answer something an earlier one already did, and if two sources you did pull disagree on a material fact, stop and surface it to the advisor as a blocking question before the brief is finalized rather than presenting both. If the disagreement is by orders of magnitude for the same household, follow the Magnitude-Conflict Convention: name the conflict, and exclude the outlier's figures from every table and total rather than quoting them as evidence.

**Look for the tools before you trust the registry.** `ToolSearch` by the system's own name is the check that decides: if its tools come back, that system is connected and callable — use them. `ListConnectors` can answer "No installed connectors found" even in a session with several live, working connectors, and in some clients it renders a user-facing card rather than returning data at all. So it explains a gap, it never establishes one, and an empty result means **unknown**, never "nothing is connected". When it does return entries, read `enabledInChat`, not `connected`: `connected: true` with `enabledInChat: false` is authenticated but switched off for this chat, so tell the advisor they can enable it here rather than reporting it as unconnected; a missing or `null` `connected` is unknown, not disconnected.

Once identity is confirmed, **pull from all connected sources in parallel** — one slow or missing source degrades only its own section of the doc, not the rest. Before starting this multi-source pull, tell the advisor what you're about to gather and why, so it doesn't look like a silent hang. And once the household and timeframe are known, don't ask "should I start?" — narrate what you're doing and go; approval gates in this skill apply only to format conversions and length changes at the end, not to starting the work.

**Dispatch the parallel pull as one `claude-for-financial-advisors:source-extract` subagent per source, all in a single message** — the Agent tool, `Agent(claude-for-financial-advisors:source-extract)`, one call per source in the same response so they run concurrently rather than one after another. Each dispatch hands over the household identity as you have it, the one system that subagent is to query, the field schema for that source (the bullet list under its heading below), and the window the timeframe implies — or, where a source reads current state and has no meaningful review window, a plain statement of that instead of a window. Each returns a filled schema block. Two reasons this is a subagent rather than a tool call you make yourself: the raw payloads — email threads, full holdings tables, an estate look-back — stay out of your context so you assemble the doc from six compact blocks instead of six raw dumps, and the extraction work for all six happens at once rather than sequentially in your own reasoning.

**You still own identity.** The subagents are instructed never to resolve it: a `claude-for-financial-advisors:source-extract` that finds several candidates or a mismatch returns `IDENTITY MISMATCH` with what it saw and stops. Handle that the way the disambiguation rule below says — show the advisor the candidates, confirm, re-dispatch. Verify the returned blocks agree on the household before assembling anything from them. Fanning the reads out does not move that responsibility; it stays with you.

A subagent that returns `NOT CONNECTED` is the Connector Placeholder Convention case. It won't offer the manual fallback itself — it isn't in the conversation with the advisor. You make that offer.

### 1. CRM — Salesforce (or Redtail/Wealthbox)
**Disambiguation rule:** query the CRM by name AND confirm identity before pulling anything. If multiple contacts match the name, show the advisor the candidates (name + masked email + account) and ask which household is theirs — common names will return coincidental matches, and pulling the wrong client's data into a prep doc is a privacy incident. Never proceed on a name match alone. When one candidate is a clearly better match (exact email match, most recent activity), pre-select it as the default choice in that prompt — the advisor still has to confirm it, never skip that step outright.

A recognizable or public-figure name that matches a connected CRM record is a household like any other — treat it accordingly and do not refuse, hedge, or ask for extra justification because the name is well-known. This is guidance for how you proceed, not content for the document: the prep doc itself should read exactly like any other household's, with no note, disclaimer, or aside about the name being recognizable.

Pull the household record:
- Notes/insight-type fields (freeform notes, comments) from the CRM record
- Household members, key dates (birthdays, RMD age, Medicare eligibility at 65, Social Security decision points)
- Open tasks/action items (structured task list) — **at both grains: the household record's own items and each person contact's.** A CRM household is often itself a record with its own id (Wealthbox's is a contact), and household-linked tasks are invisible to contact-only lookups
- Service history — recent requests, complaints, referrals given

### 2. Portfolio — Orion / Addepar / Envestnet-Tamarac
Pull household and per-account data: market values, performance vs. benchmark, allocation vs. target and drift, realized/unrealized gains, income and fees, cash balances and large flows for the review period.

**Orion Connect specifics:** its five newly-shipped tools (`oc_search_accounts`, `oc_get_current_value`, `oc_list_household_accounts`, `oc_get_household_address`, `oc_lookup_account_owner`) are read-only identity/AUM lookups only — household/account identity, current value, mailing address. They do **not** return holdings, performance vs. benchmark, allocation/drift, gains, fees, or flows; don't infer any of those from a current-value number. Use these five for household/account identity and AUM with no confirmation needed — they're fast reads, not report runs.

For holdings, performance, drift, gains, fees, or flows, fall back to Orion's existing report chain: `oc_search_households` → `oc_list_reports` → `oc_get_report_parameters` → `oc_generate_report`. That chain runs a job in Orion Connect rather than reading one, so confirm with the advisor before generating a report, say roughly what it will contain, and show progress/status copy while it runs rather than a silent wait. The generate step returns a **link to the finished report**, not the report's contents — hand the advisor the link, say what it is, and say it **expires after 24 hours** (a unique per-run URL); don't fetch it to extract figures from, and don't treat having produced a link as having read the data. If the advisor comes back to an expired link, regenerate the report (with the same confirm-first step) rather than troubleshooting the URL. If the report chain isn't live or doesn't return data for the period, degrade explicitly — state the AUM figure you do have and mark performance/drift/fees/flows "— pending Orion report," rather than presenting AUM as if it were the full portfolio picture.

Where the connector exposes reconciliation status (e.g. Envestnet/Tamarac's `getAccount` returning `reconciliationStatus` / `lastReconciliationDate`), pull and surface it — flag stale or unreconciled data before quoting balances built on it.

**Held-away alts check (lightweight):** if iCapital or Addepar is connected and isn't already this doc's portfolio source (Addepar-as-source already covers alts via lookthrough), run one cheap exposure check there — iCapital's `query_positions` by `account_name` resolves server-side but returns null account/fund names, so report a name-matched position count only, never a self-summed dollar total; Addepar's `search_entities` + one `get_exposure` by asset class returns platform-computed value/% per bucket — quote those rows as returned, don't re-aggregate them into an "alts total" of your own. Positions found → one line in the template, cited to the platform, pointing to `/alts-brief` for detail (no capital calls, lockups, or performance here). Alts are optional, so absence isn't a gap: no alts platform connected, or zero matches → **omit the line entirely, no "pending"**. The one line written without positions: platform connected but the check failed → "— alts coverage not checked; run /alts-brief if this household holds alternative investments".

### 3. Estate & Tax — Wealth.com
Pull an estate/tax look-back report: what changed since the last review (documents, titling, beneficiaries) and year-specific tax constants relevant to the household (contribution limits, RMD factors, and similar figures). Always cite these as reported by Wealth.com for the applicable tax year — never recompute or estimate a constant Claude wasn't given.

### 4. Meeting Notes — Zocks (or CRM notes as fallback)
There is no Zocks "prep" endpoint — Zocks' MCP only exposes atomized reads (contacts, meetings, per-meeting AI results, insights), not the finished brief its own product UI generates. Build the picture yourself:

- **Fan out, then dedupe.** Resolve the household's contacts, list their past meetings, then pull AI results/summaries per meeting. Per-meeting extractions repeat near-duplicate household/financial detail across meetings, and a long-standing household can return dozens of results and summaries — collapse those into one current picture instead of listing each copy.
- **Precedence, in order:** prefer Zocks' AI results/summaries over the raw transcript; use the transcript only when you need the client's own words verbatim (e.g., a direct quote), not as the default source; if Zocks isn't connected, or is connected but returns no contact/meetings for this household, fall back to whatever the CRM has logged from prior meetings — the common case is the latter, not a missing connector; only fall back further to a manual paste/upload if neither is available.
- **Default to summary-first.** Don't dump every `ai_results_*` payload for every meeting — pull summaries for the last 1-2 meetings by default. Treat a full extraction fan-out across more meetings as a deeper, slower pass and ask the advisor before spending that extra latency.
- **Household is a relationship, not a record.** Zocks has no household object — it's "the people who usually meet together." Treat the CRM's household record as the book of record and use Zocks only to color it; reconcile what a meeting extraction says against the CRM rather than treating any single meeting's extraction as authoritative.
- **One item, one row.** When a Zocks action item and a CRM task describe the same underlying thing (e.g. a Roth conversion analysis mentioned in a meeting and tracked as a CRM task), merge them into a single line using the CRM's status — never list it twice just because it showed up unresolved in Zocks and resolved in the CRM.
- **Cite the meeting**, by date (and meeting ID if the tool returns one), for anything sourced from Zocks — and never attribute a fact to a CRM or planning source that the Zocks MCP itself didn't return — its tools expose Zocks's own meeting data only, not the systems Zocks integrates with.
- **Layer firm-level Insights separately**, when the connector has an insights tool — firm-wide trending topics, sentiment, referral opportunities. That's distinct from "what changed for this household" and belongs in Talking Points (see below), not folded into Since We Last Met. A household-scoped insight or sentiment result from the same tool, tied to this client specifically, is a Relationship Signal, not a firm trend — route it there instead.
- **Relationship signals — source from insights and notes, not task fields.** Populate the Relationship Signals section from Zocks' AI results/summaries and insights (`insights_query_insights`, when the connector exposes it) first, then CRM freeform notes fields (meeting notes, "notes"/"comments" on the contact or household record) — never from CRM structured task/action-item fields. Task fields track what's due, not what's true about the relationship, and are used inconsistently across RIAs; a CRM with an empty task list can still have a rich notes field. This section should still populate from whichever of Zocks or CRM notes is available — it does not wait on both, and it never blocks on the task list being empty or unmaintained. If both Zocks and CRM notes are unavailable, mark the section "— pending Zocks / CRM connector" per the Connector Placeholder Convention rather than omitting it. Cite every signal the same way as above: date (and meeting ID if returned) for anything Zocks-sourced, the field name (e.g. "per CRM notes field") for anything CRM-sourced. A signal with no source behind it doesn't go in the doc.
- **This skill still writes the Proposed Agenda itself** from the gathered data — never call a Zocks-generated agenda/prep or its Default Prompts a substitute for that section.

### 5. Email — Gmail / Outlook (available today for most users)
Search correspondence with the household's email address over a window that follows the chosen timeframe — roughly the last 3-4 months for a quarterly review, ~12 months for an annual review, or whatever's appropriate for an ad hoc one. Extract:
- Open requests or questions the client is waiting on
- Commitments the advisor made ("I'll get you that Roth analysis")
- Life events mentioned (job change, new grandchild, home purchase, health, inheritance)
- Tone/sentiment — anything suggesting concern or dissatisfaction

### 6. Plan Snapshot — MoneyGuide
**This source has no review window.** It reads the household's plan as it stands right now, so the dispatch hands that fact over in the window's place — never a blank, and never the review timeframe, which would imply a lookback this source doesn't have.

**Identity stays inside this pull**, the same way every other source resolves its own system: the subagent searches MoneyGuide by surname — households are stored surname first and the search matches the start of the name — and returns `IDENTITY MISMATCH` with what it saw if several match or none do.

Pull:
- Net worth — total, assets, liabilities
- Probability of success — only when the household has a plan
- Each goal — name, initial expense, start year — only when the household has a plan
- The plan's last-updated date, which dates the plan without saying what changed in it

Quote every figure exactly as MoneyGuide returns it: the output is prose and already rounded, so never recompute, re-derive or re-round it, and never fill a gap with a figure of your own.

**No plan on file** is a normal, informative answer from a working connector, not a degraded read: net worth still comes back and still goes in the doc, while probability of success and goals are marked not applicable for this household rather than left blank. An absent probability of success is never rendered as zero and never dropped silently — and never marked pending either, which would send the advisor chasing a connector problem instead of reading the connector's honest answer.

## Assemble the Prep Doc

Use `templates/pre-meeting-template.md` in this skill's folder as the document skeleton. Fill every section; where data was unavailable, mark it "— pending [connector]" rather than leaving blanks silently.

The doc's sections, in order:
1. **Client Snapshot** — household, AUM, tenure, risk profile
2. **Relationship Signals** — client concerns/sentiment, life events, upcoming personal dates (birthdays, anniversaries, milestones), proactive talking points — each signal cited to its source
3. **Since We Last Met** — open action items from last meeting with assignee and status (mark "unknown — pending CRM" unless the CRM's own task list confirms done; never infer "done" from a meeting-notes source that doesn't report completion), notable correspondence
4. **Portfolio Review** — performance vs. benchmark table, top contributors/detractors, allocation drift table, cash position
5. **Planning Opportunities** — proactive items: rebalancing, tax-loss harvesting, Roth conversion window, RMD/QCD planning, 529 funding, beneficiary review, insurance gaps
6. **Proposed Agenda** — a 30-45 minute agenda with time boxes
7. **Talking Points & Anticipated Questions** — 3-5 talking points in plain English (portfolio/market-facing — personal talking points live in Relationship Signals); firm-level Insights/trends when available; likely client questions (especially about underperformance or markets) with suggested framing
8. **Action Items Draft** — pre-drafted next steps to confirm in the meeting

This skill never computes or invents a plan probability, funded status, or Monte Carlo result — MoneyGuide's own net worth, probability of success and goals go under Planning Opportunities as color, quoted exactly as they came back, never as a plan section of their own.

## Output

- Write the prep doc as a markdown file first. One page-ish summary up front, supporting tables behind it.
- Keep language client-friendly — the advisor may screen-share parts of it.
- After writing the markdown file, ask the advisor whether they'd like it converted to .docx or .pdf (via the docx skill) — don't create either format unless they ask. Also ask whether they'd like the doc longer or shorter, and revise if so.
- End the chat response with 2-3 sentence "if you only read one thing" summary of the client's situation.

## Out of Scope (for now)

- **No trade execution.** This skill never places, stages, or confirms a trade.
- **No sending to the client.** The prep doc is for the advisor; sharing it is the advisor's call, through their normal workflow.
- **No inventing plan probabilities or performance.** See "Assemble the Prep Doc" above.
- **No legal advice.** Estate/tax findings are flagged for the advisor (and the household's attorney/CPA where relevant) to evaluate, not acted on here.

## Important Notes

- **Underperformance goes first, not buried.** If the portfolio trailed its benchmark, put it in talking points with an honest explanation — advisors lose trust by dodging it.
- Never fabricate performance numbers, plan probabilities, or client details. If a source is unavailable and the advisor can't provide data, show the section as pending.
- Flag compliance-sensitive content: if the prep doc will be shared with the client, recommend running client-facing excerpts through **/compliance**.
- Watch for milestone triggers in the data: turning 50 (catch-up contributions), 59½ (penalty-free withdrawals), 62-70 (Social Security), 65 (Medicare), 73 (RMDs).
- Treat all client data as confidential; only include what's needed for this meeting.
