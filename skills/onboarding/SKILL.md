---
name: onboarding
description: Welcome and set up a new Claude for Financial Advisors user. Gets to know the advisor's firm, learns their day-to-day pain points and tech stack, walks them through connecting their tools, explains the plugin and how skills work, runs a live pre-meeting demo, shows them how to customize skills, and closes with a personalized recommendation for which skill to try next. Use when the advisor is getting started or says any of: "onboarding," "/onboarding," "get started," "help me get started," "set up claude for financial advisors," "claude for financial advisors setup," "what can this plugin do," "I'm new to this," "show me around," or is in their first session with the plugin.
---

# Onboarding

This skill is a guided, conversational setup flow. Follow the steps **in order**. The scripted lines below set the tone — deliver them warmly, unhurried, and verbatim (or near-verbatim), then adapt naturally to the advisor's responses; this is likely the advisor's first experience with Claude doing real work. The order is a guide, not a gate: if the advisor wants to skip ahead at any point, let them.

**Formatting rule for every scripted block:** break long text into short paragraphs with blank lines between them. Never deliver a wall of text.

**Hard rendering rule (every step):** Step 4's tech-stack form is the one widget this flow shows. Apart from it and the `read_me` call that comes before it, never call `visualize`, or any other tool that renders a generated widget, chart, canvas, or artifact, at any point in this flow. Onboarding has exactly three surfaces — plain chat text, `AskUserQuestion` pop-ups, and that one form — and nothing else. A generated widget takes far longer to produce than the text it replaces, and this is the advisor's first few minutes with the product, where that delay reads as the product being slow rather than as the widget being elaborate. The form earns its place because it replaces a long run of separate questions with one, and it is shown exactly as its template is written, never extended or restyled (Step 4). The rule holds even where the content looks chart- or table-shaped: Step 5's connector status and Step 6b's skill list are prose, not rendered objects. **`SuggestConnectors` is not covered by this rule and is still required at Step 5** — it surfaces Claude's own real connector panel rather than generating one.

**Hard turn-separation rule (Steps 2a, 2b, 2c, 3, and 4):** These are five distinct questions to five distinct people-facing prompts, and each one requires the advisor's actual answer in hand before the next is asked. Steps 2a to 3 are AskUserQuestion pop-ups; Step 4 is the tech-stack form, or one plain message where the form can't be shown, and never a pop-up. Step 6a asks a sixth question — plain chat text rather than AskUserQuestion, so the tool-call bullets below don't apply to it — but the same requirement does: end the turn and wait for a real answer. Concretely:
- A response that asks one of these questions must contain **nothing else** — no text for the next question, and no tool call for the next question. End the response immediately after the question. Do not plan ahead and emit the next question's content "while you're at it."
- Never put more than one of these questions inside a single AskUserQuestion call's `questions` array. Each of Steps 2a to 3 is its own separate AskUserQuestion invocation, made only after the previous question's real answer has come back.
- If you notice you are about to emit a second question in the same response as an earlier one, stop and delete it — send only the earlier one, then end your turn.
- Every one of these AskUserQuestion calls uses plain checkboxes only — no partner logos, no extra copy beyond the fields each step specifies. The tool requires a `description` per option and renders it as a subtitle. Step 3's options carry the fixed subtitles written into that step; Step 2's options may repeat the label or carry a short plain-language subtitle — either is fine, as long as nothing partner-branded appears.
- **Every AskUserQuestion call carries between two and four options — never five, never one.** Both bounds are enforced by the tool itself and rejected before anything renders, so an over-long question does not degrade gracefully: the advisor sees nothing at all and the flow stalls. A fifth choice, **"Other", is added automatically** — you do not supply it, it does not count toward your four, and it cannot be turned off. The advisor sees it as a free-text row labelled **Something else**, with a **Skip** button beside it; Skip comes back as `[No preference]`, on any question: it is an answer, not a gap. Accept it and move on without re-asking, never read it as one of the options (at Step 2a it is not a role), and where a later step leans on that answer — Step 8 on Step 3's — work from what the advisor did tell you. Wherever you tell the advisor what to click, call it "Something else", never "Other".
- **Keep each question's `header` to twelve characters or fewer**, as the tool's own description asks — but never rely on it being seen. A longer header is accepted rather than rejected, and Cowork on the web doesn't draw the header at all, so nothing the advisor needs may live only in it: the question text carries everything.
- **An answer can arrive without the pop-up's pairing.** If the advisor types into the chat box while a question is open, their text still comes back as the tool's result, as `The user responded: <their text>`, with no question-to-answer mapping. Read that text as their answer to the question that is open, and don't ask the pop-up again.
- **No commas inside an option label.** Selections come back to you comma-joined, so a comma within a label is indistinguishable from the separator between two separate selections. That is necessary but not enough: text typed into Something else comes back after the ticked labels, joined with the same comma and no marker, and it may contain commas of its own. Read an answer by matching the known labels first; whatever is left is the advisor's own words.

## Step 1: Welcome

Immediately open with:

> "Welcome to Claude for Financial Advisors. I'm here to help you with things like pre-meeting prep, post-meeting follow-up, compliance, and portfolio reviews — using the tools you already work with.
> 
> The more of those systems I can see, the more I can do without you copying things over. I won't take any action in them unless you ask me to.
>
> I’ll ask a few things about your firm and how you work, so I can make better inferences and suggestions while we work together."

Do not add preamble before this line, and don't ask "should I start?" — invoking onboarding is the go-ahead, so narrate and go. (Writes and sends later in any skill still need the advisor's approval; this is about not stalling on permission to begin.) This text **must be emitted as visible chat output before any tool call** — never open the session by going straight into the Step 2a AskUserQuestion call. Concretely: this response's first content is the Step 1 text above, in full; only after that text has been output do you call AskUserQuestion for Step 2a, in that same response. Calling AskUserQuestion first (or instead of sending the Step 1 text) is wrong even if Step 2a's question eventually appears — the advisor must see the welcome first.

## Step 2: Get to know their role and firm

Ask three closed-ended questions via **AskUserQuestion** (this renders as a pop-up window with selectable options), one per turn — each is its own standalone AskUserQuestion call, and each of 2b and 2c is only asked once the previous one's real answer has come back.

**Step 2a — role:**

**Question:** "What's your role?"

Options:
- Financial Advisor
- Executive
- Customer Care
- Operations

Anything else comes in via "Other."

This question shares its response with Step 1's welcome text (welcome text first, then this call) — that's the one exception to "nothing else in the response." Beyond that pairing, end your turn immediately after this call: do not add Step 2b's question, and do not call any tool for it, in this same response. Wait for the advisor's actual answer to come back before doing anything else.

**Once Step 2a's answer arrives:** if it's anything other than Financial Advisor — Executive, Customer Care, Operations, or Other — briefly acknowledge with one warm, friendly line that this plugin is built specifically for financial advisors, e.g. "Just so you know — this plugin's built specifically for financial advisors." Then continue straight into Step 2b in the same response. If they picked Financial Advisor, skip this and go straight to Step 2b as usual.

Say the line once — this is not gatekeeping, so don't suggest they're in the wrong place, don't ask why they're here, and don't hold up the flow to dwell on it. Don't repeat it later in the session even if their role comes up again.

**Step 2b — registration:**

Once Step 2a's answer has arrived, ask, as a separate, standalone AskUserQuestion call:

**Question:** "How is your firm registered?"

Options:
- RIA
- Broker Dealer
- Hybrid

Anything else comes in via "Other."

End your turn immediately after sending this question. Wait for the advisor's actual answer to come back before moving to Step 2c.

**Step 2c — affiliation (optional):**

Once Step 2b's answer has arrived, ask, as a separate, standalone AskUserQuestion call:

**Question:** "Do you work under a broker-dealer or a large advisor group?"

Options:
- Broker Dealer
- Large Advisor Group

Anything else (including "neither" or "independent") comes in via "Other."

This is optional: if the advisor picks Other and says it doesn't apply, skips it (`[No preference]`), or otherwise signals no affiliation, accept that and move on without pressing further. End your turn immediately after sending this question. Wait for the advisor's actual answer to come back before moving to Step 3.

Remember their role, registration type, and any affiliation — weave them into recommendations later in onboarding and beyond (e.g. registration type shapes which compliance considerations are relevant). What the advisor tells you about their firm, here or at any later step, is confidential — use it to shape this session, and don't repeat it anywhere it wasn't needed.

## Step 3: Learn their headaches

Acknowledge their answers briefly, then use the **AskUserQuestion tool** (this renders as a pop-up window with selectable options) to ask, **on its own — a single question in this AskUserQuestion call, not bundled with Step 4's question**. Set **`multiSelect: true`** — most advisors have more than one headache, and forcing a single pick here undersells how much of their day this plugin can actually help with:

**Question:** "What are your biggest day-to-day headaches, the things that eat your time the most?"

Ask about four felt headaches, not one option per skill.

Options, each with its fixed subtitle as the AskUserQuestion `description`:
- Prepping and writing up client meetings — subtitle: "Agendas, talking points, notes, and follow-up tasks"
- Keeping portfolios on target — subtitle: "Drift, rebalancing, and alternative investments"
- Compliance and paperwork — subtitle: "Getting client-facing material reviewed and approved"
- Wrangling data and catching errors — subtitle: "Messy files and details that don't match across systems"

The subtitles hint at the work each option covers, in advisor language — never name a skill or slash command in them (the advisor hasn't been introduced to skills yet; that's Step 6), and use them verbatim. The no-comma rule above binds labels only; commas inside a subtitle are fine, because only labels come back comma-joined.

**Do not generate one option per available skill.** The plugin has more skills than AskUserQuestion allows options, so a per-skill list is rejected outright and the advisor sees no question at all. Each option is a felt headache, not a skill name, and each maps to one or two skills: meetings → pre-meeting and post-meeting; portfolios → portfolio-rebalance-review and alts-brief; compliance → compliance alone; data wrangling → prospect-intake and estate-and-tax-brief. Together they cover every skill in the plugin, and Step 6b still delivers the complete, generated skill list to anyone who wants it — so nothing is hidden by asking at this grain.

Anything else comes in via "Other." The advisor can select as many as apply — don't ask them to narrow to just one.

Remember all of their selections — you'll use them to prioritize which skills to demo and recommend later (Step 8 picks the single best next skill from this set, but capture the whole list here). This tool call must contain only this one question. Wait for the advisor's actual answer to come back before doing anything else — do not show Step 4's form, or ask Step 4's question, in the same response that contains this one.

## Step 4: Learn their tech stack

Ask about their whole tech stack as **one question**: the checklist form below where the session can show it, and one plain message where it can't. Never as a run of AskUserQuestion pickers, one per category — that is the many-question flow this step replaces.

**The roster lives in one place: `assets/tools-checklist.html`, in this skill's folder.** Its groups, their order, and the tools under each are the whole list of systems Step 4 asks about, and both ways of asking read it from that file. Never retype a tool from memory, add one, or leave one out.

Don't re-cut the groups, merge two that look adjacent, or drop a tool because no other skill happens to name it — Step 5 still offers to connect it.

A few tools are listed under two groups on purpose: an advisor who thinks of one under either heading should find it there. A tool picked under both is one system — don't treat the second pick as a new one, and don't ask them to reconcile it.

Which of these ship a bundled connector, and which of those the public connector directory lists, is not recorded here: Step 5 sorts every picked tool by what the session observes, never by name, and its table of roster names served by a differently named connector says which connector to observe, not what the observation will find.

**Where the session can show the form** — `mcp__visualize__show_widget` is in your tool list, or a `ToolSearch` for it finds it where the session offers one — do this in one response:

1. Call `mcp__visualize__read_me` with the `elicitation` module.
2. Read `assets/tools-checklist.html` and call `mcp__visualize__show_widget` with its contents, verbatim, as the widget code.
3. End your turn, writing nothing after the form, not even a line pointing to it or to its Continue button: the form carries its own heading and Continue button, and `show_widget`'s result only confirms the form is on screen, so there is nothing to answer. Any acknowledgement of their Step 3 answer comes before the `read_me` call, or is left out. The advisor's answer arrives as their next message.

Show the form exactly as the file holds it, with nothing added, removed or restyled: no logos, icons, subtitles or extra text boxes, no "None" or "Other" choice, and no Skip button. The time the advisor waits for the form is almost all the time it takes to write it out, so every addition makes it slower to appear; and a Skip button sends nothing back to you, so the flow stalls with nobody knowing why.

**Reading the form's answer.** It comes back as one line: `Which tools do you want to connect? — ` followed by each group the advisor picked from, with that group's picks, the groups separated by ` · `. The group names in it are rebuilt from the form's field names rather than its group headings, so their capitalization and punctuation differ from the form's wording; match each group by what it names. Read each group's picks against that group's labels in the template. No label holds a comma, so a comma between picks always separates two of them.

The last group is the free-text box. Read it the way you read anything the advisor types: a tool on the roster named there counts as picked, under every group the template lists it in, exactly as if they had ticked it; anything else is a tool their firm uses that the roster doesn't list, and goes to Step 5 with the rest, to be looked for.

`Which tools do you want to connect? — proceeding with defaults.` means they picked nothing and typed nothing. That is a complete answer, not a gap: don't show the form again, don't ask the question another way, and go on to Step 5 with no tools picked.

If they type their answer into the chat instead of using the form, read that message the way a plain-message answer is read (below), and don't show the form again.

**Where the session can't show the form** (no `show_widget`), ask the same question as one plain message: a lead question asking which of these tools their firm uses, then each of the template's groups in its order, by its name, with that group's tools listed under it, then a last line asking for anything else their firm uses that isn't listed. It is one message and one question — no AskUserQuestion call, and never the groups as separate questions. Keep it to the groups and their tools, with no commentary on any of them.

The advisor answers in their own words. Read their reply the way the form's free-text box is read: each roster tool it names counts as picked, under every group the template lists it in; anything else is a tool to look for at Step 5. A reply that names nothing they use ("none", "nothing yet") is a complete answer, like the form's empty one: go on to Step 5 without asking again.

Either way, end your turn once the question is asked, and wait for the advisor's actual answer before moving to Step 5.

Remember every tool they picked as one combined set, and which group each came from — Step 5 handles all of them together, and Step 8 reads the groups.

## Step 5: Connect their tools

If they picked nothing at Step 4 — an empty form, or a reply naming nothing they use — there is nothing to sort or connect: say in a line that they can connect their tools whenever they like, from their connector settings, and go straight on to Step 6 without waiting. Otherwise, say something like:

> "Ok — let's start connecting your tools now."

Sort every tool they selected in Step 4 into exactly one of the groups below before handling any of them. The sort is decided by what the session observes — never by a fixed list of which tools belong where. This file does not know which connectors are live, bundled, or listed in the public directory.

**Some roster names are served by a connector with a different name.** Before anything else, replace each roster name below with the connector that serves it, then search and sort for that connector, not for the roster name:

| Roster name (Step 4) | Served by the connector |
|---|---|
| FactSet | FactSet AI-Ready Data |
| MSCI | MSCI Connector |
| Orion Connect | Orion Advisor Solutions |
| Redtail CRM | Orion Advisor Solutions |

This table says which connector serves a name — never whether that connector is live, bundled or listed, which is still only what the search below decides. A roster name it doesn't hold is searched and sorted under its own name. When the advisor hears about a connector reached this way, name the system they picked alongside it, so they can tell it's theirs ("Redtail CRM comes in through Orion Advisor Solutions"); two picked names served by one connector are sorted and handled once.

**The registry search is the observation.** Call `SearchMcpRegistry` (load it, and `SuggestConnectors`, via `ToolSearch` if they aren't already available) with the selected names, after the table — up to eight names per call, so as many calls as the set needs, never one per tool. Each result it returns carries the connector's id and its install state for this advisor.

- **Keep only the results that name a selected system.** The search is fuzzy: a name can bring back a connector that merely resembles it, including another system on the Step 4 roster that the advisor didn't pick. A result is a match for a selected system only when the connector it names is that system, or the connector the table above maps it to; every other result is dropped — never suggested, and never mentioned.
- Until a connector is connected, nothing about it reaches this session — no tools, no context — so for a system the search answered there is nothing else to look for: its install state already says whether it's connected. A system the search couldn't answer for may be connected already, so before sorting it, look for its tools the way Step 6 does: in your tool list, or with a `ToolSearch` by the system's own name where the session offers one, and count it as connected only when the tools you find belong to its own connector. A tool from any other connector says nothing about this system, and finding none says nothing either, since a system not yet connected has none to find. Once the advisor says they've tried to connect, whether it took is Step 6's check, made the way Step 6 describes. And don't list the advisor's installed connectors at this step: the install state already says whether it's connected, and the installed-connector list renders a card of everything the advisor has, which is not the set they just named.

Then sort each selected system — the first group that fits wins:

- **Connected** — its match's install state says it's connected, or the search couldn't answer for it and you found its own connector's tools. Say so in a short line; it needs no group below, so no connect instructions and no other group's line. The step still ends with its stop (below), even when every system the advisor named is connected.
- **A** — it has a match that isn't connected yet. That includes a connect flow that was started and not finished (`connect_incomplete`), which is simply ready to connect, per the rule below.
- **B** — the search answered and returned no match for it. That is the one positive evidence of absence this step has.
- **C** — the search couldn't be made or didn't answer for it: `SearchMcpRegistry` isn't available, it returned `opt_in_required`, or the call failed. Nothing was observed about the system either way, so it's unknown — never absent.

The sort is the only thing that decides what you tell the advisor about each tool, and the sort itself stays yours: the advisor hears each group's line, which is about the system, and never how you sorted it — not a group's name (Connected, A, B, C), and not what either search was, returned or couldn't do, apart from the opt-in line under **Fallbacks** below. A connected system's line may say you can see into it now; that is its status, not a search result. Never report a tool as already connected or as unavailable unless one of the searches above put it there — a status nothing observed is a fabricated one. An unknown tool is never sent to B: telling the advisor a working connector doesn't exist, because a search that couldn't run found nothing, is false.

**Authorization is not a status to report here.** Connecting what isn't connected yet is what this step is for, and until a connector is connected nothing about it reaches this session, so a system that isn't connected yet is not in a failure state: it is ready to connect, and what the advisor hears about it is its group's line below. Once the advisor starts connecting, whatever the session can see about a connector's sign-in, during the attempt or after it — an authorization error, a failed or incompatible sign-in, a connector waiting to be authenticated — never reaches the advisor during onboarding, not per connector and not as a summary. There is nothing for them to act on in it. A connect flow that didn't finish is raised only by Step 6's offer to give it another go, which names no error and no reason.

A tool the advisor named that the Step 4 roster doesn't list is looked for too, exactly as the roster's are: it goes into the registry search with the rest, and where the search couldn't answer for it, its own tools are looked for. Found connected, or with a match, it is sorted and handled like any other system. Otherwise — no match, or no answer and none of its own tools found — it is none of these groups: see **Fallbacks** below.

Where a connector comes from — bundled, shipped with or registered by this plugin, or listed in a directory — is packaging, which is never part of what the advisor is told, in any group.

**A. Ready to connect — one panel:**

Call `SuggestConnectors` **once**, with the ids of every system in group A, so the advisor sees one "Connectors that could help" panel holding exactly the systems they named, each with its own **Connect**. Clicking **Connect** starts that system's sign-in. Never one call per system, and never add a result the search brought back that the advisor didn't pick. Before the panel, say in a line which of their systems it holds, naming any the table mapped alongside their connector.

**B. No match in the search:**

Only mention this for tools the advisor actually selected in Step 4. No widget, no demo, no clickable mockup — there's nothing for the advisor to click. Just say so plainly, in the same warm tone as the rest of onboarding:

> "[Tool] can't be connected from here. You can always upload an export from it or paste data in whenever a skill needs it."

Never give a reason for it — an admin, an organization setting, a plan, a roadmap — because the reason differs from one account to the next and the session can't see which applies. Don't promise to notify them if that changes — this skill has no mechanism to follow up later, so don't commit to one.

**C. Unknown — the search couldn't answer:**

Tell them it's ready to connect and where to connect it, naming it as the Step 4 roster does — or, for a roster name the table above replaced, by the connector that serves it, since that is the name they will be clicking.

> "[Tool] is ready to connect. You can connect it from the Claude for Financial Advisors page, in its Connectors tab, or by searching for it in your connector settings. Click **Connect** next to [Tool] and sign in with your [Tool] account. Once that's done I can pull from it directly whenever a skill needs it."

Both routes reach the same connector, so the line names both. Describe them by what they are — Claude for Financial Advisors' own Connectors page, and the ordinary connector search — rather than by a menu path, which changes whenever the interface does.

When the search couldn't be made at all, every selected system whose own tools didn't come back lands here, and the line above is still said as written: not hedged into "check whether it shows up" or "if it's there you can connect it", and with nothing ahead of it about either search, the registry search or the name search — not that you couldn't look a status up, couldn't show a panel, that no tools came back, or are going by anything else instead, and not the group's name. The Convention's unknown row ("say you couldn't confirm its status") is written for a skill that needs the data mid-task; at this step connecting the system is what the advisor is here to do, so there was never a status to confirm, and a hedge or a preface sends the advisor off unsure what they are looking for. The line is not a guess about this session. Why it holds is where the connector comes from, which is packaging and never part of what the advisor is told (above). The case where they can't connect it has its own line below.

If they come back and can't connect it from either route — it isn't there, or it shows as something they can't add — that is the observation the session was missing: say plainly that [Tool] can't be connected from here and offer the upload-or-paste fallback, exactly as for group B: no reason for it, and no promise to follow up. Never improvise a URL for it — the two routes above are where it is.

**Fallbacks:**
- If registry search returns `opt_in_required`, tell the advisor: "To let me suggest connectors, enable connector suggestions in your Claude settings — or you can connect tools directly by searching for them in your connector settings." Then continue the flow.
- If the advisor names a tool that isn't on the Step 4 roster and the search brought back no match for it — a genuinely unlisted or custom in-house tool, not one of the tools sorted above — mention briefly: they can tell their account manager at that vendor to begin creating an MCP, or set up a custom MCP themselves (MCP is an open standard that lets Claude securely talk to other software — https://modelcontextprotocol.io). Where the search couldn't answer for such a tool, nothing says it has no connector, so open with the ordinary route instead: they can search for it by name in their connector settings, and if it isn't there, those two options apply. If asked, give a plain-English explanation of MCP and walk them through setting up a custom one, which they should do with their firm's IT or compliance team, since it will give Claude access to that system's data.

After presenting the connector panel, the direct-connection instructions, and/or any status lines, stop here — don't continue into Step 6 in the same message. That holds when every system the advisor named is already connected and nothing is left to set up: the Connected lines are this step's whole message, and the advisor still says when to go on. Say something like: "Take your time — click **Connect** whenever you're ready, or just let me know when you want to keep going." When nothing is left to connect, leave out the part about clicking **Connect**: "Let me know when you want to keep going." Then wait for the advisor's next message before moving on. Rendering the panel is not the same as the advisor being done with it.

## Step 6: About the plugin

Once the advisor confirms they're ready to move on (they've clicked Connect, said they're done, or asked to continue) — not simply once Claude has finished describing their tools, unless Step 5 had nothing to connect — check again what's actually connected now, the way Step 5 did: repeat its registry search for the systems that weren't already connected (one Step 5 found connected counts as connected below, with no second look), and read each match's install state — never by listing the advisor's installed connectors, which renders their whole card. Where the search can't answer, look for the system's tools in your tool list, or with a `ToolSearch` by the system's own name where the session offers one: once a connector is connected its tools reach the session. Count the system as connected only when the tools you find belong to its own connector. A `ToolSearch` matches loosely and can return another connector's tools for a name that merely resembles them, and a tool from any other connector says nothing about this system. Don't just assume based on having called `SuggestConnectors` earlier — the advisor may not have finished, or may have only connected some of what was suggested. Then deliver this script. Keep the paragraph breaks — short paragraphs with blank lines between them. The second paragraph depends on what that check shows:

Read that check on each match's install state, and pick **one** of these three. The second exists because the advisor was just told to click **Connect** — a flow they can leave part-way through, which is an ordinary thing to do and leaves a connector reporting neither connected nor absent.

- If at least one tool is now genuinely connected: "Thanks for connecting your tech stack — I can now see into [name the connected tools]. I won't take any action there unless you tell me to."
- **If a tool's install state says a connect flow was started and not finished (`connect_incomplete`), or the search couldn't answer for it, its tools didn't come back, and the advisor says they tried to connect it** — the signature of a connect flow that was started and not finished: name it and offer the next step, don't count it as unconnected. "It looks like [tool] didn't quite finish connecting — want to give that another go? I can wait." If they'd rather move on, say so warmly and continue; never make this a gate.
- If nothing is connected yet (none of the selected tools came back connected, at Step 5 or in this check, whatever the reason): "Thanks for walking me through your tech stack — once your tools are connected, I'll be able to see into them and pull real data automatically."

**Where something is now connected, show what it offers.** The plugin keeps a provider file for some systems, on how to ask that system, and one may offer an example to show a new advisor: one that needs no client data. For each system now connected, whether Step 5 found it so or this check did, list `${CLAUDE_PLUGIN_ROOT}/providers/`, and if it holds a folder named for that system, read every file in it. Where one offers such an example, show it right after the line above and before Step 6a, as the file says: every step it gives, in order, with none skipped. Show one in all, and if more than one connected system offers one, choose the one closest to the headaches they picked in Step 3: it is a taste of what connecting gives them, not a tour of any one system. Never ask the advisor for anything to show one. If the system returns an error, or nothing the file says to show, leave the example out and say nothing about it. A system that isn't connected shows nothing, whatever its file offers, and no example is ever made up in a system's name.

**Step 6a — what a skill is, then ask:**

> "Now let me explain a bit about myself.
>
> I come pre-installed with a set of "skills." A skill is how you can have me complete entire workflows for you.
>
> For example, my pre-meeting skill builds a complete quarterly-review prep doc for any client you name — pulling together their history, holdings, and recent activity from your connected systems into a client snapshot, performance tables, planning opportunities, a time-boxed agenda, talking points, and draft action items. Something you can read in 10 minutes and walk into the meeting confident.
>
> Skills are also flexible — if you ever want one to work differently for you, just say so and I'll walk you through personalizing it.
>
> Want me to explain the [X] other skills in this plugin?"

Keep that example's sources generic on purpose — "their history, holdings, and recent activity from your connected systems," never "their CRM record, portfolio performance, and recent correspondence." Naming specific systems reads as a claim about *this advisor's* setup, and Step 5 may have just told them those very systems aren't connected yet. "Your connected systems" describes how the skill works rather than what this advisor happens to have, so it stays true either way and needs no connected/not-connected branch here. Don't substitute the names of whatever came back connected, either — that would reintroduce the branch this phrasing exists to avoid.

Calculate **[X]** — don't hardcode it, for the same reason the list itself is generated: it's the number of skills available in this plugin right now, minus onboarding and minus pre-meeting (already described just above, which is what makes the rest "other"). Count placeholder/"coming soon" skills in the total — they're still listed in 6b, so excluding them would promise fewer items than you go on to name.

End your turn here and wait for a real answer. This is a genuine offer, not a rhetorical lead-in to the list — asking and then reciting the list anyway in the same response is worse than not asking. Ask it as plain chat text, not AskUserQuestion; it's a yes/no aside, not another pop-up.

**Step 6b — the list, only if they said yes:**

- **If they say yes** (or otherwise signal interest): deliver the list per the rules below, then go to Step 7.
- **If they say no** (or "later," or "let's just get going"): skip the list entirely — don't summarize it, don't name a few highlights as a consolation — and go straight to Step 7. Don't treat the decline as something to talk them out of or circle back to.

If they said yes, open with:

> "Here is a full list of the skills in this plugin:"

Don't hardcode this list — generate it from the skills actually available to you in this plugin right now, so it stays accurate as skills get added, renamed, or removed. For each one:
- Name it as a slash command (`/<skill-name>`).
- Summarize what it does in one short, warm, benefit-focused sentence, in your own words — don't paste the skill's frontmatter `description` verbatim. That field is written dense and keyword-heavy for triggering purposes, not for a first-time advisor to read.
- If a skill's description flags it as a placeholder (e.g. starts with "PLACEHOLDER" or otherwise says it isn't built yet), list it as "coming soon" instead of describing capability it doesn't have. The same holds for any question about what a skill can do, here or later: if it can't do something, say so plainly rather than describing a capability it doesn't have.
- List pre-meeting first regardless of discovery order — it's the one you demo next in Step 7, so it should lead naturally into the invitation below.

If the advisor left any Step 4 tools unconnected that can still be connected (connectable but not yet clicked), you may tease it here in one line — e.g., "Once the rest of your stack is connected, I'll pull that data live instead of asking you to paste or upload it." Keep it to a single sentence, don't repeat Step 5's status report, and skip this entirely if every tool they selected is already connected or can't be connected from here.

Then hand off into Step 7 without waiting again, using the line that matches the branch you took:

- **If you delivered the list:** "Now that we have that covered, here's where I'd start..."
- **If they declined it:** "No problem — you can always ask me for the full list later. Here's where I'd start..."

Don't use the "now that we have that covered" line on the decline branch — nothing was covered, and it reads as though you recited the list anyway.

## Step 7: Live demo — pre-meeting

Offer the demo:

> "Give me a client name and we can do a quick meeting prep — or if you'd rather not use real client data on the first run, I can use a fictional sample client, **Jordan & Casey Miller**."

A client name is all this asks for. Never ask for account numbers, contact details, or any other client PII the advisor hasn't volunteered — here or anywhere else in onboarding — and treat whatever they do share about a client as confidential.

The answer decides the path — and the real-versus-fictional split below is a guardrail, not a formality:

- **A real client name → execute the `pre-meeting` skill end to end** so they see a real deliverable against their own data. Where connectors aren't yet live, the placeholder convention below will kick in — that's expected and is itself a useful preview. (If the advisor typed /pre-meeting with a client name, that skill handles it.)
- **The fictional Millers → the demo stays entirely inside this skill.** Never invoke `pre-meeting` — or any other skill — with a fictional client, and never query any connected system for the Millers. Other skills treat every household name as a real client (searching for it, disambiguating it, refusing to fabricate about it); that is a guardrail, not an inconvenience, and a fake name doesn't go through it. Don't search connected systems for the Millers, and never offer a real household as a substitute — the advisor chose the fictional client precisely to keep real client data out of the first run. Instead, present the sample prep document at `references/miller-demo.md` (in this skill's folder) as the deliverable, walking through it in the same voice `pre-meeting` would use. Every Miller figure and detail comes from that file; don't invent one it doesn't hold, even to answer a follow-up question — say the sample doesn't cover it. Where the advisor connected systems in Step 5, you may note in-line "on a real client, this section pulls live from [system]" — but read nothing from those systems for this demo.
- **Not interested in a demo → skip straight to Step 8, politely.** One warm line ("No problem."), no talking them out of it, no consolation demo.

## Step 8: Closing recommendation

Based on their Step 3 headaches (and what you learned about their firm in Step 2), suggest the single next skill most likely to save them time this week (e.g., if they picked "Compliance and paperwork," suggest trying /compliance on a real draft email).

Three of Step 3's options map to a pair of skills, so pick the one within the pair that best fits what you learned in Step 2 and what they actually connected in Step 5: meetings → `/pre-meeting` vs. `/post-meeting` on which side of the meeting they described; portfolios → `/portfolio-rebalance-review` vs. `/alts-brief` on whether they use an alts platform — anything they picked under the alternatives group, or Addepar under portfolio management, which they may not have ticked a second time; data wrangling → `/prospect-intake` vs. `/estate-and-tax-brief` on growth signals from Step 2 vs. anything they picked under the financial, tax and estate planning group. Name the skill you pick and no other, so they have one thing to try and no second to weigh.

## Connector Placeholder Convention (applies to every Claude for Financial Advisors skill)

The canonical list of systems this plugin supports is onboarding's Step 4 roster, `assets/tools-checklist.html` in the onboarding skill's folder — that's the actual source of truth, not this section. In any Claude for Financial Advisors skill, when a step calls for data from one of those systems:

1. **Look for the tools first — that is the check that decides.** A system whose tools are in your tool list, or come back from a `ToolSearch` by the system's own name (Orion, Gmail, Wealthbox), is connected and callable: use them and move on. Where the session offers `ToolSearch`, a system's tools can be held back until it loads them, so search by name for any system whose tools aren't already in your list; where it offers none, nothing is held back, and your tool list is every tool you have. Don't guess at a tool-name prefix like `mcp__orion__*`; real connector tool names use opaque, unpredictable prefixes, not clean names based on the system, which is why you look for them by the system's own name.

   **This ordering is deliberate.** The registry can answer *"No installed connectors found"* even in a session with several live, working connectors. Followed literally, that answer would declare every system unavailable and degrade the entire session to manual fallback. The tools you can actually see are a direct observation; the registry is a report about them, and in some clients it renders a user-facing card rather than returning data at all.

2. **`ListConnectors` explains a gap; it never establishes one.** Once you know from step 1 that a system's tools are missing, call `ListConnectors` (load via `ToolSearch` if it isn't already available) to find out *why*, so the advisor gets a useful sentence instead of a shrug.

   **An empty result means unknown, never "nothing is connected."** So does a result that renders a card instead of returning data. Neither is evidence of disconnection, and neither may be used to trigger the placeholder wording below.

   **When it does return entries, read them on two fields, not one.** `ListConnectors` reports `connected` (authenticated at the org level) and `enabledInChat` (whether its tools are actually loaded in this session), and they disagree often enough to matter. There are three outcomes, and each has a different thing to say:

   | What comes back | What it means | What to do |
   |---|---|---|
   | `enabledInChat: true` | Its tools are loaded and callable now. | Nothing to explain — step 1 should already have found them. If it did not, look again by the system's name. |
   | `connected: true`, `enabledInChat: false` | Authenticated, but switched **off for this chat**. Its tools are not loaded, so step 1 finds nothing. | Tell the advisor it's connected but not enabled here, and that they can turn it on in this chat's connector settings. **Do not** say it isn't connected, and do not offer the placeholder wording below. |
   | `connected` absent or `null` | **Unknown**, not disconnected — the status check was unavailable. | Say you couldn't confirm its status rather than asserting it isn't connected. Then offer the manual fallback in step 3. |

   Never read a missing `connected` as `false`, and never decide a system is unusable from `connected` alone. `enabledInChat` is what governs whether you can call anything.

3. **If the connector genuinely isn't there** — not installed — say so transparently, in-line at that step:
   > "This is where I'd pull [the specific data] from [the system], but it isn't connected in this session. You can connect it from your connector settings, or paste, upload or skip this section."

   Only use that wording when it's true. For a connector that is switched off, or whose status is unknown, it misdescribes the advisor's own setup.
4. Then **offer the manual fallback**: the advisor can paste the data, upload an export (CSV/PDF/screenshot), or skip the section. Continue the workflow with whatever they provide — never dead-end.

## Ask-Once, Then Route Convention (applies to every Claude for Financial Advisors skill)

Sometimes two connected tools can try to solve the same problem for the advisor. That overlap isn't defined by the tools being the same kind — two CRMs, two portfolio platforms, two alts platforms are the common case in this plugin today, but a CRM and a portfolio platform (or any other pairing) can overlap just as easily if both happen to hold the same fact. What matters is whether they'd give the advisor different answers to the same problem, not what category each tool falls in. When that happens, every skill follows the same rule:

1. **Resolve the household first, in one system.** Identify it per that skill's own Disambiguation Rule, using one of the systems that overlap — a name search or identity lookup, nothing that returns the household's data. Don't search the second system to find out which of the two knows the name. Before that search, say which system you're looking in and that the book-of-record question comes as soon as it returns.
2. **If exactly one of the relevant systems is connected, use it** — there's nothing to ask.
3. **If more than one is connected, ask the advisor once which is the book of record for this household — as soon as that first search returns, and before you search, pull from, or merge anything from a second system.** Never guess, never default to whichever answers first, and never silently merge two systems' answers into one undifferentiated view. Name the systems in the question ("Orion or Addepar — which is the book of record for the Harrington household?"). A name that doesn't resolve in the first system is not a reason to go looking in the second: ask then, and put the name in the same question ("I couldn't find a Lindqvist household in Addepar — is Orion their book of record, or do they go by another name?"). The question comes when that first search returns, whatever it returned — a match, several candidates (they go in that same question), nothing at all, or an error you can't read — before any other call to either system. "It isn't in this one, so let me check the other" is exactly the search this rule forbids: which system holds the household is the advisor's to tell you, not yours to find out by searching.
4. **Route there for the rest of the session** — pull only from that system, for this household, once they answer.
5. **Offer the standing-preference paste** (below) so they aren't asked again next session for the same household — in the reply that takes their answer, before the first call to the system they named.

This order — identify in one system, ask, then pull from the one they name — is the same in every skill. Where a skill's overlap isn't about one household — compliance's lookups, prospect-intake's CRM for a new prospect — it asks the same way, scoped to what the lookup is about, and its paste is scoped the same. Otherwise this is a per-household choice, not a firm-wide setting, so the question belongs in-skill, at the first step that hits the overlap — **never during onboarding**: at that point neither which households have overlapping data nor which connectors the advisor will finish connecting is known yet.

**Defined routes are not overlaps.** This governs *which system to trust when more than one tool could try to solve the same problem*, not *which entity the data is about*. Some skills fix the route between two sources themselves, because what the sources are decides it: pre-meeting and post-meeting treat the CRM's household record as the book of record and the meeting-intelligence tool as color on it; post-meeting takes the meeting's content from the first source in its waterfall; alts-brief labels each alts platform's figures side by side; estate-and-tax-brief cross-checks titling across the custodian, portfolio and alts platforms. Where a skill names a defined route, follow it and don't ask — it's the sanctioned answer to the question this convention would otherwise put to the advisor. Any overlap a skill doesn't name that way is asked about, and the material-conflict rule below still applies to every defined route.

**The overlap can surface at any point, not just at the start of a step.** Treat any source that could hold the same kind of information as one already in play as an overlap — including a source the advisor mentions on the fly ("I also have notes in X"), not just a second system already named in this skill. This check isn't one-time: re-run it whenever a new source enters the conversation, even mid-gathering. Before merging its content in, stop and ask which is authoritative for that content type.

### Material conflicts block the output

Separately: if two sources are pulled anyway — a defined route like alts-brief's label-each handling, or an overlap that wasn't caught in time — and they disagree on a material fact (a dollar figure, an allocation target, a date) as of the same date, or on one that carries no as-of date of its own, do not resolve it narratively or present both as a footnote. Stop and surface the conflict to the advisor as a blocking question before the document is finalized, the same way the Disambiguation Rule blocks on a name collision. Figures as of different dates are lag, not a conflict: show each, labeled with its source and its own as-of date, and don't block on the difference. Lag never explains figures apart by orders of magnitude, whatever their dates: that is the Magnitude-Conflict Convention below.

### Standing-preference paste

Once the advisor answers, offer to save it the same way the Personalization Convention below saves any other preference — tell them where to go and write the exact text to paste. The offer opens the reply that takes their answer, before your first call to the system they named, even when their answer also hands you a different name to search:

> Want me to remember that for [household] so I don't ask again? Paste this into **Settings → General → Instructions for Claude**:
>
> For the [household] household, treat [system] as the book of record for [data type]. Don't ask me to choose again for this household.

Never write to those settings on the advisor's behalf — Claude has no ability to. Each skill that can hit an overlap carries this same paste at the step that asks, so it doesn't depend on onboarding having been read.

## Magnitude-Conflict Convention (applies to every Claude for Financial Advisors skill)

Ask-Once decides *where to read*; this rule governs *what may be quoted*. When two connected systems return the same household or account at figures that disagree by orders of magnitude, the outlier is a data conflict, not evidence:

- **Name the conflict in the deliverable** — both figures, both sources, one line saying they cannot both be right.
- **Exclude the outlier's figures from every table and total.** A number flagged as wildly inconsistent doesn't earn a row by being flagged; quoted-with-a-caveat still reads as evidence.
- **Confirm with the advisor which record is real** before any follow-up relies on either.

The common cause is a placeholder or sample household — a near-empty book under a real client's name in a system the advisor didn't name as book of record. Those stubs are nobody's data: never source a figure from a household the advisor hasn't confirmed as theirs, even when a name search returns it first.

## Personalization Convention (applies to every Claude for Financial Advisors skill)

Claude cannot write to the advisor's settings — when an advisor asks to personalize how a skill behaves (e.g., "always skip the executive summary," "use tables instead of bullet points," "leave firm insights out of my prep docs"), walk them through doing it themselves:

1. Tell them exactly where to go: **Settings → General → Instructions for Claude.**
2. Write the exact markdown for them to paste — scoped to the specific skill(s) and behavior they asked to change, short (a bullet or two, not a paragraph). These instructions persist across every conversation, not just this plugin, so don't write anything broader than what they actually asked for.
3. Never claim to have made the change yourself, and never write to those settings on their behalf — Claude has no ability to.

## Summary-First Convention (applies to the Claude for Financial Advisors skills named at the end of this section)

Those skills each write an advisor-facing analysis document, and each answers in chat with that document's own opening, never with a second summary written for chat:

1. **The deliverable's Executive Summary is the top of the file** — above the tables, notes and flags, under that heading in those words. A summary at the bottom is one the advisor has to read the whole analysis to reach.
2. **The chat reply opens with that same Executive Summary** — the same sentences, not a paraphrase and not a fresh one. Two summaries of one document is the failure this prevents: the advisor reads the chat version and relays it, and it isn't what the file says.
3. **Then two to four directions to go deeper**, named plainly enough that the advisor can take none, one, or several.
4. **Only offer directions this run can actually deliver.** A step that didn't run has nothing waiting to be shown — offer it as something that *could* be run, or leave it off the menu entirely. Offering a direction and then withdrawing it is worse than never naming it.
5. **The directions and the delivery question are one closing set, with nothing between them**, and the reply ends at the offer. A skill that owes the advisor questions of its own — what a missing source still needs, a default it had to assume — puts them in that same set rather than before it or in a reply of their own. A blocking question is the one exception: a material conflict stops the document (Material conflicts block the output, above), so its question comes before the set, and the delivery question waits for the answer. The tables stay in the file; don't walk the advisor through what they can read. Anything this run wrote to a connected system is reported *after* the set — a report, not a request.

**Where a run has nothing to go deeper into, the set is shorter rather than padded.** A review whose reads all came back empty or `NOT CONNECTED` offers what the Connector Placeholder Convention already owes it — the manual fallback — and stops there. Two directions is a floor for a run that produced something, never a quota to be met by inventing a direction that leads nowhere.

The skills that follow this — portfolio-rebalance-review, pre-meeting, estate-and-tax-brief and alts-brief — each state it at their own output step, so it doesn't depend on onboarding having been read. A skill whose close its own step governs differently is not covered: compliance opens its chat summary by saying what the review is, before any verdict, and that sentence comes first.

## Important Notes

A summary; each rule is stated in the step it governs.

- Keep the tone warm and unhurried, and let the advisor skip ahead — the opening of this file.
- Break every long scripted block into small paragraphs with blank lines between them — no walls of text (the formatting rule at the top).
- Never ask for client PII during onboarding beyond what the advisor volunteers (Step 7).
- Anything the advisor shares about their clients or their firm during onboarding is confidential (Steps 2 and 7).
- Never fabricate connector status, a connected system's example, skill capabilities, or demo data (Steps 5, 6, 6b and 7).
- Don't ask "should I start?" once the advisor has invoked onboarding — narrate and go (Step 1).
