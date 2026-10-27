# Contributing to Claude for Financial Advisors

**This repository is not actively maintained or monitored.** It is published as a reference implementation. Pull requests and issues may not receive a response. The guidance below is for anyone forking or adapting the plugin, and applies to any pull request that does get reviewed.

Everything in this repo is Markdown and JSON.

## Before your first PR

Sign the CLA. The first time you open a pull request, the CLA Assistant bot will comment with a link to the [CLA](CLA.md) and ask you to confirm. Reply with `I have read the CLA Document and I hereby sign the CLA` and the check will pass. You only need to do this once.

## What goes where

- **Skills** live in `skills/<skill-name>/SKILL.md`, with the same frontmatter the existing skills use (`name`, `description`). Keep the description short; it is the trigger signal. Supporting material goes in `skills/<skill-name>/templates/` or `skills/<skill-name>/references/`.
- **Agents** live in `agents/<name>.md`. Each one has a single, narrow job and states which connectors it reads from.
- **Connectors** are declared once in `.mcp.json` and described in [README.md](README.md). Add a connector only when a skill actually reads from it.
- **Provider files** hold what is particular to asking one provider's system, the things that change when that system does: how to word a question to it, which conversation a question belongs in, and what to do with an answer only it gives. They live in `providers/<provider>/<job>.md`, one folder per provider, named as its connector is keyed in `.mcp.json` (`providers/cais/ask-caisey.md`). Skills and agents state connector behaviour generically and never describe one provider's, so that description goes here. `source-extract`, which makes the plugin's reads of a connected system, lists `providers/` before its first call and reads the folder for the system it was sent to. A skill whose lead can finish one of those reads itself tells the lead to do the same, in the same generic words, and a skill whose output carries a provider's citations tells its lead to read them too when the advisor asks to see a source that provider cited. Onboarding reads the folder of each system the advisor has just connected, for an example to show a new advisor that needs no client data (`providers/cais/new-advisor-example.md`). So a provider's files are read whenever a skill reads that provider, shows a source it cited, or shows what it offers once connected, and at no other time: unlike `partials/`, nothing here is attached when a skill loads.
- **Partials** in `partials/` are passages several skills share, each attached where a `SKILL.md` names it as `@${CLAUDE_PLUGIN_ROOT}/partials/<path>`.

## Design rules

- **No client data in the repo.** Demo material must be clearly fictional. Never paste real statements, holdings, names, or account numbers, even redacted.
- **Every write to an external system pauses for advisor approval.** A skill may draft a CRM note, task, or client message, but it must stop and ask before anything leaves the session.
- **No trade recommendations or execution.** Skills describe options and surface data; the advisor decides.
- **Client-facing output routes through `/compliance`.** If a skill produces something a client might see, say so in the skill and point at the compliance check.
- **Degrade gracefully.** When a connector is not available, fall back to paste or upload rather than failing.

## Checks

Before pushing, confirm `.claude-plugin/plugin.json`, `.claude-plugin/marketplace.json`, and `.mcp.json` are valid JSON and that any relative paths referenced from a `SKILL.md` exist. The `check-mcp-urls` workflow probes every connector URL on pull requests that touch `.mcp.json`.

## Reporting issues

You can open a GitHub issue, but this repository is not actively monitored and issues may not receive a response. For security concerns, follow [SECURITY.md](SECURITY.md) instead.
