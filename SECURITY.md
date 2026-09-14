# Security Policy

## Reporting a vulnerability

This repository is maintained by [Anthropic](https://www.anthropic.com/). If you find a security issue in this plugin, please report it responsibly and do not open a public GitHub issue.

Anthropic's security program is managed on HackerOne. Report validated vulnerabilities through the [HackerOne submission form](https://hackerone.com/4f1f16ba-10d3-4d09-9ecc-c721aad90f24/embedded_submissions/new). Program guidelines are on the [Anthropic HackerOne page](https://hackerone.com/anthropic).

Include a description of the issue, steps to reproduce, potential impact, and a suggested fix if you have one.

## Scope

This policy covers the contents of this repository: the skill, agent, and connector definitions. The plugin holds no client data and runs no server; data flows through the third-party connectors listed in [README.md](README.md). Vulnerabilities in those third-party services or in Claude itself should be reported to their respective maintainers.

## Response

This repository is a reference implementation and is not actively monitored. Reports through HackerOne reach Anthropic's security team, but fixes to this repository are best effort and not guaranteed.

## Disclosure

We follow coordinated disclosure. After a fix is available we will publish a security advisory on GitHub. We ask that reporters refrain from public disclosure until a fix has been released or 90 days have passed from the initial report, whichever comes first.
