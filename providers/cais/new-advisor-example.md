# Showing a new advisor what CAISey can answer

These rules are for onboarding, once CAIS is connected, when it shows a new advisor an example of what CAISey, CAIS's assistant, can answer. A read of CAIS never does this.

The example is an education question. CAISey answers it from CAIS IQ, CAIS's education content, and it names no client, so it needs no client data.

1. **Ask this question through `ask_caisey`, word for word, with no `session_id`**, so it opens a thread of its own:

   > What is an interval fund, and how do its repurchase windows work?

   It names no household, so it carries no identity request (`ask-caisey.md`, "The questions"). Ask it once and nothing after it: the follow-up topics CAISey offers are not asked.
2. **Say that you asked CAISey, and that the question is now one of the advisor's own threads in their CAISey history**, so a thread they don't remember opening is no surprise there.
3. **Show the answer in CAISey's words**, without its list of follow-up topics or any line about how the information may be used.
4. **Carry what it cites and its session link**, as `ask-caisey.md`, "Reading an answer", says. Keep the answer's bracketed numbers, and under it give each number's citation once, from this answer's own `citations` list, in that file's form: `[1] [CAIS citation <citation_id>: <file_name>, page <pages>]`. Then give the session link, its address whole, as where the advisor can read the thread in CAIS with its sources.
5. **If CAISey says its material doesn't cover the question, there is nothing to show.** It has answered that way where CAIS held no CAIS IQ content.
