# Claude for Financial Advisors

A plugin from Anthropic that gives financial advisors a set of ready-to-run workflows for Claude Cowork that draw live data from the tools their firm already uses, by way of third-party connectors.

The plugin is a set of skills written as plain instructions (Markdown and JSON). It holds no client data: when an advisor runs a skill, Claude pulls what it needs from the connected third-party systems. The advisor must explicitly approve any action that writes back to an external system (e.g. CRM updates, client drafts). Claude gathers, organizes, summarizes, compares and presents information, while the advisor exercises judgment, makes the recommendation and gives the advice.

Educational and operational support for advisors; not investment, legal, or tax advice.

**Maintenance status:** Reference implementation. Not actively maintained or monitored, and not accepting contributions. Issues and pull requests may not receive a response. This project is provided AS IS, without warranty.

## Skills

| Skill | What it does |
| :---- | :---- |
| `/onboarding` | Guided first-run setup: learns the advisor's firm, priorities, and tech stack, then connects their tools and runs a sample demo. |
| `/pre-meeting` | Builds a client review prep doc \- snapshot, relationship signals, open items, portfolio review, planning topics, agenda, talking points. |
| `/post-meeting` | Turns a meeting artifact or transcript into a CRM note. |
| `/compliance` | Pre-checks client-facing material against SEC Marketing Rule, antifraud, books-and-records, and Reg BI; produces a findings table, suggested disclosures, and a clean redraft. |
| `/prospect-intake` | Normalizes a prospect's statements into a summary, an analyst handoff, and a what-to-expect memo. |
| `/portfolio-rebalance-review` | Reviews performance, drift vs. IPS or model, and model mapping; drafts a tax-aware rebalance plan for the advisor to evaluate, with alternatives on request, and a memo documenting the advisor’s rationale. |
| `/alts-brief` | Meeting-ready read on a household's alternative investments, sized against the liquid book. |
| `/estate-and-tax-brief` | Cross-checks the estate plan against actual account titling and beneficiaries; prior-year tax look-back; optional CRM follow-up tasks. |

Each skill pulls from whichever connectors are live and falls back to paste/upload when a system isn't connected. Every write to a client system pauses for advisor approval.

## Connectors

| Connector | What it does |
| :---- | :---- |
| 55ip | J.P. Morgan Asset Management (55ip) extends tax transition technology to Claude, so advisors can evaluate the tax impact of moving a client's holdings into a new portfolio. |
| Addepar | Brings investment intelligence into Claude, giving advisors governed access to portfolio data, analytics and workflows across public and private markets. |
| BlackRock | Brings Advisor Center’s portfolio analytics, model portfolios, and investment research resources to advisors through Claude. |
| Black Diamond | Brings portfolio, performance, holdings and rebalancing data into Claude, so advisors can review their book, spot clients out of tolerance, surface tax loss harvesting opportunities, and generate client reports, tasks and rebalance sessions with their approval. |
| CAIS | CAIS opens its alternative investment platform to advisors in Claude, so they can query fund data, review manager and fund performance, and see a client's public and private holdings together. |
| Conquest Planning | Conquest brings its Strategic Advice Manager planning engine into Claude so advisors can create new client plans, review and apply strategies, update existing plans and prepare for client meetings all in one place. |
| Envestnet Tamarac | Brings portfolio and performance reporting, including reconciliation status, from Tamarac into Claude so that advisors can conduct household reviews, drift checks, and meeting prep. |
| FINNY | FINNY, the growth and prospecting platform, connects Claude to an advisor's campaign results by channel and sequence step, and lets advisors enrich prospect lists from their CRM, or upload new lists directly in Claude. |
| Flourish Financial | Flourish connects Claude to financial advisors’ books of business, enabling them to work with client information, check rates, manage invitations, and prepare for client meetings, all within Claude. |
| Franklin Templeton | Franklin Templeton provides publicly approved fund and share-class information in Claude, including performance, fees, NAV, holdings, distributions, and fund documents, along with its published market and investment insights. |
| iCapital | Brings a client’s alternatives book into Claude, with holdings and performance across private funds, so advisors can see NAV against commitments, unfunded capital, recent calls, and distributions alongside the liquid portfolio. |
| Invesco | Invesco brings portfolio intelligence and practice management insights into Claude through its Invesco for Advisors connector. |
| Jump | Jump, an AI platform for financial professionals, gives Claude client context (meetings, documents, tasks) and Jump-built skills that prepare next steps for the advisor’s review. |
| MoneyGuide | Brings a household's financial plan into Claude — net worth, probability of success, and goals. |
| MSCI | MSCI brings its proprietary data and analytics into Claude, so wealth firms can assess investment options across public and private markets, monitor performance, and evaluate rebalancing opportunities. |
| Nevis | Nevis, the AI platform for wealth management, draws on client information from meetings, documents, and communications to support meeting prep and follow-up in Claude. |
| Orion Advisor Solutions | Brings portfolio and performance reporting from Orion Connect, and client records from Redtail CRM, into Claude so that advisors can conduct household reviews, drift checks, and meeting prep. |
| Schwab | Brings custodial data from Schwab into Claude: accounts, balances, positions and their lots, transactions, and alerts. |
| Vanguard | Brings information on its model portfolios and advisor investment solutions into Claude, so advisors can draw on Vanguard's trusted research, portfolio construction, and asset allocation expertise when making decisions for clients. |
| Wealthbox | Connects Claude to client records and meeting history to power onboarding, meeting prep, and follow-up. |
| Wealth.com | Gives Claude a structured view of each client’s estate plan, including trust and will summaries, and the full balance sheet, so advisors can begin document review using an organized summary of available information instead of a binder full of documents. |
| Zocks | Brings Claude the client intelligence it captures from every conversation, including profiles, goals, life events, and commitments. |

These join other connectors already available in Claude, including Microsoft 365, Salesforce, Box, FactSet, S\&P Global, Morningstar, and more.

The Claude for Financial Advisors plugin, which bundles advisor skills and connectors into a single install, has been tested with BlackRock, Charles Schwab, Addepar, Envestnet, iCapital, Orion, Wealthbox, Wealth.com and Zocks, and advisors choose which of these to connect during guided setup. Some partners also offer their own plugins: BlackRock is launching one for Advisor Center, joining existing plugins from S\&P Global and LSEG.

## Installation

```
claude plugin marketplace add anthropics/claude-for-financial-advisors
claude plugin install claude-for-financial-advisors
```

## Contributing

This repo is not actively monitored (see the maintenance status above). [CONTRIBUTING.md](CONTRIBUTING.md) describes how the plugin is put together and the design rules to keep if you fork or adapt it, plus the CLA requirement in case a pull request is ever reviewed.

## Security

To report a security vulnerability, see [SECURITY.md](SECURITY.md).

**Security considerations.** The skills and agents here are instructions, not code. Some guardrails are structural: the file-parsing subagents are allowlisted to read-only tools, and the connector-reading subagents are denied shell, file-write, and web tools. The rest (advisor approval before any write, connector-reading subagents never calling a connector's write or send tools, arithmetic in a shell that never sees document text) are enforced by the model following them, not by the runtime. Subagents read content from third parties: prospect statements, client emails, CRM notes, testimonials. Treat that content as untrusted. That rule is written into each extracting agent, and into `post-meeting` and `compliance`, whose main-conversation input is itself instruction-shaped; a value taken from such content is escaped in a table cell and stored as text, never as a formula, in a spreadsheet conversion. If you adapt these agents, keep the read-only and shell rules in place, prefer connectors that expose read-only tools, and do not widen any agent's tool list without reviewing what it ingests.

## Disclosure
**Important Information: For Financial Professional Use Only**  
Anthropic provides this plugin as an AI-powered technology service to assist financial professionals with research, analysis, information retrieval, issue identification, and the preparation of drafts, summaries, presentations, and other work product. Anthropic is a technology provider only and is not acting as, or holding itself out as, an investment adviser, broker-dealer, bank, lender, fiduciary, insurance intermediary, payment services provider, or other regulated financial services provider.

The plugin supports professional workflows but does not provide investment advice, personalized recommendations, suitability determinations, portfolio allocations, target prices, rankings, or recommendations regarding any security, investment, financial product, transaction, counterparty, or strategy. Anthropic does not make investment decisions, act on behalf of clients or users, monitor accounts, portfolios, holdings, customers, or market activity, exercise discretion or authority over any account or transaction, facilitate or execute transactions, provide ongoing portfolio management or advisory services, or consider any person's financial circumstances, investment objectives, risk tolerance, tax status, or other personal characteristics when generating outputs.

Outputs are generated by AI in response to user prompts and may contain errors, omissions, inaccuracies, incomplete information, or outdated information. No output constitutes investment, legal, tax, accounting, regulatory, compliance, or other professional advice; a recommendation, suitability or fiduciary determination; a credit or underwriting decision; or an offer, solicitation, or recommendation to engage in any transaction or strategy. Outputs must be independently reviewed and may not be relied upon as the sole basis for any investment, trading, lending, underwriting, compliance, supervisory, or customer-related decision.

Users remain solely responsible for all investment decisions, client communications, supervisory review, recordkeeping, and compliance with applicable laws, regulations, and firm policies. Anthropic does not monitor, supervise, or assess the suitability, legality, appropriateness, or compliance of actions taken based on outputs. The plugin is intended only for appropriately licensed, registered, authorized, or approved firms and financial professionals. Content obtained from third-party providers remains subject to applicable third-party terms and restrictions. Outputs are not intended for direct distribution to clients, investors, or the public without any review and approval required by the user's firm.

## License

Licensed under the [Apache License, Version 2.0](LICENSE).

Copyright 2026 Anthropic PBC.
