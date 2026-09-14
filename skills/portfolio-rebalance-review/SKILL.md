---
name: portfolio-rebalance-review
description: Run a portfolio review for a client or household — performance pulled from Orion, allocation drift analysis vs. IPS or model portfolio, model-portfolio mapping, rebalance preparation with tax-aware rebalance opportunities to consider, and a rebalance rationale memo for the compliance file. Triggers on "portfolio review", "/portfolio-rebalance-review", "drift analysis", "rebalance [client]", "how is [client]'s portfolio positioned", "rebalance rationale", or "map [client] to a model".
---

# Portfolio Rebalance Review

Review a client portfolio end to end: performance → drift → model mapping → rebalance prep → rebalance rationale memo.

## Inputs

Required: **client/household name**. Optional: which accounts to include, the model portfolio or IPS targets (if not on file), and whether the advisor wants analysis only or a full rebalance prep.

## Step 1: Pull Performance & Holdings

**Disambiguation rule:** confirm which household before pulling anything, the same as every other skill in this plugin — if a name search returns more than one match, show the advisor the candidates and ask which one before proceeding. Never proceed on a name match alone.

Query the household's connected portfolio-data system for:
- Accounts with registrations (taxable, IRA, Roth, 401k, trust) and market values
- Holdings with quantity, market value, cost basis, and unrealized gain/loss
- Performance: QTD/YTD/1yr/3yr vs. assigned benchmark, net of fees
- Assigned model/target allocation and cash balances
- Recent flows (contributions, distributions, RMD status)

**Which system to query:** this data can live in Orion, Addepar, or Envestnet/Tamarac depending on the firm. Check `ListConnectors` (load via `ToolSearch` if not already available) to see which is actually connected — don't guess at tool-name prefixes, use `ToolSearch` by system name once you know which is live.

**Look for the tools before you trust the registry.** `ToolSearch` by the system's own name is the check that decides: if its tools come back, that system is connected and callable — use them. `ListConnectors` can answer "No installed connectors found" even in a session with several live, working connectors, and in some clients it renders a user-facing card rather than returning data at all. So it explains a gap, it never establishes one, and an empty result means **unknown**, never "nothing is connected". When it does return entries, read `enabledInChat`, not `connected`: `connected: true` with `enabledInChat: false` is authenticated but switched off for this chat, so tell the advisor they can enable it here rather than reporting it as unconnected; a missing or `null` `connected` is unknown, not disconnected.
- **If none is connected**: follow the Connector Placeholder Convention — say *"This is where I'd make a call out to [system] to pull [holdings/performance] once that connector is built,"* then offer the fallback. Before asking for a manual upload, offer to search Google Drive (if connected) for an existing export or statement. If nothing turns up there either, ask the advisor to upload an export (CSV/PDF), a custodian statement, or paste holdings. Continue with whatever's provided, and mark anything still missing "— pending [connector]" rather than leaving it blank — a missing source degrades only the section that depends on it, not the whole review.
- **If more than one connected tool could try to solve this for the household**, ask the advisor once which is the book of record — per the Ask-Once, Then Route Convention — rather than guessing or merging both, and use that system for the rest of this review. This applies even when the household name hasn't resolved against either system yet: ask before searching further, not after finding out neither recognizes the name — an unresolved household is not an exception to this rule. Offer to help the advisor save the choice using the Personalization Convention so they aren't asked again next session. This check runs for the rest of the review, not just once up front — if the advisor brings up another source mid-review, ask before pulling from it too. And if a second system's figures get pulled anyway and disagree with the routed one on a material fact (a balance, a drift percentage), that's a blocking question for the advisor, not a note in the output.

**Held-away accounts:** ask the advisor whether the household has accounts held away from this system (a spouse's 401k, assets at another custodian) relevant to wash-sale coordination or the overall allocation picture. If yes, request those holdings manually — there's no connector for outside accounts.

**Addepar specifics:** call `get_portfolio_data` with `lookthrough=false` for drift purposes. Lookthrough decomposition (fund → underlying constituents) is a separate risk-review lens — an IPS names sleeves at the fund-commitment level, and lookthrough silently returns an incompatible set of buckets with no warning. Group by `direct_owner`, not `account` — the latter is not a valid grouping key and errors. For tax-lot detail, pass a tax-lot value in `groupings` only if the live tool schema actually documents one — cost basis and lot-level fields are not confirmed to exist on every Addepar deployment. If the schema doesn't support it or the returned columns don't include cost basis, say so explicitly rather than presenting fabricated lot data; this feeds the same "Completeness" gate below.

**Orion specifics:** Orion Connect's tools (household search, report catalog, report parameters, `oc_generate_report`) are identity and report-catalog/generation only — they do not return structured holdings, cost basis, or tax lots. Generating a report yields a **link to the finished report**, not its contents — hand it to the advisor and say it **expires after 24 hours** (a unique per-run URL; regenerate rather than troubleshoot an expired one); don't fetch it to scrape a holdings table out of, and don't treat having produced a link as having read the data. If Orion is the only connected system, holdings/cost-basis data has to come from the advisor (upload/paste) rather than being inferred from a report. If Orion later ships a structured holdings or cost-basis tool, wire it into this same Step 1 data shape alongside Addepar.

**Before generating an Orion report:** confirm with the advisor first — `oc_generate_report` runs a job in Orion Connect rather than reading from it. No confirmation is needed for household search, the report catalog, or report-parameter lookups; those are identity/catalog calls, not report generation.

**Data-sanity gate — run before Step 2, don't skip.** Dispatch this to the `claude-for-financial-advisors:holdings-sanity` subagent — `Agent(claude-for-financial-advisors:holdings-sanity)` — rather than working through it inline: hand it the holdings data, the reported portfolio/household value, and the sleeve structure from the IPS or model if one is on file. It returns a flag table with a `Contaminates` column naming which downstream figure each problem feeds, plus an explicit list of the checks that passed.

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

- **IPS on file — the IPS governs.** Prefer pulling it from the connected system of record if one exposes it as structured data; today no connected system does (Addepar has IPS/target bands in-product but doesn't expose them over MCP yet), so ask the advisor for target bands or accept a pasted IPS — this instruction upgrades automatically once a connector exposes IPS data. A BlackRock model is comparison material for Step 3 here, never a substitute target.
- **Household managed to an assigned model** (per Step 1's pull or the advisor naming one): if BlackRock Advisor Center is connected (Step 3's connection check) and the assignment matches its lineup (`list_models` — which also carries the firm's entitled third-party models), hydrate the target from `get_model`: roll its holdings up by their `asset_class` tags into sleeve targets and carry the model's `as_of` date into the drift table. The model supplies point targets only, never bands — bands stay firm/IPS policy; ask if unknown.
- **No target on file:** if Advisor Center is connected, offer its lineup rather than improvising — show candidate families and risk profiles and let the advisor pick; never select a model or a risk profile for them. Label the drift table and every figure derived from it *"vs. advisor-selected illustrative model [name] — not a client IPS"*, and carry that label into the memo and the client-facing summary. If they decline, or it isn't connected, ask for target bands as above.

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

Flag positions exceeding the rebalancing band (default ±5% absolute or the firm's stated band — ask if unknown). **Bands may differ by liquidity tier** — illiquid sleeves (private equity, venture, real assets) typically carry a wider band than liquid ones; use what the IPS specifies, ask if it doesn't distinguish. A breach in an illiquid sleeve is a commitment-pacing signal, not a trade instruction — it can't be traded back — so surface it as context rather than generating a change against it.

Also flag: concentrated single positions (>10% of household), cash drag, and style drift within an asset class.

**IPS review cadence:** check the IPS's own "last reviewed" date against the firm's stated review cadence. If none is on file, default to flagging anything not reviewed in the last 12 months (a common industry norm for fiduciary accounts) and ask the advisor for the firm's actual policy — same pattern as the band default above. Note staleness alongside the drift table; don't block the rebalance on it.

**Analysis-only path:** if the advisor asked for analysis only (see Inputs), stop here — output the drift analysis and the plain-English summary from Step 6, and skip Steps 3–5.

## Step 3: Model-Portfolio Mapping

**Which model source:** check for BlackRock Advisor Center the same way Step 1
checks for a portfolio-data system — `ToolSearch` by name first (never guess
an `mcp__blackrock__*` prefix), `ListConnectors` only to explain a gap, and
read `enabledInChat` rather than `connected`. If it's connected, source the
model lineup from `list_models` and the chosen model's holdings from
`get_model`. If it isn't, work from the model definition the advisor supplies
— and if none is supplied, say so and skip the mapping rather than scoring
against an assumed model. **A BlackRock model is not automatically the drift
target** — when the household has its own IPS, models feed only this step's
mapping. When Step 2 sourced the target from an assigned or advisor-selected
model, score against that same model here, never a different one.

**Advisor Center's MCP doesn't expose synced client accounts today:** its
"households" are proposal groupings, and accounts imported or synced from the
book of business aren't readable over MCP yet — so don't look a client
household up there; this instruction upgrades automatically once the MCP
exposes synced accounts. Independent of that: never offer to create or update
anything there — `create_portfolio` and every other Advisor Center write is
out of scope for this skill.

**Terms gate:** a `[terms_not_acknowledged]` or `[terms_unavailable]` refusal
from any BlackRock call means the advisor hasn't accepted (or needs to
re-accept) Advisor Center's terms. Call `show_terms` and present the terms
text in full — never summarized or paraphrased. Once the advisor explicitly
accepts, call `acknowledge_terms` with `displayed_hash` set to `show_terms`'s
`content_hash`, then retry the original call. Never call `show_terms` or
`acknowledge_terms` on your own initiative or as a routine check — only on an
actual refusal, or if the advisor asks about terms directly.

- If the household is assigned a model, score current holdings against it: matched positions, close substitutes, and orphan positions.
- If no model is assigned, propose the best-fit model — from BlackRock's lineup when connected, otherwise ask the advisor for the firm's lineup or use standard risk-based sleeves: Conservative 30/70 → Aggressive 90/10 — and show the gap analysis. Present any BlackRock-sourced model or fund suggestion as an option Advisor Center is offering, not as Claude's own recommendation.
- Output a mapping table: current holding → model position → action (hold / substitute / sell).

## Step 4: Rebalance Prep

Identify the adjustments that would bring the household back to target, tax-aware:
- Rebalance in tax-advantaged accounts first (no tax consequences). If the household has none, say so explicitly in the options list and memo rather than silently skipping this.
- In taxable accounts: avoid realizing large short-term gains; harvest available losses while rebalancing; watch wash sales (30-day window, across every household account whose holdings and transaction history you **actually gathered** — being connected is not the same as being gathered: a held-away account the advisor confirmed exists but never supplied holdings for sits outside that check, and so does a connected account whose feed returned no cost basis or tax-lot detail, the gap Step 1's Addepar note and Completeness check already record). Where the check couldn't reach the whole household, say so plainly: name the accounts it covered and the ones it didn't, and carry that same scope into the memo's wash-sale line rather than a bare "yes".
- Check RMD status before an IRA sell appears in the options — don't create a distribution shortfall, and prefer using an already-required RMD as the funding source for a needed sale over an unrelated one.
- For a concentrated position with a large embedded gain, weigh trimming against the client's likely holding horizon: selling now forfeits any step-up in basis at death. For an elderly or terminally-ill client this is a real tradeoff to reason through and document, not a default "trim because it's outside the band."
- Prefer directing pending contributions/cash to underweights over selling
- Respect client restrictions (ESG screens, concentrated/legacy stock, lockups)

**Cash locked for capital calls (alts path):** if Addepar is the connected system, call its upcoming private-fund capital-activity tool before treating cash balances as available — net out committed-but-uncalled capital-call obligations from what counts as deployable cash, same as an illiquid-sleeve band breach is a pacing signal rather than a trade instruction. Surface any near-term call as a flag/context line next to the cash figures it affects, don't silently consume it into the options list. No connected system currently supplies forward-looking call dates for Orion/Envestnet-only households — note that gap rather than guessing.

**Zocks AI results as a rebalance-rationale input:** if Zocks is connected, pull its AI results/insights for the household and check whether the potential changes line up with what the client has actually expressed — stated risk tolerance, life events — and with the IPS on file. Flag a mismatch inline against the specific change it bears on (e.g. a risky change against an explicitly stated low risk tolerance, or a life event suggesting the IPS itself may be stale), not as a general blurb at the top of the output. Zocks is a sentiment/context source, never a portfolio book of record.

**Execution feasibility check** before finalizing the options list — flag, don't silently assume:
- Odd lots: if a computed share count isn't a round lot, note it and suggest rounding to the nearest round lot or a dollar-based order.
- Mutual funds: flag that the order is subject to the fund's minimum and daily NAV cutoff — tell the advisor to confirm both with the custodian/fund company, don't assume a number.
- Less-liquid ETFs (international, high-yield, muni): flag that premium/discount to NAV should be checked at time of trade.
- Trading the same security across many households at once (block trading) is out of scope for this skill's single-household design — note it as a known limitation, don't attempt to aggregate.

**Rebalancing options:**

| Account | Action | Security | Shares/$ | Reason | Est. Tax Impact |
|---------|--------|----------|----------|--------|-----------------|

Include totals: estimated realized gains/losses, transaction costs, and before/after drift.

**Execution note:** this skill *lays out rebalancing options*; it never executes anything. Once custodian connectors (e.g., Schwab) support order staging, offer to stage the order file — until then say: *"This is where I'd stage these orders to [Schwab/custodian] once that connector is built"* and output the options list in the custodian's upload format if known.

## Step 5: Rebalance Rationale Memo

Every rebalance gets a memo for the compliance file. Use `templates/rebalance-rationale-memo.md` in this skill's folder. It must state: client objective/IPS reference, what drifted and why, what is being changed and why each adjustment serves the client's interest, tax impact considered, and alternatives considered. Leave the template's Approval table empty — nothing in this review establishes who approved a change, and the advisor and any reviewer the firm requires sign for themselves. Never fill in a name or a date there, and never describe the memo as approved.

## Step 6: Output

Write everything to a markdown file first:
- Drift analysis table + before/after allocation comparison
- Rebalancing options with tax impact summary
- Model mapping table (if Step 3 ran)
- Rebalance rationale memo (using `templates/rebalance-rationale-memo.md`)
- 3-sentence plain-English summary the advisor could relay to the client

Then ask the advisor whether they'd like the options list exported to Excel and/or the rationale memo converted to Word/PDF — don't create those formats unless they ask.

## Out of Scope (for now)

- **No trade execution.** This skill prepares options for advisor review; it never places an order.
- **No fabricating holdings, lots, or cost basis.** When a connected source doesn't confirm a figure, say so — never fill the gap with an invented number.
- **No suitability or legal determination.** This skill surfaces drift, tax impact, and rationale for the advisor to weigh — it never states that a portfolio is or isn't suitable.

## Important Notes

- This review can take a few minutes once portfolio data starts flowing across steps — say so up front rather than asking whether to proceed; invoking the skill is already the go-ahead.
- Don't rebalance for rebalancing's sake — drift within bands is fine; tax costs can exceed the benefit. Show the breakeven when it's close.
- Check pending cash flows (contributions, withdrawals, RMDs) before a sell appears in the options.
- All buy/sell output is **a draft for advisor review — options laid out for the advisor to weigh, not advice** — the advisor owns suitability and best execution.
- Document everything: the rationale memo is a books-and-records item; run any client-facing summary through **/compliance**.
- Coordinate wash sales across every household account whose holdings and tax-lot detail you actually gathered — a held-away account once the advisor supplies its holdings, a connected account once its feed actually returns cost basis — and say so when the check couldn't cover them all.
- Treat client portfolio and account data as confidential; only include what's needed for this review.
