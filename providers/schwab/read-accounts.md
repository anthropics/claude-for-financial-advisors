# Reading Schwab accounts

These rules are for a read of Schwab, a custodian. Such a read pulls a household's accounts, balances, positions, tax lots, gains, transactions, money movement or alerts. These rules say what is particular to Schwab's answers. Everything else about the read follows the read's own instructions. That includes the fields it fills and the status it returns.

**Pass the account keys you were handed.** Schwab returns each account as a masked number, such as `****1234`, together with an `accountKey`. Every tool that takes an account takes the key. The masked number is not a valid input. A key is good for this conversation only. Never return one in a field.

**An account's date opened is its `establishedDate`.** `schwab-as-list-accounts` and `schwab-as-list-profiles` return it. `linkedToMasterDate` is the date the account was linked to the firm's master account, not the date it was opened. `schwab-as-list-account-owners` returns only `linkedToMasterDate`, so read the date opened from `schwab-as-list-accounts`.

**Only read, and leave the alert inbox as you found it.** Every Schwab tool reads. One of them also changes something. `schwab-as-list-alert-details` marks each alert it returns as viewed in the advisor's Schwab alert inbox, in the advisor's name. Never call it to fill a field. Call it only when the advisor asks what an alert says. Tell them first that reading it marks the alert as viewed. `schwab-as-list-alerts` gives each alert's subject, type, priority, status, reply type and date. That is all a read needs. An alert is open when its status is New or its reply type asks for action. The service-request status feed only reads. You may use it.

**Alert and status-feed dates are Eastern Time.** Leave out the end date. It defaults to now. An end date later than the present moment in Eastern Time is refused. Alerts go back at most 18 months. The status feed goes back at most 90 days.

**Cost basis can be closed to the login.** The unrealized gain, tax lot, realized gain and cost-basis preference tools can return `401 Unauthorized` while every other tool answers. That happens when the advisor's login isn't entitled to cost-basis data. Treat it as the tool's error. Quote it. Mark the gain, lot, holding-period and cost-basis fields as not available. Never work out a gain, a cost or a holding period from transactions or positions instead.

**Return each account holder with the role Schwab gives.**

**Schwab reports no account beneficiaries.** Mark every beneficiary field "— not available (Schwab doesn't report account beneficiaries)". A name on a money-movement instruction is the payee of that instruction. It is never an account's beneficiary.

**Leave out IDs, and leave out contact details unless asked.** Never return a position, lot, transaction, alert or feed ID. Never return a taxpayer ID, even masked. Schwab returns email addresses, phone numbers and mailing addresses in full. They appear in account profiles, document preferences and the status feed. Leave them out of every field and document unless the advisor asked for them. When the advisor asks what contact details Schwab has on file for a client, give the ones Schwab returned. The counterparty on a money movement can include a bank account number. Return the counterparty's name, and mask any account number in it.

**Carry values in the form Schwab gives them, and say what that form means.**

- Amounts come back as numbers. The day change comes back as text, such as `+$8.84(0.77%)`. Carry it as given. An amount in parentheses with no sign is a loss or a negative amount. Carry it as given, and mark it as a loss.
- A holding period comes back as a number of days or as a label meaning long-term. A number of days means the lot is short-term. Carry it as given.
- The "percent of account" figure is a fraction from 0 to 1. It is a share of the account's long securities market value, not of the account's total value. `0.2479` means 24.79% of the securities held long. Carry it as given. Put the position's market value and the account's long securities market value beside it. Never convert it.

**A holding can come back in more than one row.** Schwab lists a security once for each way it is held. One row can be of type Cash and another of type Margin. Return them as one holding. Add the rows' quantities, market values and percentages together. Say in `notes` which holdings you combined.

**Balances carry their as-of time.** Read balances with `schwab-as-list-balances`, even for one account. It is the only balance tool that returns an as-of time. That time is Eastern Time. Carry it with every balance and position figure. Where a figure has no as-of time, say so. Never give today's date in its place.

**Report accounts Schwab rejects.** The tools that take several accounts list any account they couldn't read under `invalidAccounts`. Say in `notes` which accounts those were. Name them by masked number. Never fill their fields from another account.

**Follow a list to its end.** Some lists come back one page at a time. Read every page before you fill a field from the list. If a page couldn't be read, say so in `notes`.
