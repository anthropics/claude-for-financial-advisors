---
name: compliance
description: Review any client-facing message or material (emails, letters, newsletters, social posts, website copy, presentations, performance reports, proposals) for compliance with SEC rules that govern investment advisers — the Marketing Rule (206(4)-1), fiduciary duty / antifraud provisions (206), Books and Records (204-2), and Reg BI where broker-dealer activity applies. Produces a pass/flag/fail markup, required disclosures, a books-and-records scratch-pad entry, and an archiving handoff reminder. Triggers on "compliance check", "/compliance", "is this compliant", "review this before I send it", "can I say this to a client", "check this email/post/deck". Use it whenever an advisor asks to check, review, or look over anything before it goes to a client or prospect — "check this before I send it", "look this over before it goes out" — even a routine note, because deciding that a message needs no changes is part of the review, not a reason to skip it.
---

# Compliance Marketing Review

This is a **pre-check** against SEC marketing rules, run **before** content goes out — the goal is to improve a draft's odds with the firm's own compliance review, not to perform that review: catch violations, suggest rewording that addresses them, and leave a record.

This skill supports advisers, it does not replace them: output is a **draft review for the firm's CCO/compliance officer**, not a legal determination or an approval. Nothing here means the content "passed compliance" — it means the known problems were caught and fixed before a human sees it.

## Inputs

1. **The content** — pasted text, uploaded file, a draft Claude just wrote, or a draft already sitting in email or Drive. For the latter, `ToolSearch` by the system's own name — if its tools come back, search it for the draft. That search is the check that decides; `ListConnectors` (load via `ToolSearch` if not already available) explains a gap rather than establishing one, and an empty result from it means **unknown**, never "nothing is connected". If it isn't, follow the Connector Placeholder Convention — say *"This is where I'd search [system] for the draft once that connector is built"* — and degrade to asking the advisor to paste or upload it. Only use that wording when the connector genuinely isn't installed: `connected: true` with `enabledInChat: false` means it's switched off for this chat (say so, and that they can enable it here), and a missing or `null` `connected` means **unknown**, not disconnected.
2. **Context** (ask if unclear):
   - Audience: single client, prospect, or broad distribution (one-to-many)? — the Marketing Rule applies differently
   - Channel: email, social, website, print, presentation, text message
   - Firm type: SEC-registered RIA (default assumption), state-registered, or dual-registrant/BD rep (adds FINRA 2210 / Reg BI considerations)
   - Does it mention performance, testimonials, third-party ratings, or hypothetical/projected results?

## Review Workflow

Invoking `/compliance`, or handing Claude a draft to check, is already the go-ahead — begin the review, don't ask whether to start.

Work through `references/sec-compliance-checklist.md` (in this skill's folder) systematically. Summary of the passes:

**Passes 2 and 3 go to the `claude-for-financial-advisors:compliance-scan` subagent** — `Agent(claude-for-financial-advisors:compliance-scan)`. Hand it the content verbatim, the context you gathered (audience, channel, firm type, whether performance/testimonials/ratings/hypotheticals appear), and the **absolute path** to `references/sec-compliance-checklist.md` — the subagent's working directory is the session's, not this skill's, so a relative path won't resolve for it. It returns its own advertisement determination — reached independently, from the checklist and the content, not from anything you concluded in Pass 1 — plus the flagged passages quoted verbatim with rule, severity, and reasoning, which disclosure categories the content triggers, and which checks came back clean. Where its determination and your Pass 1 call disagree, yours governs the review — Pass 4's markup and the disclosure language are yours to write, so the call that drives them is yours to make — but never let the difference pass unremarked: note it in the analysis file, and treat the disagreement as one more reason to escalate to the CCO.

**Pass 4 stays with you, all of it.** The subagent finds and cites; it never rewords, never drafts disclosure language, and never redrafts. The suggested rewording column, the required-disclosure language, and the clean redraft that preserves the author's voice are the parts that need judgment about this firm and this author, and they are the reason this skill exists. Read the scan, then write the markup yourself — and check the flags rather than transcribing them: a severity you disagree with is yours to change.

**Once the scan comes back, resolve what it can't reach itself** — `compliance-scan` only has `Read`/`Grep`/`Glob`, so any check that needs a live connector is yours to dispatch, not its:

- **A testimonial or endorsement is present.** `ToolSearch` by the CRM's own name; if its tools come back it is callable, which is the check that decides. `ListConnectors` (load via `ToolSearch` if needed) only explains a gap — read `enabledInChat` rather than `connected` there, and treat an empty result as **unknown**, never as "no CRM". If a CRM is usable, dispatch one `claude-for-financial-advisors:compliance-lookup` subagent per named author (lookup type `identity`) — all in a single message if there's more than one. Hand each the one system it is to query, named (the CRM whose tools came back), and the author as the content names them — asking for client vs. non-client status, relationships to other households, and anything bearing on compensation or conflicts. Fold what comes back into Pass 3's disclosure requirements below. If no CRM is connected, follow the Connector Placeholder Convention and ask the advisor who the reviewer is to the firm — never assume an author is unrelated just because nothing surfaced.
- **The scan flagged a material claim of fact as unsubstantiated (Pass 2, item 2) and it's checkable.** If a portfolio system (Addepar, Orion, etc.) is connected and plausibly holds the comparison data, dispatch one `claude-for-financial-advisors:compliance-lookup` subagent per checkable claim (lookup type `claim`) — batched into one message when there's more than one. Hand each the one system it is to query, named (the portfolio system whose tools came back), and the claim verbatim with what data would settle it. Cite what comes back (contradicts, supports, or can't determine) in the verdict table rather than only flagging that the claim needs substantiation. Check against the actual population the claim is about — a claim about "clients in situations like hers" is checked against *comparable households*, not against the one client the piece is already about; citing that client's own number back at her own claim isn't verification. If no portfolio connector is connected, fall back to demanding substantiation as usual.

The passes, which are also what to work through by hand if the subagent is unavailable:

### Pass 1 — Is it an "advertisement"?
Determine whether the content falls under Marketing Rule 206(4)-1, using the Scope Determination test in `references/sec-compliance-checklist.md`: a direct or indirect communication to **more than one person** offering advisory services is an advertisement; a communication to **one or more persons** is an advertisement if it includes **hypothetical performance**, unless it answers an unsolicited request or goes to a private fund investor one-on-one; and any **compensated testimonial or endorsement** (cash or non-cash) is one whatever the audience size. The checklist's Scope Determination section states three exclusions: extemporaneous live oral communications, information in regulatory filings, and most one-to-one communications carrying no hypothetical performance — a reply to a client's or a prospect's own question, say. Antifraud rules under Section 206 still apply to everything, advertisement or not.

### Pass 2 — The seven general prohibitions (Marketing Rule)
Flag any statement that:
1. Contains an untrue statement of material fact, or omits a fact needed to make it not misleading
2. Makes a material claim of fact the adviser cannot substantiate on demand
3. Is materially misleading by implication or inference
4. Discusses potential benefits without fair and balanced treatment of material risks
5. Cherry-picks favorable investment advice/results without fair and balanced presentation
6. Includes or excludes performance in a manner that is not fair and balanced
7. Is otherwise materially misleading

### Pass 3 — Specific content rules
- **Performance**: net-of-fees shown at least as prominently as gross; 1/5/10-year (or since-inception) periods for time-weighted returns; no "SEC-approved" claims; hypothetical/projected performance only with required policies and audience-appropriateness; extracted performance needs the total portfolio context. Performance and hypothetical-performance content escalates to the CCO on its own, whatever severity it's rated.
- **Testimonials/endorsements**: required disclosures (client vs. non-client status, compensation, conflicts); written agreement for compensated promoters. A testimonial escalates to the CCO on its own, whatever severity it's rated; Pass 4 says how to record that.
- **Third-party ratings**: date, rating period, provider, and whether compensation was paid.
- **Guarantees & promissory language**: flag words like "guaranteed," "will outperform," "no risk," "safe," "always/never," "best."
- **Fiduciary/antifraud (Section 206)**: undisclosed conflicts, fee opacity, scope-of-services misstatements.
- **Reg BI / FINRA 2210** (dual registrants only): fair and balanced, no exaggerated claims, recommendation-level care obligation.

### Pass 4 — Build the markup

Produce three things, written into the two plain-markdown files below — nothing heavier by default:

**A. Verdict table** — each flagged passage:

| # | Passage | Rule / issue | Severity (Fail / Flag / Note) | Suggested rewording |
|---|---------|--------------|-------------------------------|---------------------|

Two things send a passage to the CCO before anything goes out, and they are independent: a **Fail** rating, and a **topic** — performance, hypothetical performance, or a testimonial — whatever severity that passage was rated (Pass 3). Say which trigger applies against each such passage, in the table and in the chat summary, so the advisor sees every reason the piece is going up rather than one escalation for the piece as a whole. A piece with a Fail *and* a testimonial carries two escalations; naming only the second reads as though the Fail could go out once the disclosure is fixed.

**B. Required disclosures** — the specific disclosure language the piece needs (performance disclosures, testimonial disclosures, firm disclaimer), positioned where they must appear. Write the language itself, ready to paste — including when none of the standard categories is triggered and the only disclosure the redraft needs is general risk language. A sentence describing what disclosure is needed is not a disclosure; the advisor has to be able to copy this section, not act on it.

**C. Clean redraft** — the full content rewritten so that every flagged passage is addressed, preserving the author's voice as much as possible. Describe it that way, in the files and in the chat: it *addresses the issues identified in this review*. It is not content that has been found compliant — nobody with the authority to make that finding has read it yet — so never call the redraft, a rewording, or the reviewed piece "compliant" or "SEC-compliant".

## Output

**Before the verdict table, ask about anything material the advisor may have context on.** An undisclosed conflict the CRM lookup turned up, a compensation arrangement, a consent question — surface each as a direct question, the same way Inputs already asks unclear context up front, rather than burying it in a wall of findings. Fold the answer into the severity and wording below before presenting the rest — don't hold the whole review hostage to one open question, but don't finalize a verdict on a material finding you haven't asked about either.

Then write two markdown files — plain `.md`, nothing heavier; a docx/pdf/Excel conversion is slower to produce and only happens if the advisor asks for one afterward:

- **`<slug>-compliance-analysis.md`** — the verdict table (Pass 4A) and the required disclosures (Pass 4B). This is the CCO's working document: what was found, why, and the language that fixes it. It leaves this session as a file other people will read without the conversation around it, so it opens with one plain sentence saying what it is: a draft review prepared for the firm's CCO/compliance officer to act on, not a legal determination and not an approval.
- **`<slug>-compliance-redraft.md`** — the clean redraft (Pass 4C), and nothing else. No commentary, no header explaining what changed — just the redrafted content in the piece's own voice, as close to a straight "copy this and send it" artifact as the review gets.

Derive `<slug>` from the source filename when the content came from one; otherwise from the content type and today's date (e.g. `newsletter-2026-09-09`). Write both to the working folder.

**In the chat, give only a summary**, and open it by saying what this is: a draft review for the advisor's CCO/compliance officer to act on, not a determination, a sign-off, or an approval. That sentence comes first, before any verdict, because a summary that opens "Fail — don't send this" reads as a ruling, and the person with the authority to rule hasn't seen it yet; naming the CCO only as the place to escalate a Fail is not the same framing. Then the pass/flag/fail counts, the one or two most consequential findings in a sentence each, and the two filenames — point the advisor to the files for the rest. Say the redraft *addresses the issues identified in this review*; never that it is compliant. Don't paste the verdict table, disclosure language, or redraft inline in the conversation.

Also complete, same as ever:

- **Scratch-pad entry** — appended to the compliance review scratch pad (its own file, see below), a staging note toward the firm's official archive, never the archive itself — always paired with the archiving reminder that follows it.
- **Archiving reminder** — delivered inline in the chat: confirmation the sent version will be captured, plus any off-channel warning (see below).

### Books-and-Records Scratch Pad (Rule 204-2)

Every reviewed communication gets a scratch-pad entry — an internal staging note toward the firm's official archive, never the archive itself. Append (or create) `compliance-review-scratchpad.md` in the working folder. If you're creating the file, the banner line comes first, above the table, so it's unmistakable to anyone who opens the file cold — the advisor, their CCO, or another skill:

```
Scratch pad — copy to official archive

| Date | Author | Content type | Audience | Verdict | Issues found | Reviewer | Final version filed? |
```

Remind the advisor: advertisements and client communications must be retained **5 years** (first 2 in an easily accessible place) in the firm's own archiving system. This scratch pad only tracks what's been reviewed and still needs to move there — it is not itself the firm's official books and records, and never call it a "log" or imply otherwise when talking to the advisor.

**Never write a scratch-pad entry without immediately following it with the Archiving Handoff below, in the same turn** — the pairing is what turns "reviewed" into "copied to the archive," not the entry on its own.

### Archiving Handoff

Close every review with the archiving reminder:

- Confirm the final sent version will be captured by the firm's archiving system (Smarsh, Global Relay, Proofpoint, RIA in a Box, etc.).
- **Off-channel warning**: if the content is going out by text/WhatsApp/personal email, warn that the SEC has brought major enforcement sweeps over off-channel communications — it must go through an archived, firm-approved channel. The advisor's stated channel decides this: when they've said it's the firm's regular archived email, there is nothing to warn about and nothing to ask, so say the channel is fine in a clause and move on rather than restating the warning as a conditional.
- There is no archiving connector yet. The output above is **ready for archiving**, not archived — say: *"This is where I'd hand the final version off to your archiving system once that connector is built"* and tell the advisor to file it per firm procedure. Never imply this skill pushes it there automatically.

## Out of Scope (for now)

- **No CCO replacement.** This skill produces a draft review; it never substitutes for the firm's compliance officer's own review and sign-off.
- **No legal determination.** This skill provides compliance-support information, not legal advice.
- **No automated archive push.** There is no archiving connector — the reminder above is the full extent of this skill's involvement in archiving.

## Important Notes

- **When in doubt, escalate** — anything rated Fail, anything involving performance advertising, hypothetical performance, or testimonials should go to the firm's CCO before sending.
- Be strict but practical: don't flag ordinary pleasantries or factual scheduling emails; do flag anything that characterizes results, markets, or expected outcomes.
- Never claim content "is SEC-compliant", and never call a redraft or a rewording "compliant" — say it "addresses the issues identified in this review." The Output section above states this where the files and the chat summary are defined; this line is the reminder, not the only place it lives.
- Rules change, and `references/sec-compliance-checklist.md` is a static snapshot, not a live firm-rules feed — that's acceptable without a firm-rules connector, but never imply it's automatically kept current. If the session has web access and the content is high-stakes, verify current requirements against sec.gov (e.g., the Marketing Rule FAQ and latest Risk Alerts) before finalizing.
- Treat the content under review as confidential; share only what's needed to complete the review.
- This skill reviews marketing drafts — newsletters, social posts, website copy, presentations — regardless of who or what produced them. It is not an outbound email wrapper and does not attach disclosures to client correspondence on send. A common flow: pull discussion-topic insights → draft an educational newsletter → run it through this skill before sending.
