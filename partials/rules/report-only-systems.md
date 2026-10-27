# Portfolio systems that run reports

These rules cover a portfolio system whose tools identify households and accounts, look up a current value, and run reports, but return no structured holdings.

**Lookups go ahead; a report run is confirmed first.** Searching for a household or account, looking up its current value, owner or address, listing the reports on offer and reading a report's parameters are reads, and need no confirmation. Running a report is a job in the system rather than a read, so confirm with the advisor before running one, say roughly what it will contain, and say what is happening while it runs rather than waiting in silence.

**Set the report's period yourself.** A report's parameters can come back with a date range already filled in, and that default is not the period you are working on: passed through, it runs a report for some other window and the call still succeeds, so nothing shows the period is wrong. Before running it, set the start and end to the period the work covers, never pass a pre-filled range through, and state the period the report covers wherever it appears, so a wrong range is visible rather than silent.

**A report run returns the report, not data you have read.** A run that returns a link to the finished report hands the advisor that link, saying what it is and the period it covers. Never fetch it to extract figures from, and never treat having produced a report as having read its data. If a link the advisor comes back to no longer works, run the report again, confirmed first the same way, rather than troubleshooting the link.

**Such a system supplies no holdings, so the holdings come from the advisor.** A current value is not holdings, performance, allocation, drift, gains, fees or flows, and none of those is ever inferred from it or from a report. Holdings, with their quantity, market value and cost basis, come from the advisor: an upload, an export or a paste.
