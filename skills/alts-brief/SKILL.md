---
name: alts-brief
description: Prepare a meeting-ready alternative-investments brief for a client, household, IRA, trust, or named account before a review — pulls the alts book from iCapital or Addepar (positions, NAV vs. commitment, called and unfunded amounts, contributions/distributions/subscriptions/redemptions, lockups, and the platform's own IRR/TVPI/DPI/RVPI) and sits it beside the liquid book and IPS target from the connected portfolio system (Orion Connect, Addepar, or Envestnet), then flags cash needed for capital calls, alts % vs. target, concentration, unfunded concentration, and stale as-of data. Cites the source platform on every figure and keeps disclosures on the page. Does not recommend funds, forecast NAVs/IRRs, or simulate allocation changes. Triggers on "alts brief", "/alts-brief", "prep the [client]'s alts review", "what's going on with [client]'s PE/private investments", "how's [client] doing on their alts", or any request to review a household's alternative investment positions before a meeting.
---

# Alts Brief

Give an advisor a clear, meeting-ready read on a household's alternative investments: what's on platform, how it's tracking against their goals, and what needs attention — sitting alongside the liquid book so the advisor can see the whole household, not just the alts sleeve in isolation.

## Inputs

Required: **client/household name**. Optional: an as-of/cutoff date if the advisor wants the "recent activity" portion of the brief anchored to a specific date rather than "latest available" (default: since the last brief, or trailing 12 months if this is the first one), and a specific fund or manager to focus on if they only want part of the book.

**Identity grain:** alts are frequently held at a level *below* the household — an IRA, a trust, a single person, or one named account. Take the advisor's grain literally: "the Harrington IRA," "the Whitmore Trust," "Robert Harrington," or "the Northgate account" each mean that entity, not the whole household. Don't silently widen a named account up to its household, and don't narrow a household down to one account.

**Disambiguation rule:** confirm identity before pulling anything, the same as every other skill in this plugin — if more than one client, household, or account matches the name, show the candidates and ask which one before proceeding. Never proceed on a name match alone. When nothing is connected there is no match to confirm: take the name and grain as the advisor gave them, say so in the brief, and go.

Once identity is confirmed, **pull the alts book and the liquid/IPS picture in parallel** (Steps 1 and 2) — one missing source degrades only its own section, never the other. Before starting, tell the advisor what you're gathering and why, so a slow multi-source pull doesn't look like a silent hang. And once the client and grain are known, don't ask "should I start?" — narrate what you're doing and go; the approval gates here are the format conversion at the end and the /compliance recommendation, not permission to begin.

## Data Gathering — Steps 1 and 2 run at the same time

Steps 1 and 2 read different systems and share no state. **Dispatch them as `claude-for-financial-advisors:source-extract` subagents in a single message** — one `Agent(claude-for-financial-advisors:source-extract)` call per platform the household could be on, all in the same response rather than in sequence: iCapital and Addepar for the alts book; Orion Connect, Addepar, or Envestnet for the liquid book. You don't know which of them is connected until a read tells you, and finding out is the read's job, not a step before it: each searches for its own platform's tools by name and returns data or `NOT CONNECTED`, so a platform the household isn't on costs one short return. Never run a connectivity check from this session first — not yourself, and not through a general-purpose helper — because in a session with no platform tools that check ends with no read dispatched at all, and the Convention below is triggered by a read's `NOT CONNECTED`, never by a guess made before it. Hand each the household identity as you have it, the one system that subagent is to query (iCapital or Addepar for a Step 1 read; Orion Connect, Addepar, or Envestnet for a Step 2 read), the field schema under its step heading, and the lookback window.

Each returns a filled schema block. They do not summarize, flag, or size anything against the IPS target — Steps 3 and 4 are yours, and they need the raw figures, not a subagent's read of them.

**Identity stays with you.** `claude-for-financial-advisors:source-extract` never resolves a household: if it returns `IDENTITY MISMATCH`, show the advisor the candidates, confirm, and re-dispatch. A `NOT CONNECTED` return is the Connector Placeholder Convention case — the subagent won't offer the manual fallback, because it isn't in the conversation. You do. The same goes if more than one connected tool could try to solve the same problem for this household on either read below (not necessarily two of the same kind): ask the advisor once which is the book of record, per the Ask-Once, Then Route Convention, rather than guessing, and offer to help them save the choice using the Personalization Convention. (Step 1's iCapital/Addepar handling below is a deliberate exception to this, not an oversight.) This check isn't one-time — a source the advisor mentions on the fly mid-gathering counts too, and asking again before merging it in. And if two systems were pulled anyway and return the same household at figures apart by orders of magnitude, follow the Magnitude-Conflict Convention: name the conflict, and exclude the outlier's figures from every table and total rather than quoting them as evidence.

## Step 1: Alts Book — iCapital or Addepar

Alts can come from either platform, and some households have both, so the Step 1 reads go to both (see Data Gathering above): the one that returns data is the one you use, and both returning `NOT CONNECTED` is the Convention case at the end of this step. Don't guess at a tool-name prefix like `mcp__icapital__*`; real connector tool names use opaque, unpredictable prefixes, which is why the read searches by the platform's name.

**The read looks for the tools before it trusts the registry.** `ToolSearch` by the system's own name is the check that decides: if its tools come back, that system is connected and callable, and the read uses them. `ListConnectors` can answer "No installed connectors found" even in a session with several live, working connectors, and in some clients it renders a user-facing card rather than returning data at all. So it explains a gap, it never establishes one, and an empty result means **unknown**, never "nothing is connected". When it does return entries, read `enabledInChat`, not `connected`: `connected: true` with `enabledInChat: false` is authenticated but switched off for this chat, so tell the advisor they can enable it here rather than reporting it as unconnected; a missing or `null` `connected` is unknown, not disconnected.

- **iCapital** — the alts platform book: positions, valuations, cash flow, and iCapital-computed performance metrics.
- **Addepar** — publics and privates in one view, with lookthrough into the underlying holdings of private funds. Addepar names the funds it holds and classifies them by strategy and vintage year, and it is the path for capital calls, cash coverage, and reconciliation (see Step 2). Commitment and unfunded figures may come back empty here — take those from iCapital, or mark them "— pending [source]", rather than reporting a zero as though the household had no commitment.

If both are connected, **label each figure with the platform it came from** rather than merging them into one undifferentiated list — the advisor needs to know which system to open to act on a row, and the two are not reconciled to each other. Where the two overlap, prefer whichever platform actually returns the field: Addepar can supply a fund name and a computed paid-in percentage for positions iCapital returns only as an internal identifier. Say so when you do it ("fund name per Addepar"), and never present a figure from one platform as though it came from the other. (This is a deliberate exception to the Ask-Once, Then Route Convention: the two platforms aren't reconciled to each other, so labeling both preserves information a single pick would lose.) That exception covers which fields each platform supplies, not a genuine conflict: if the two ever disagree on a material fact for the same position (a valuation, a commitment amount, a date), labeling both isn't enough — stop and surface it to the advisor as a blocking question before the brief is finalized.

Query the connected platform(s) for the client's alternative investment positions:
- Fund/product name, manager, strategy (PE, private credit, real estate, hedge fund, etc.), and structure (drawdown commitment vs. evergreen/interval fund) — Addepar returns fund name, strategy and vintage; iCapital may return none of it, in which case take it from the advisor, an uploaded statement, or a Drive document rather than inferring it
- Commitment amount, called-to-date, and NAV, with the as-of date for each
- Unfunded commitment remaining
- Recent activity: capital calls and distributions in the lookback window (default: since the last brief, or trailing 12 months if this is the first one). When it's unclear which applies, use trailing 12 months, say so where the activity is reported, and ask in the brief's close (Step 6) — don't hold the brief for the answer.
- Lockup/liquidity terms (lockup end date, redemption windows/gates, or "committed capital, no redemption" for drawdown funds) — these usually live in the fund's own documents rather than in a platform field, so expect to source them from a Drive document, an uploaded statement, or the advisor; mark them "— pending [source]" rather than assuming a standard lockup for the structure
- Cash-flow detail per position: contributions, distributions, subscriptions, redemptions, and remaining unfunded commitment. Note which fund holds the largest share of the remaining unfunded — that's the one most likely to generate the next call.

  Classify each transaction by its **type**, not by whether its amount is positive or negative — signs are not a reliable indicator of direction here. And sort by trade date yourself before calling anything "recent": activity may arrive ordered by when it was last edited.

### Performance metrics — pass through, never recalculate (iCapital)

Report performance metrics **as the platform computed them** — IRR, TVPI, DPI, RVPI. Narrate and rank them (best and worst, early vs. mature); never recompute one, never derive one the platform didn't return, and cite the platform on the numbers.

- A metric the platform doesn't return is **"not available"** — not a guess, not a dash implying zero, and never a different metric substituted in. MOIC and TWR are commonly asked for and commonly absent; say so rather than offering TVPI in their place.
- **Returns are inception-to-date.** There is no per-quarter IRR to report, so don't compute one from period figures.
- **Check whether IRR arrives as a decimal fraction** before display, and flag an implausibly large result as reflecting an early or unusual cost basis rather than presenting it as a return the advisor can explain.
- **Say whether figures are net or gross**, and don't mix the two in one comparison.
- **Report commitment, called, and unfunded as three separate figures.** Don't derive a "% funded" from them or reconcile them against each other — they're reported independently and aren't meant to tie, and dividing them can produce a figure over 100%. If a platform reports a paid-in percentage itself, use that and cite it; otherwise leave it out rather than calculating one. Same for two different value figures on one position: show one, say which.
- **Lifetime figures need lifetime windows.** On some platforms, columns like called-to-date, paid-in, and total commitment are computed over the query window — a trailing-12-month pull can understate called-to-date or return commitments as zero. Pull cumulative figures with an inception-wide window, prefer a field that stores the lifetime total over one computed across a window, and when two sources both carry the figure, cross-check them and surface a disagreement rather than picking one silently. State the window behind any windowed figure.

> **Connector Placeholder Convention:** if neither iCapital nor Addepar shows as connected, say: *"This is where I'd make a call out to [platform] to pull [client]'s alts positions once that connector is available."* Then offer the fallback: the advisor can upload a statement/export (PDF/CSV) or paste position details. Don't wait for it — write the brief now (Step 6): the position table carries one row of "— pending [source]" cells in place of the positions you couldn't pull (never an invented fund or figure), each flag is named with what it's missing, and the questions you owe the advisor sit in the close *after* the file exists. A brief that is all pending rows is still the deliverable; a question with no brief behind it is a stall. Anything the advisor provides afterwards is a fresh pass over the same brief.

## Step 2: Liquid Book & IPS Target — Orion Connect, Addepar, or Envestnet

Pull the rest of the household picture so alts can be sized in context:
- Total liquid/investable assets and current cash balance
- IPS or model target allocation to alternatives (%) — **check the connected platform for it first.** If a platform exposes target allocations or IPS bands, use them and cite the platform. Not every platform surfaces targets even when it holds them, so if nothing comes back, ask the advisor for the target or accept a pasted IPS — and say which of the two the number came from, since a target the advisor supplied from memory carries different weight than one read from the system of record.
- Household AUM for sizing alts against the total. If the connected platform reports allocation or exposure percentages itself, use those and cite them rather than summing an alts NAV onto a liquid balance by hand — the two may be valued as of different dates, and a platform-computed percentage is the defensible number. If you do have to combine figures, say which values and as-of dates you combined, and present the result as approximate.

Follow the same **Connector Placeholder Convention** as Step 1 — if none of the three is available, fail gracefully: say so, then offer the manual fallback (paste, upload, or skip) and continue to the brief with the liquid/IPS figures marked "— pending [source]".

**When Addepar is connected**, it answers the capital-call and cash questions directly, and these feed the flags in Step 4:
- **Upcoming capital activity** — future-dated calls by fund and date, returned alongside the household's cash balances so coverage is assessed against real cash rather than an estimate. Forward-looking only: it does not return calls that already happened.
- **Private-fund cash movements** — recent calls, contributions and distributions matched against actual cash-account activity, which answers "did that call clear?" rather than only "was it announced?" Use the platform's own matches; report an unmatched item as unmatched rather than pairing it yourself.

A call that is upcoming has not cleared — don't describe it as funded, and don't net it against cash beyond the comparison the platform already provides.

**Documents, if Drive is connected.** `ToolSearch` for Google Drive's tools by name — the one lookup you make yourself, for documents only; the platform reads check their own connectivity — and if they come back, offer to search it for the household's fund documents — capital call notices, subscription agreements, K-1s, quarterly letters — which often carry terms and dates the platforms don't expose. This is additive: if Drive isn't connected, or a search finds nothing, say so in one line and continue. Never block or delay the brief on document search.

## Step 3: Assemble the Position Table

| Fund | Manager/Strategy | Commitment | Called-to-Date | NAV | Unfunded | Recent Activity | Lockup/Liquidity | As-of Date |
|------|------------------|-----------|-----------------|-----|----------|-----------------|-------------------|-----------|

One row per position. If a field is unavailable for a given fund, mark that cell "— pending [source]" rather than guessing — [source] names what would supply it (iCapital, Addepar, a fund document, the advisor). That is the one sentinel this skill uses, everywhere a value is missing.

Where performance metrics are available, put them in a **second table** rather than widening this one past readability — one row per fund, columns for the metrics that fund actually has (IRR, TVPI, DPI, RVPI), with "not available" in any cell the source didn't return. Say in a line above it that these are the platform's own computed figures, as of the report date, and whether they are net or gross.

**Naming rows:** iCapital may identify a position only by an internal product id, without fund name, manager, or strategy. Don't infer any of them — from the id, a prior brief, or knowledge of the manager. If Addepar is connected, use the fund name, strategy, and vintage year it returns and attribute them to Addepar. Otherwise label the row with the identifier given, say the name isn't available, and offer to take it from the advisor or an uploaded statement; a wrong fund name on a meeting document is worse than a missing one.

## Step 4: Flags

Surface what actually needs the advisor's attention — this is the point of the brief, not just the data dump above:

- **Cash needed for upcoming calls**: any known/expected near-term capital calls vs. the household's current cash balance. Flag if calls would draw cash below a reasonable buffer (if the advisor hasn't given a threshold, flag against a stated assumption and ask for theirs in the close).
- **Alts % vs. target**: alts as a share of household AUM against the IPS target, flagging meaningful drift either direction. Say which basis you used — NAV alone, or NAV plus unfunded — and unless the advisor has already said which the firm uses, ask in the brief's close (Step 6) whether the firm counts unfunded commitment toward the target: it materially changes the answer. Prefer a platform-reported allocation percentage over one you compute, and note where the target itself came from (see Step 2).
- **Concentration**: any single manager, strategy, or vintage year that's an outsized share of the alts book.
- **Unfunded concentration**: which fund holds the largest share of remaining unfunded commitment — the likeliest source of the next call, and the one to watch against cash.
- **Stale as-of data**: private-fund NAVs typically lag a full quarter. Flag any position whose as-of date is older than ~100 days so the advisor knows the number isn't current, not that nothing changed.

If a flag can't be evaluated because its input is missing — no IPS target, no cash balance, no as-of date — **name the flag and say what's missing**, rather than dropping it from the list. A flag that silently disappears reads as "nothing to worry about here," which is not the same as "this couldn't be checked."

## Step 5: Identify the Platform's Disclosures & Disclaimers

Before assembling the final output, gather the disclosures that have to travel with this data — platform-level disclaimers, fund-specific disclosures, and as-of/data-lag notices.

Take them from whatever the source actually provides: text returned alongside the position data, or the disclosure pages of an uploaded statement or export. **If the connector returns no disclosure text, say so and ask the advisor for their firm's standard alts disclosure language** — never draft disclosure or disclaimer text yourself, and never carry forward remembered wording from another brief. These are reproduced verbatim in Step 6, so identify them here rather than reconstructing them afterward.

## Step 6: Output

Write the brief to a markdown file first:
- A meeting-ready brief: the position table first, then the flags — the flags cite rows in the table, so the table belongs above them. Give the flags their own heading so an advisor short on time can go straight to it.
- Cite the source platform for the alts data — iCapital, Addepar, or both, naming which one each figure came from when the book spans two — and reproduce the disclosures/disclaimers identified in Step 5 on the page as provided; don't paraphrase or drop them.
- Close with a 2-3 sentence plain-English summary of the household's alts picture, including how it sits against the IPS target.
- Then a short **For the advisor** section naming every question the brief was written around rather than waiting on — each asked as a question the advisor can answer in a word, not as a default you've noted ("I'll use trailing 12 months unless you say otherwise" states a choice; "Is trailing 12 months the right window, or is there a prior brief to measure from?" asks one): which lookback window to use if you defaulted to trailing 12 months (Step 1), whether the firm counts unfunded commitment toward the target (Step 4), their cash-buffer threshold if you assumed one (Step 4), and anything a "— pending [source]" cell is waiting for. These are asked here, in the file, and again in the chat — never instead of the file.
- If the advisor asked for the whole household picture — or the request reads that way, not just the alts sleeve — say in the brief that this covers alts only and point at **/portfolio-rebalance-review** for the full multi-asset drift review and rebalance prep.
- If any part of this brief will be shared with the client directly, recommend running it through **/compliance** first — this skill produces an advisor-facing prep document, not pre-cleared client communication.

Then ask the advisor whether they'd like it converted to .docx or .pdf — don't create either format unless they ask. Markdown is the first-pass deliverable; heavy formats are materially slower, so produce them only on request.

## Out of Scope (for now)

- **No fund recommendations.** This skill reports what's on platform and flags what needs attention — it does not suggest adding, dropping, or replacing a fund or manager.
- **No NAV/IRR forecasting or projections.** Report only actuals as reported by the source platform. Never estimate future NAV, project IRR, or otherwise forward-look.
- **No allocation simulations.** Questions like "what if we added 10% private equity?" are out of scope — say so, and note that scenario modeling may fit better as a future extension or a different skill.
- **No recalculated or hypothetical performance.** "Recalculate IRR assuming a higher exit multiple," "what will NAV be next quarter," and similar are out of scope — the metrics are the platform's computed figures, and re-deriving them under different assumptions produces a number no system will stand behind. Say so plainly and offer what the record does show.
- **No transactions.** This skill never initiates a subscription, capital call funding, or redemption — those go through the advisor's normal iCapital workflow.

## Important Notes

- Never fabricate commitment amounts, NAVs, call/distribution history, fund names, or performance metrics. Missing data is "— pending [source]," not a guess.
- **A zero is not the same as "not reported."** If a figure comes back empty or zero from a platform that doesn't carry it, say it isn't available from that source — never present it as a real zero. "$0 unfunded" tells an advisor the commitment is fully drawn, which is a different and possibly wrong statement.
- **An empty result is an answer, not a gap to fill.** If no positions come back, say "no alts positions found for [client]" — never assemble an illustrative table in the space. A zero result may mean the client holds nothing or that the advisor isn't entitled to see that book; don't assert which, and suggest they confirm entitlements if they expected positions.
- This skill covers the alts sleeve specifically. For a full multi-asset-class drift review and rebalance prep, that's **portfolio-rebalance-review** (`/portfolio-rebalance-review`) — Step 6 says to point at it in the brief whenever the advisor wants the whole household picture rather than just alts.
- Treat client data as confidential; only include what's needed for this review.
