---
name: portfolio-rebalance-review
description: Run a portfolio review for a client or household — performance pulled from the connected portfolio platform (Orion Connect, Addepar or Envestnet/Tamarac), allocation drift analysis vs. IPS or model portfolio, model-portfolio mapping, rebalance preparation with tax-aware rebalance opportunities to consider, and a rebalance rationale memo for the compliance file. Triggers on "portfolio review", "/portfolio-rebalance-review", "drift analysis", "rebalance [client]", "how is [client]'s portfolio positioned", "rebalance rationale", or "map [client] to a model".
---

# Portfolio Rebalance Review

Review a client portfolio end to end: performance → drift → analytics → model mapping → rebalance prep → rebalance rationale memo, with a current-against-proposed comparison when the advisor asks for one.

## Inputs

Required: **client/household name**. Optional: which accounts to include, the model portfolio or IPS targets (if not on file), and whether the advisor wants analysis only or a full rebalance prep. Once you have the household, say up front that the review can take a few minutes once portfolio data starts flowing across steps, rather than asking whether to proceed — invoking the skill is already the go-ahead to run the review. Say in the same breath which systems you'll be pulling from and **which relevant ones aren't connected** — the ones you looked for and found no tools for (Step 1), never the ones the advisor says aren't — with a one-line offer to connect one. It never blocks — make the offer and carry on with the best pass the connected systems allow. If the advisor has already said they don't have a given provider, don't raise it again.

**Take the advisor's grain literally.** A household and a single account are both valid subjects, and which one an advisor works at is set by their firm — whether compliance and risk tolerance are judged at household or account level varies. "The Harrington IRA" means that account; "the Harringtons" means the household. Don't silently widen a named account to its household, or narrow a household to one account. Whichever they named is what the drift denominator, the flags and the memo are about.

**When the ask is meeting-shaped, point at /pre-meeting.** An advisor getting ready for a client meeting — an agenda, talking points, what's changed since they last met — wants the prep doc, not this review. Cover what they asked of the portfolio here, and offer /pre-meeting for the meeting itself rather than building an agenda or talking points into this review.

## Step 1: Pull Performance & Holdings

**Disambiguation rule:** confirm which household before pulling anything, the same as every other skill in this plugin — if a name search returns more than one match, show the advisor the candidates and ask which one before proceeding. Never proceed on a name match alone.

Query the household's connected portfolio platform for:
- Accounts with registrations (taxable, IRA, Roth, 401k, trust) and market values
- Holdings with quantity, market value, cost basis, and unrealized gain/loss
- Performance: QTD/YTD/1yr/3yr vs. assigned benchmark, net of fees. A platform's benchmark fields can return a plausible return for an asset-class index rather than the household's assigned benchmark — a real figure answering a different question. Present a benchmark return as the household's only when the source ties it to this household or account — a performance report that states the household's benchmark does, a bare benchmark field returned for a position or an index does not; otherwise name what it is measured against, or mark the comparison "— pending assigned benchmark".
- Assigned model/target allocation and cash balances
- Recent flows (contributions, distributions, RMD status)

**Which system to query:** this data can live in the portfolio platform — Orion Connect, Addepar, or Envestnet/Tamarac — depending on the firm. Look for each platform's tools by its own name, the way the next paragraph says, to see which are actually connected — don't guess at tool-name prefixes — and read the rest of this step for what a gap means and what to do when more than one comes back.

**Look for the tools before you trust the registry, or the advisor.** Your own tools are the check that decides: a system whose tools are in your tool list, or come back from a `ToolSearch` by the system's own name, is connected and callable — use them. Where the session offers `ToolSearch`, a system's tools can be held back until it loads them, so search by name for every system this review reads whose tools aren't already in your list, the analytics platform's included (Step 2b); where it offers none, nothing is held back, and your tool list is every tool you have. An advisor saying a system isn't connected, or that none is, doesn't decide it either, since advisors often don't remember what they connected: look for every system this way before you name any of them as not connected, and when you do, say it as what you found, not as what they told you. `ListConnectors` can answer "No installed connectors found" even in a session with several live, working connectors, and in some clients it renders a user-facing card rather than returning data at all. So it explains a gap, it never establishes one, and an empty result means **unknown**, never "nothing is connected". When it does return entries, read `enabledInChat`, not `connected`: `connected: true` with `enabledInChat: false` is authenticated but switched off for this chat, so tell the advisor they can enable it here rather than reporting it as unconnected; a missing or `null` `connected` is unknown, not disconnected.
- **If none is connected**: follow the Connector Placeholder Convention — say *"This is where I'd make a call out to [system] to pull [holdings/performance], but it isn't connected in this session,"* then offer the fallback. Before asking for a manual upload, offer to search the firm's document store (for example @${CLAUDE_PLUGIN_ROOT}/partials/connectors/storage-examples.md) for an existing export or statement. If nothing turns up there either, ask the advisor to upload an export (CSV/PDF), a custodian statement, or paste holdings. Continue with whatever's provided, and mark anything still missing "— pending [connector]" rather than leaving it blank — a missing source degrades only the section that depends on it, not the whole review.
- **If a portfolio system is connected but none that is connected can return holdings** — its tools identify the household and generate reports, but return no structured holdings — **ask for the holdings and stop.** One turn: name the shape this review can use (each holding with quantity, market value and cost basis, and each account's registration), offer the same path as the branch above — a search of the firm's document store for an existing export or statement, then an upload (a CSV or PDF export, a custodian statement) or a paste — and write no review until they arrive. This is the one exception, among missing sources, to carrying on with the best pass the connected systems allow: a rebalance review with no holdings has nothing to review — drift, wash-sale coordination, the draft plan and the memo all depend on them — so a review written first would be the IPS restated beside a list of what's still owed.
- **If more than one connected tool could supply this household's holdings and performance** — most often two portfolio platforms — follow the Ask-Once, Then Route Convention in its fixed order: **identify the household in one of them** — its household or entity search, nothing that returns holdings or performance — **then ask the advisor once which is the book of record for this household, then pull only from that one** for the rest of this review. Never search the second platform before they answer, never guess, and never merge both. One move at a time:
  1. **Before the search, say what comes next** — which platform you're looking for the household in, and that you'll ask which one is their book of record as soon as that search comes back.
  2. **Run one household or entity search, in that platform only.**
  3. **When it comes back, ask — whatever it returned.** A match, several candidates (they go in that same question), nothing at all, or an error you can't read all lead to the same next move: the question to the advisor, before any other call to either platform. A name that doesn't resolve in the first platform is not a reason to search the second: ask then, and put the name in the same question ("I couldn't find a Lindqvist household in Addepar — is Orion their book of record, or do they go by another name?"). "It isn't in this one, so let me check the other" is exactly the search this rule forbids — which platform holds the household is the advisor's to tell you, not yours to find out by searching.
  4. **When they answer, offer the paste before anything else.** The reply that takes their answer opens with it, before your first call to the platform they named — even when their answer also hands you a different name to search:

     > Want me to remember that for [household] so I don't ask again? Paste this into **Settings → General → Instructions for Claude**:
     >
     > For the [household] household, treat [system] as the book of record for portfolio holdings and performance. Don't ask me to choose again for this household.

     Claude can't write those settings, so never offer to save it yourself or say you have. Then carry on with the review, in the platform they named only.

  This check runs for the rest of the review, not just once up front — if the advisor brings up another source mid-review, ask before pulling from it too. And if a second system's figures get pulled anyway and disagree with the routed one on a material fact (a balance, a drift percentage), that's a blocking question for the advisor, not a note in the output.

**Held-away accounts:** ask the advisor whether the household has accounts held away from this system (a spouse's 401k, assets at another custodian) relevant to wash-sale coordination or the overall allocation picture. If yes, request those holdings manually — there's no connector for outside accounts.

**Addepar specifics:** query the portfolio data with lookthrough turned off for drift purposes. Lookthrough decomposition (fund → underlying constituents) is a separate risk-review lens — an IPS names sleeves at the fund-commitment level, and lookthrough silently returns an incompatible set of buckets with no warning. Group by `direct_owner`, not `account` — the latter is not a valid grouping key and errors. For tax-lot detail, pass a tax-lot value in `groupings` only if the live tool schema actually documents one — cost basis and lot-level fields are not confirmed to exist on every Addepar deployment. If the schema doesn't support it or the returned columns don't include cost basis, say so explicitly rather than presenting fabricated lot data; this feeds the same "Completeness" gate below.

**A portfolio system that runs reports but returns no holdings** — its tools identify the household, list reports and run them, but return no structured holdings, cost basis or tax lots — is worked with the way @${CLAUDE_PLUGIN_ROOT}/partials/rules/report-only-systems.md says. If it is the only connected system, Step 1's ask-and-stop above is the answer.

**Data-sanity gate — run before Step 2, don't skip.** Dispatch this to the `claude-for-financial-advisors:holdings-sanity` subagent — `Agent(claude-for-financial-advisors:holdings-sanity)`, in the foreground (`run_in_background: false`) so what it returns arrives as the call's result inside this turn — rather than working through it inline: hand it the holdings data, the reported portfolio/household value, and the sleeve structure from the IPS or model if one is on file. It returns a flag table with a `Contaminates` column naming which downstream figure each problem feeds, plus an explicit list of the checks that passed. If the dispatch errors, is refused because the concurrent-subagent limit is reached, or hasn't come back by the time you would otherwise end your turn, run the seven checks below yourself — never end your turn, or start Step 2, with the gate outstanding.

Act on what comes back before touching Step 2. A `BLOCKING` verdict means a flag would corrupt the drift table itself — resolve it with the advisor first. `FLAGS` means the review continues with the caveats carried into the output. The subagent never repairs data, drops a position, or estimates a missing field, so anything it raises is still yours to decide about.

The seven checks it runs, which are also the checks to run by hand if the subagent is unavailable:
- **Temporal:** flag any position whose valuation date precedes its first funding/contribution transaction.
- **Magnitude:** flag any position whose gain looks implausible for the asset type and time held (e.g. a private fund up several multiples within months of being funded) and confirm the mark with the advisor before using it.
- **Aggregation:** flag any sleeve or household-level return that's materially inconsistent with the sum of its parts (one position driving the whole household return while everything else is flat/negative is a signal, not a fact).
- **Reconciliation:** reconcile the holdings total against the reported portfolio value.
- **Completeness:** flag any position missing a field a later step depends on — cost basis, quantity, or an asset-class/sleeve tag — rather than silently excluding it from drift or tax calculations.
- **Staleness:** flag when a mark's as-of date is materially older than the rest of the portfolio it's being compared against (common for private/illiquid holdings priced less often than public ones) — note the mismatch rather than treating all "Current %" figures as equally current.
- **Duplication:** flag if the same underlying exposure may be represented more than once — e.g. a fund-level position and its lookthrough decomposition both pulled into the same table, or a held-away account that turns out to already be visible through the primary connector.
- If anything trips, say so explicitly and state which downstream conclusions (drift %, performance %) depend on the suspect figure — never let a bad mark flow silently into the drift table.

## Step 2: Drift Analysis

Compare current allocation to the IPS targets / assigned model. **Sourcing the target:**

- **IPS on file — the IPS governs.** Check the connected system of record for it first; if it returns no target bands as data, ask the advisor for them or accept a pasted IPS. A model from the model-research platform (for example BlackRock Advisor Center) is comparison material for Step 3 here, never a substitute target.
- **Household managed to an assigned model** (per Step 1's pull or the advisor naming one): if BlackRock Advisor Center is connected (Step 3's connection check) and the assignment matches its model lineup (which also carries the firm's entitled third-party models), hydrate the target by reading that model's holdings: roll them up by their `asset_class` tags into sleeve targets and carry the model's `as_of` date into the drift table. The model supplies point targets only, never bands — bands stay firm/IPS policy; ask if unknown. Not connected → ask the advisor for the model's sleeve targets in one turn — and its bands, if the firm sets them — and carry on from those. Step 2b's analytics are skipped, never substituted, and the close offers to connect the platform.
- **No target on file:** if Advisor Center is connected, offer its lineup rather than improvising — show candidate families and risk profiles and let the advisor pick; never select a model or a risk profile for them. Label the drift table and every figure derived from it *"vs. advisor-selected illustrative model [name] — not a client IPS"*, and carry that label into the memo and the Executive Summary. If they decline, or it isn't connected, ask for target bands as above.

Source sleeve names, targets, and bands from whatever IPS/model you end up with — don't assume a fixed set of asset classes. Fall back to the table below only when no IPS/model sleeve structure is on file:

| Asset Class | Target % | Current % | Drift | $ Over/Under |
|------------|----------|-----------|-------|-------------|
| US Large Cap Equity | | | | |
| US Small/Mid Cap | | | | |
| International Developed | | | | |
| Emerging Markets | | | | |
| Investment Grade Bonds | | | | |
| High Yield / Credit | | | | |
| Alternatives | | | | |
| Cash | | | | |

A cell you can't fill is never blank and never guessed: a target or band you don't have reads "— pending target from advisor", and Drift and $ Over/Under are computed only where both the target and the current weight are in hand — otherwise they read "— pending [what's missing]".

Flag positions exceeding the rebalancing band (default ±5% absolute or the firm's stated band — ask if unknown). **Bands may differ by liquidity tier** — illiquid sleeves (private equity, venture, real assets) typically carry a wider band than liquid ones; use what the IPS specifies, ask if it doesn't distinguish. A breach in an illiquid sleeve is a commitment-pacing signal, not a trade instruction — it can't be traded back — so surface it as context rather than generating a change against it.

Also flag: concentrated single positions (above this review's 10%-of-household flag — name it as this review's, never as a standard or usual threshold), cash drag, and style drift within an asset class.

**Every drift, risk and concentration figure carries its governing reference.** Name it once, exactly as the drift analysis's Data Note labels it, and reuse that same string wherever a figure appears — this step's tables, the Step 5 memo, the Step 6 Executive Summary — so a figure and its reference can't drift apart between sections. Frame each strictly as within or outside that reference, never as good or bad; this skill takes no suitability position (see Out of Scope). This holds for the prose around the figures too: the Executive Summary, notes and the chat reply describe and never characterize. Saying what matters most (Step 6's Executive Summary) is still required.

The reference, in order: the IPS band; then the assigned model's risk profile; then a risk tolerance the household has actually stated, captured in meeting notes by the meeting-intelligence tool or the CRM. A captured tolerance is weaker than a policy document, so label it as what it is — quoted as stated, with its date — and let its form decide how it can be used. One carrying numbers ("no more than a 15% drawdown", "keep equity under 70%") is a usable reference: frame within or outside it. One that is a word — "moderate", "conservative" — is not, because there is no band to be within. Put it beside the figure as context, quoted, and compute no verdict from it; the advisor reads the two together and makes that call. With none of these on file, say "no stated reference on file" rather than presenting a figure bare or inventing a reference for it.

**IPS review cadence:** check the IPS's own "last reviewed" date against the firm's stated review cadence. If none is on file, default to flagging anything not reviewed in the last 12 months (a common industry norm for fiduciary accounts) and ask the advisor for the firm's actual policy — same pattern as the band default above. Note staleness alongside the drift table; don't block the rebalance on it.

**Analysis-only path:** if the advisor asked for analysis only (see Inputs), the review ends after Step 2b — write the drift analysis using `templates/drift-analysis.md`, Executive Summary first, and skip Steps 3–5.

## Step 2b: Portfolio Analytics

The drift table says *where* the household sits against its reference. This step says how much risk that carries and what happens to it under stress — the volatility and stress-test reads advisors ask for by name ("if rates rise, how does that affect my portfolio?", "is my equity too high?"). It is additive colour on top of the portfolio platform's own figures from Step 1, never a replacement.

**Connection check.** Look for the portfolio analytics platform's tools (for example BlackRock Advisor Center) by its own name, the way Step 1 says, never by a guessed `mcp__*` prefix. An advisor saying nothing is connected is not this check: run it anyway, and the listing below, before asking about a save or saying none was made. Its tools are there → this step runs; it is part of the review, not an offer. When it needs a new saved portfolio, the one thing it waits on is the advisor's agreement to that save (below). Not connected → skip it entirely and carry on unaffected. Never mark a figure "pending" because a step that didn't run would have produced it. And never compute one in its place: without it, no volatility, stress, scenario or risk-driver figure appears that you worked out yourself — not from assumed sensitivities, asset-class rules of thumb or the holdings themselves, and not as a substitute table in the section's place. A figure another connected system returns itself can be quoted, attributed to that system and paired with its reference.

**Find or build the saved portfolio.** The analytics read a *saved* portfolio. List the advisor's saved households and saved portfolios first — the advisor may already have one, theirs or an earlier review's, and both list saved objects only, so accounts synced from the custodian never appear. A saved one whose name isn't this household's is never used without asking, whatever it holds, since another client on the same model can hold the same funds: at most it is a candidate. Several plausible candidates, or one predating this review period → show each by name and the date it carries and ask which to analyze, whether to save a new one from today's holdings, or whether to leave the analytics out. Word the new option as the save it is, and end on the no: *"…, save today's holdings to your [platform] library as "[name]" (I can't delete it afterwards, and picking this is your OK to save it), or leave the analytics out of this review?"*, so an advisor who picks the save has agreed to it, knows it, and isn't asked again. That option is then the save request (below), and the question names every answer, the no included, so nothing follows it, and no second request for the same save goes beside the question. **That question is a data question** — a fact only the advisor holds. Exactly one current match → say which you're using and carry on: nothing is saved, so nothing is asked. Nothing saved → build one, once the advisor agrees to the save (below).

Build it as a saved household, named `"{Household Name} — Rebalance Review {YYYY-MM-DD}"`. Dated and accumulating: this platform is not a system of record and its portfolios are point-in-time artifacts, so each review keeps the ground truth its own figures were computed against. Never reuse a name — names must be unique among the advisor's saved portfolios and households, compared trimmed and case-sensitively, so a second run the same day takes a short distinguishing suffix.

**Never save a portfolio built on positions the advisor didn't supply.** Step 1's gathered holdings are the only book either of this skill's writes may carry — with the proxy or exclusion the advisor named for a position that wouldn't resolve, and nothing else. No stand-in, sample or illustrative positions, under this household's name or any other, however clearly they are labelled. With no holdings of the household's own in hand there is nothing to build from, and Step 1's ask-and-stop is already the answer.

**Ask before the first save, once per conversation, in one line, and save only with the advisor's agreement.** A saved portfolio stays in the advisor's library, because this plugin has no tool that deletes one, so like every other write in this plugin it waits for their agreement. Ask it in this one sentence, with the household, the platform and the name filled in, since it already says that nothing saved fits, what you would save, where, under what name, why, and that it stays, and ends in a question that names both answers: *"Nothing is saved for [household] in your [platform] library, so to run its risk and stress-test analytics I need to save these holdings there as "[name]", and I can't delete it afterwards — OK to save it, or should I leave the analytics out?"* That sentence is the whole request, and nothing else in the message is about the save: not what the saved-portfolio listing found, or what a no means, which the sentence says; not what happens once they answer, which the review that follows shows; and not how to answer it. Saying you are about to save is not asking. Ask as late as you can while still before the save: once the saved-portfolio listing and the security search below are back, in the same message as any question still open then, such as a proxy or exclusion for a position that wouldn't resolve, with this line last so that a bare "yes" plainly answers it. Where the which-to-analyze question above is open, its new-save option is this request in the sentence's place, and that question goes last instead. Then end your turn; the save, the analytics and the rest of the review, its file included, follow their answer. Ask once per conversation, not once per save: once the advisor has agreed in this conversation, every later save in it goes ahead without asking, the proposed book Step 4b saves included, and Step 6's close still names each one.

**What counts as agreement** is the advisor's own words, in this conversation, agreeing to a save in the platform: a yes to this line, picking the new-save option above, accepting Step 6's comparison offer, agreement given the same way in /pre-meeting earlier in this conversation, or saying it unprompted ("save whatever Advisor Center needs"). A yes with a condition you can meet, such as a proxy or an exclusion, is agreement to the save with that condition. A yes on a condition you can't meet, such as deleting it afterwards, is not agreement: say in a line that you can't, and go on as below. Invoking this review, a "go ahead" before you asked that doesn't mention saving, asking for risk or stress figures before being told they need a save, and a standing instruction in their settings are not agreement, so ask. A run that saves nothing asks nothing: one that analyses a portfolio already saved, one where the platform isn't connected, one with no holdings to build from, or one whose security search failed (below).

**Without agreement, nothing is saved and the analytics don't run.** That covers a no, and a reply that answers the other questions in the message and passes over the save: asking again would be a second request, so go on exactly as for a no. Carry on with everything that needs no saved portfolio: the drift analysis, the model reads and mapping, the plan and the memo. Leave this step out exactly as when the platform isn't connected: no figure marked pending, no gap line, none computed in its place. Nothing in the file says the analytics were left out or why — not the Data Note, not the Executive Summary, not the memo — because a line about them tells whoever reads the file that something is still owed when nothing is. The one place that says why the analytics are missing is Step 6's close, in one line, in chat. Step 4b isn't offered, and neither are the further reads. Don't ask again in this conversation, here or in the other skill. If the advisor then asks for them, or says yes after all, that is their agreement, because the close has told them what the analytics need: save, run them, and fold them into the file already written. Asking for the comparison counts the same way.

**Re-key the holdings.** Resolve every position through the platform's security search first, the whole set in one batched call rather than one per ticker. **If the search itself fails** — the platform refuses it, or answers with an error in place of the resolutions — no position can be keyed, so there is nothing to build and no save to ask for: don't ask it, and go on as for a write that fails (below). `unique` proceeds by its CUSIP. `ambiguous` or `none` is never guessed and never silently dropped — offer a proxy security the advisor names or explicit exclusion, with the portfolio total visibly adjusted down by that position's value, and record the adjusted total in the Data Note so no figure below is read against the wrong base. **Being asked to pick the proxy is not the advisor naming one.** "You choose" authorizes a substitution without making it, and which security stands in for another is a selection judgment this skill doesn't make (see Out of Scope) — the proxy sets the base every drift, risk and concentration figure below is read against. Put the choice back: what the security search returned, what each candidate is, and that excluding the position is an answer needing no pick. Pressed again, say that plainly and ask again. Until the advisor names a security or excludes it the position stays unresolved — a proxy you picked is not one they named, however the invitation was worded. Weights and market values are never mixed within an account; weights are fractions summing to 1.0 against a per-account value; a figure Step 1 didn't gather is a question for the advisor rather than a number to fill in.

**If the write fails, say so — never reshape the holdings to make it succeed.** Never drop a position, merge holdings, substitute a proxy the advisor didn't name, or adjust a weight to get past a rejection. Report it with whatever the platform said, skip the analytics for this review, and keep Step 1's data, which never had these constraints. Don't restate the platform's limits here: it enforces them, it changes them, and a copy can only drift toward being wrong. If the platform rejects one account rather than the whole household, offer to leave that account out — with the total visibly adjusted, the same choice as an unresolved position — and analyze the rest. The advisor's call; never reshape the account itself.

**Run the analytics** once the portfolio exists: the allocation, risk and scenario reads, and the platform's fund-health read. Read every result the way @${CLAUDE_PLUGIN_ROOT}/partials/rules/platform-analytics.md says, and fund health the way @${CLAUDE_PLUGIN_ROOT}/partials/rules/fund-health.md says. **Benchmark the risk read to the closest model the allocation read returns** — the benchmarked call returns tracking error and the active contribution behind it, which turns an allocation difference into its impact rather than just naming it. Step 2's reference rule governs every figure.

Fund health goes with the holdings it describes, whichever read returns it — this step's or a further read Step 6 offers — in the drift analysis's Current Allocation section, marked † with one line under the section: *† flagged by [platform]; not a recommendation*.

**The platform's advisory fee colours its net figures.** Its analyses carry an advisor-level fee preference; state it in the Data Note beside the figures, and note where it differs from the net-of-fees basis Step 1's performance already uses. Never write it — changing that preference is a billing action, not an analysis artifact, and is out of scope.

**Fold as three independently-omittable reads** into the drift analysis's Risk & Scenario Analysis section — **Volatility & Risk Drivers**, **Stress Test Impact** and **Allocation vs. Closest Model**. A read that didn't come back is omitted on its own and never drags the other two out.

## Step 3: Model-Portfolio Mapping

**Which model source:** check for BlackRock Advisor Center the same way Step 1
checks for a portfolio platform — your tool list, or a `ToolSearch` by its
name (never guess an `mcp__blackrock__*` prefix), `ListConnectors` only to explain a gap, and
read `enabledInChat` rather than `connected`. If it's connected, source the
model lineup and the chosen model's holdings from it. If it isn't, work from the model definition the advisor supplies
— and if none is supplied, say so and skip the mapping rather than scoring
against an assumed model. **A model from the model-research platform is not
automatically the drift target** — when the household has its own IPS, models
feed only this step's mapping. When Step 2 sourced the target from an
assigned or advisor-selected model, score against that same model here, never
a different one.

**Don't recite the lineup.** The model lineup read returns the firm's entitled models in bulk, and passing that on reads as "I have to evaluate all of these to pick a benchmark." Anchor on the household's actual asset-class split and offer a short shortlist that brackets it, naming what each is for.

**A model you aren't entitled to comes back empty rather than failing.** The holdings read on one outside the firm's entitlement returns no holdings and a zero weight total, with metadata that looks like any other. Check a hydrated target actually carries holdings before scoring against it — a silent zero target produces drift figures that are all wrong in the same direction.

**Its saved library is readable; the synced book of business is not.** The
portfolios and households saved in Advisor Center — ones the advisor built
there, or an earlier session did — are readable over MCP. Accounts synced from
the custodian are the part that isn't: they don't appear in those listings, and
an Advisor Center "household" is a grouping of saved portfolios, not the client
household the CRM knows. So a saved portfolio can be found by name, while a
client household can't be resolved here.
Independent of that: the only writes this skill
makes are the saved portfolio Step 2b builds for the analytics to read and, when
the advisor asks for the comparison, the proposed book Step 4b saves — each under
its own step's rules. Neither is saved without the advisor's agreement (Step 2b). Saving a standalone portfolio, and every proposal or model write, stays
out of scope — never offer to build a proposal, save a model, or change anything
an advisor would deliver to a client.

- If the household is assigned a model, score current holdings against it: matched positions, close substitutes, and orphan positions.
- If no model is assigned, show candidate families and risk profiles from BlackRock's lineup when connected, otherwise ask for the firm's lineup, and let the advisor pick — never select a model or a risk profile for them, and never fall back to stock risk-based sleeves. Until they pick one, leave the mapping out, as above, and carry on with the rest of the review. Present any BlackRock-sourced model or fund suggestion as an option Advisor Center is offering, not as Claude's own recommendation.
- Output a mapping table: current holding → model position → fit (matched / close substitute / orphan). A holding with no counterpart in the model reads "— none in model" in the model-position cell (an orphan), never a substitute you picked for it.

## Step 4: Rebalance Prep

Identify the adjustments that would bring the household back to target, tax-aware. **Don't rebalance for rebalancing's sake** — drift within bands is fine, and tax costs can exceed the benefit; where it's close, show the breakeven. Check pending cash flows (contributions, withdrawals, RMDs) before any sell appears in the plan:
- Rebalance in tax-advantaged accounts first (no tax consequences). If the household has none, say so explicitly in the plan and memo rather than silently skipping this.
- In taxable accounts: avoid realizing large short-term gains; harvest available losses while rebalancing; watch wash sales (30-day window, across every household account whose holdings and transaction history you **actually gathered** — being connected is not the same as being gathered: a held-away account the advisor confirmed exists but never supplied holdings for sits outside that check, and so does a connected account whose feed returned no cost basis or tax-lot detail, the gap Step 1's Addepar note and Completeness check already record). Where the check couldn't reach the whole household, say so plainly: name the accounts it covered and the ones it didn't, and carry that same scope into the memo's wash-sale line rather than a bare "yes". Also flag any proposed sale at a loss paired with a buy of a fund tracking the same or a substantially similar index (for example BND → AGG), for the advisor or tax preparer to confirm. Never call a pairing safe.
- Check RMD status before an IRA sell appears in the plan — don't create a distribution shortfall, and prefer using an already-required RMD as the funding source for a needed sale over an unrelated one.
- Where the draft plan sells from a concentrated position with a large embedded gain, note beside that row that the sale realizes the gain and forfeits any step-up in basis at death, for the advisor to weigh against the client's circumstances. Never estimate the client's holding horizon or health, and never let either decide whether a row is drafted.
- Prefer directing pending contributions/cash to underweights over selling
- Respect client restrictions (ESG screens, concentrated/legacy stock, lockups)

**Cash locked for capital calls (alts path):** if a connected system has a tool for the household's upcoming private-fund capital activity, call it before treating cash balances as available — net out committed-but-uncalled capital-call obligations from what counts as deployable cash, same as an illiquid-sleeve band breach is a pacing signal rather than a trade instruction. Surface any near-term call as a flag/context line next to the cash figures it affects, don't silently consume it into the plan. When nothing answers for capital calls — no such tool, one that errors or refuses, or an uploaded export — note that gap once if the household holds private funds, rather than guessing, and whether or not it does, don't read any account's cash as a likely call reserve: Step 2's cash-drag flag stands as written.

**Meeting-intelligence input to the rebalance rationale:** if the meeting-intelligence tool (for example Zocks) is connected, pull its AI results/insights for the household and check whether the potential changes line up with what the client has actually expressed — stated risk tolerance, life events — and with the IPS on file. Flag a mismatch inline against the specific change it bears on (e.g. a risky change against an explicitly stated low risk tolerance, or a life event suggesting the IPS itself may be stale), not as a general blurb at the top of the output. It is a sentiment/context source, never a portfolio book of record.

**Execution feasibility check** before finalizing the plan — flag, don't silently assume:
- Fractional shares: where a computed share count is fractional, note it and suggest whole shares or a dollar-based order, if the custodian supports one.
- Mutual funds: flag that the order is subject to the fund's minimum and daily NAV cutoff — tell the advisor to confirm both with the custodian/fund company, don't assume a number.
- Less-liquid ETFs (international, high-yield, muni): flag that premium/discount to NAV should be checked at time of trade.
- Trading the same security across many households at once (block trading) is out of scope for this skill's single-household design — note it as a known limitation, don't attempt to aggregate.

**Draft rebalance plan:**

| Account | Trade | Security | Shares/$ | What it addresses | Est. Tax Impact |
|---------|-------|----------|----------|-------------------|-----------------|

`What it addresses` says what the trade changes — "brings Fixed Income back inside its band" — never why it should be made; `Trade` reads Buy or Sell. Include totals: estimated realized gains/losses, transaction costs, and before/after drift. An Est. Tax Impact without cost basis for the lot reads "— cost basis not available", never an estimate, and a total that would need it says which rows it leaves out. This table is one draft plan for advisor review — laid out for the advisor to weigh, not advice; the advisor owns suitability and best execution, so present every row that way, never as a recommendation. **One plan, not a menu of approaches:** don't draft a second or a third unless the advisor asks for one, which Step 6's close offers.

**Execution note:** this skill *drafts a rebalance plan*; it never stages, places or executes an order. The plan is a draft for the advisor to weigh, and any orders go through the advisor's own custodian workflow.

## Step 4b: Compare Current Against Proposed — when the advisor asks

Step 4's plan describes a book that doesn't exist yet. This step shows the advisor how it would have behaved against the one that does.

**It runs only when the advisor asks for it.** Saving a draft plan under the client's name before the advisor has weighed it reads as the recommended path, so the comparison waits for them to ask. Step 6's close offers it; this step is what runs when they accept, in the same session, and nothing here is offered a second time once they decline. The allocation Before / After the drift analysis already carries is Step 4's own arithmetic and needs no saved book, so it is written either way.

Don't offer it, and don't run it, when Step 2b didn't run (unless the advisor has since agreed to the save, which runs it first), when the advisor asked for analysis only, or when Step 4 produced no changes worth comparing — a household in band across every sleeve has no proposed book to compare against.

**Save the proposed book as its own portfolio.** Accepting the comparison is the advisor's agreement to this save, because Step 6's offer says it saves one, so don't ask again, even when Step 2b analysed a portfolio already saved and asked nothing. The platform's performance comparison reads saved portfolios, so the proposal has to exist as one: a saved household under `"{Household Name} — Rebalance Review {YYYY-MM-DD} (Proposed)"` — Step 2b's name, suffixed, so the pair reads as one review's two states rather than two unrelated objects in the advisor's library. Build it from Step 4's post-trade weights, resolved the way Step 2b resolved the current book, carrying forward the same proxy and exclusion decisions the advisor already made rather than asking again.

**Run the performance comparison across the two.** Report what it returns: the periods both books have history for, and the difference between them. **Where the shared window is short because a holding was recently added, say so and say how short** — the tool silently narrows to the window both subjects cover, and an eight-month comparison presented as a multi-year one is worse than no comparison at all. Quote what the platform returns rather than recomputing it, and state the advisory fee it applied.

This is history, not prediction. Say so plainly — how the proposed allocation would have behaved over a period that has already happened is not a claim about how it will behave — and carry the same caveat into the Step 5 memo wherever it cites these figures.

Fold it into the drift analysis's Before / After section, alongside the allocation comparison already there.

## Step 5: Rebalance Rationale Memo

Every rebalance gets a memo for the compliance file — it is a books-and-records item. Use `templates/rebalance-rationale-memo.md` in this skill's folder. It must state: client objective/IPS reference, what drifted and why, what is being changed and what each change addresses, and tax impact considered. Why each change serves the client's interest, why the benefits outweigh the costs, and the alternatives weighed are the advisor's own reasons: per the template's §3–§5, write them only as the advisor gave them in this conversation, never as this draft came up with them, and "— advisor to complete" otherwise. Leave the template's Approval table empty — nothing in this review establishes who approved a change, and the advisor and any reviewer the firm requires sign for themselves. Never fill in a name or a date there, and never describe the memo as approved.

## Step 6: Output

Claude never appears in the file in the first person.

Write everything to a markdown file first, opening with the drift analysis in `templates/drift-analysis.md`'s structure — its **Executive Summary** is the top of the file:
- Executive Summary: three or four sentences for the advisor — where the household stands against its reference and the one or two differences that matter most, each framed within or outside that reference and never as good or bad (Step 2). A figure with no reference reads "no stated reference on file", never "typical" or "high". When Step 4 produced a draft rebalance plan, say that it follows and which sleeves it addresses, and nothing about how well they would work or what they would cost. If the advisor shares any of it with a client, run that through **/compliance** first.
- Drift analysis table
- Risk & Scenario Analysis, and Further reads (if Step 2b ran)
- The draft rebalance plan, with tax impact summary, followed directly by the before/after allocation comparison
- Model mapping table (if Step 3 ran)
- Rebalance rationale memo (using `templates/rebalance-rationale-memo.md`)

**Summary-First Convention:** open the chat response with the same Executive Summary that opens the file — the same sentences, not a paraphrase or a second written for chat — then offer two to four directions to go deeper. The tables stay in the file. **Only offer directions this run can actually deliver** — an analysis-only pass has no plan or memo to go deeper into, a run where Step 2b didn't happen because the platform isn't connected offers the analytics as something that could be run rather than something waiting to be shown, and a run without the advisor's agreement to the save offers neither the analytics nor anything that reads them. The directions and the export question are the reply's closing pair, with nothing between them.

What there is to offer, when this run can deliver it: the three further reads — thematic and factor exposure, concentration and flagged issues, and a wealth projection from the platform's projection read — percentile bands simulated from forward assumptions, labeled as a hypothetical range and never as a forecast or a plan result — on the portfolio Step 2b built; **an alternative approach**, drafted beside the plan already written — contributions only, or a rebalance to the band edge rather than to target; and **comparing current against proposed in Advisor Center**, which runs Step 4b and saves the proposed book as a portfolio of its own, so say so in the offer: *"compare today's book against the proposed one in [platform], which saves the proposed book to your library there as "[name]" (I can't delete it afterwards)"*. Accepting it is the advisor's agreement to that save (Step 4b). Name each plainly and run only what the advisor accepts: a further read is folded into the file's Further reads section and framed as the platform's own finding, an alternative approach goes beside the plan it is an alternative to, and the comparison folds in where Step 4b says. When the ask was meeting-shaped, offer /pre-meeting among them too (see the top of this file).

**Say what you wrote, after the pair.** Name whatever this run created in a connected system — one saved portfolio, and a second only where the advisor asked for the comparison — plainly, one line each, in the advisor's language rather than the tool's. Name what each is called so the advisor can find it later; this plugin has no tool that deletes it. Say it unprompted, only for writes that happened, and never soften it into asking whether that was alright. It is a report, not a request. Where the analytics waited on the advisor's agreement to a save and it didn't come, nothing was saved, so say instead, in one line: *"[platform]'s risk and stress-test analytics aren't in this review: they run only on a saved portfolio, and nothing was saved there without your OK."*

The file carries client portfolio and account data, so it holds what this review needs and nothing more.

**Write the review in the advisor's voice, or in none.** Claude never appears in the file as "I", "me" or "my" — not in the drift analysis, and not in the rationale memo, which is a books-and-records item other people read without this conversation around it. The chat close is a message rather than part of the file, so it isn't covered.

Then ask the advisor whether they'd like what this run wrote in another format: the draft plan exported to Excel and/or the rationale memo converted to Word/PDF, or, on an analysis-only pass, which wrote neither, the drift analysis converted to Word/PDF — don't create those formats unless they ask. **Confirm what an export should cover** before generating it — this review only, or everything in the session. Ask once, and only where there's real ambiguity. In the Excel export, any cell whose text begins with `=`, `+`, `-` or `@` — a security name or note taken from a pasted or uploaded holdings list — is written as a text cell (a leading apostrophe, or the writer's explicit string type), never as a formula.

## Out of Scope

- **No trade execution.** This skill prepares a draft plan for advisor review; it never places an order.
- **No fabricating holdings, lots, or cost basis.** When a connected source doesn't confirm a figure, say so — never fill the gap with an invented number.
- **No stand-in holdings.** No portfolio is saved to the analytics platform on positions the advisor didn't supply, and a proxy for a position that wouldn't resolve is never Claude's own pick — being asked to choose one is not the advisor naming it (Step 2b).
- **No suitability or legal determination.** This skill surfaces drift, tax impact, and what each change addresses for the advisor to weigh — it never states that a portfolio is or isn't suitable. That is what makes every risk and drift figure a within-or-outside-the-reference statement rather than a good-or-bad one (Step 2).
- **No proposals or models saved to the analytics platform**, and no change to its advisory-fee preference. The only writes this skill makes are the saved portfolios its analytics read — the current book, and the proposed one when the advisor asks for the comparison, and only with the advisor's agreement (Steps 2b and 4b).
- **No forecasts.** A performance comparison says how a proposed allocation would have behaved over a period that has already happened; it never says how it will behave (Step 4b).

## Important Notes

A summary; each rule is stated in the step it governs.

- Say up front that the review takes a few minutes, rather than asking whether to proceed (Inputs).
- **Take the advisor's grain literally** — a named account is that account, a household is the household (Inputs).
- **Claude never appears in the file in the first person** (Step 6).
- **A connected system that can't return holdings means ask and stop** — the one exception, among missing sources, to carrying on with the best pass available (Step 1).
- **Every drift, risk and concentration figure carries its governing reference**, named once in the Data Note's form and reused everywhere, framed within/outside it and never good/bad — "no stated reference on file" when there is none (Step 2, carried into Steps 5 and 6).
- **The analytics pass runs when the platform is connected and the advisor agrees to the save it needs** — asked once per conversation, in one line, before the first save; without agreement the review carries on without the analytics, and every save is named in the close (Steps 2b and 6).
- **The current-against-proposed comparison is a direction the advisor asks for**, not part of the pass — it saves the proposed book as a portfolio of its own, and accepting it is agreement to that save; the allocation Before / After is written without one (Steps 4b and 6).
- **Lead the chat response with the Executive Summary**, then offer two to four directions this run can actually deliver — the tables stay in the file (Step 6).
- Don't rebalance for rebalancing's sake — drift within bands is fine; tax costs can exceed the benefit. Show the breakeven when it's close (Step 4).
- Check pending cash flows (contributions, withdrawals, RMDs) before a sell appears in the plan (Step 4).
- All buy/sell output is **a draft for advisor review — one plan for the advisor to weigh, not advice**; an alternative approach is drafted only on request (Steps 4 and 6).
- Document everything: the rationale memo is a books-and-records item; run anything the advisor shares with a client through **/compliance** (Steps 5 and 6).
- Coordinate wash sales across every household account whose holdings and tax-lot detail you actually gathered, and say so when the check couldn't cover them all (Step 4).
- Treat client portfolio and account data as confidential; only include what's needed for this review (Step 6).
