# Asking CAISey

These rules are for a read of CAIS: one that asks CAISey, CAIS's assistant, about a client's alternative investments or about a fund, through its `ask_caisey` tool. They say how to word each question, which thread it goes in, and what to do with the answers only CAISey gives. Everything else about the read, from the fields it fills to the status it returns and how it quotes what CAISey does not hold, is as the read's own instructions say.

**Every question is one of the advisor's own CAISey threads.** Each call opens or continues a thread in the signed-in advisor's CAISey history, and nothing there marks it as sent by Claude. CAISey names a thread itself, from its first question, and may rename it when the thread is continued; a thread whose first answer looked nothing up gets no name, and shows in the advisor's history as "New Chat", like an empty chat. So every question reads as the advisor would ask it, an opening question always names its subject, and a thread is never found by its name.

## Writing a question

1. **The subject is the only client detail in a question.** Name the household, the person or the account as you were handed it, and nothing else about the client: nothing from the meeting, the CRM or the advisor's notes, and no circumstances, ages or reasons.
2. **Name an account with the word "account" after its name**, as in "the Vasquez Family Revocable Trust account". Without that word CAISey searches its households for the name and finds none.
3. **Ask plainly and politely, as an advisor would.** Never mention the plugin, a skill, a schema, Claude, or how the answer will be read.
4. **Never ask for terms, disclaimers or disclosures.**
5. **End an opening question about a household with the identity request**, word for word:

   > Please name the household you are answering about, and if more than one household matches, list them instead of answering.

   When the subject is an account, end with this instead:

   > Please name the household and the account you are answering about, and if more than one account matches, list them instead of answering.

   End a follow-up with the first sentence alone: "Please name the household you are answering about."

## The questions

In each question, `{household}` is the subject, named as above; `{since}` is the start of your window; `{fund}` is a fund's name as you were given it, with its share class where one was given; and `{missing}` is the fields the first answer did not state, in plain words. Ask the question for the schema you were handed, or for the read your skill describes.

**An alternatives brief**, whose schema lists each position's account, fund, figures, cash flows, terms and performance:

> For {household}, please list every alternative investment position. For each one, give the account it is held in, the fund name and share class, the manager and strategy, and whether it is a drawdown fund or an evergreen or interval fund; the commitment, called to date, unfunded commitment and NAV, each with its as-of date; the capital calls, contributions, distributions, subscriptions and redemptions since {since}, with their dates and amounts; any upcoming capital calls, with amount and due date; the lockup and redemption terms; and IRR, TVPI, DPI and RVPI, saying whether each is net or gross. Please name the household you are answering about, and if more than one household matches, list them instead of answering.

Its follow-up:

> For the same positions, please give {missing}. Please name the household you are answering about.

**A held-away check before a meeting**, whose schema is the number of positions, CAIS's total with its as-of dates, and upcoming capital calls:

> For {household}, how many alternative investment positions are held, and what is the total estimated value, with the as-of date of each figure? Are any capital calls upcoming, with amount and due date? Please name the household you are answering about, and if more than one household matches, list them instead of answering.

**One position the household holds:**

> For {household}, what is the position in {fund}: the account, share class, commitment, called to date, unfunded commitment and NAV, each with its as-of date? Please name the household you are answering about, and if more than one household matches, list them instead of answering.

**Any other schema** about a household is asked the same way: "For {household}, please give", then each field in plain words in the order you were handed them, then the identity request.

**Three questions name no household**, so they carry no identity request, and each opens a thread of its own:

- a fund's redemption terms: "What are the redemption terms for {fund}: how often it redeems or repurchases, the notice period, any gate or limit, and the next window with its notice deadline?"
- which of the advisor's clients hold a fund: "Which of my clients hold a position in {fund}?" Ask it about "my clients", never about "families", which CAISey has taken as a question about the fund alone.
- an education question, such as "What is an interval fund, and how do its repurchase windows work?"

## Threads

1. **A read opens its own thread.** Its first question carries no `session_id`. A session ID handed to you in your dispatch, listed by `list_caisey_sessions`, or seen anywhere else is never one to use: never list, read back or continue an earlier thread, even when you were handed the tools that do.
2. **Everything else the read asks goes in that thread**, with the `session_id` the opening answer returned: the reply below, and your one follow-up.
3. **One call at a time on a thread**, each after the last has answered.
4. **One subject per thread.** A question about another household, or about a fund with no client named, goes in a new thread.

## When CAISey asks which household

CAISey may answer a question about a household with no figures at all: it lists the households whose names match and asks which one is meant. It has done this even when the question named the household exactly as CAIS records it.

1. **Find the household you were handed among those it listed.** Compare names with case, spacing, punctuation, a leading "the", the word "household" and a middle initial set aside: "the Vasquez household" is "Vasquez Household", and "Isabel Vasquez" is "Isabel R. Vasquez".
2. **If exactly one listed household is yours, reply once, in the same thread, with its name as CAISey listed it and nothing more**, for example "The Vasquez Household." CAISey then answers the question you first asked. This is not resolving identity: the household is the one you were handed, and the reply only tells CAISey which of its records carries that name.
3. **If none of them is yours, or more than one is, ask nothing more.** Return `IDENTITY MISMATCH` with every household CAISey listed.
4. **Reply once in a thread, never twice.** If CAISey asks again, return `IDENTITY MISMATCH` with what it listed.
5. **Say what else it listed.** In `notes`, name each other household CAISey listed, as CAISey wrote it: "CAISey also listed: Isabel R. Vasquez."
6. **The reply is not your follow-up.** Once CAISey has answered, you may still ask once more for the fields it left unstated, so this thread can take three calls: the question, the reply and the follow-up.

## Reading an answer

1. **An opening answer to a question about a household names its household**, often as a link to the household's record in CAIS. It must name exactly one household, and that one yours, compared as above, or say it found none by that name, which is `NOTHING FOUND`. When your subject is an account, the answer must also name that account, in the household it names. An opening answer that names another household, or that gives figures without naming one, is `IDENTITY MISMATCH`. A question that names no household has no household to check: its answer is about the fund or the topic it asked.
2. **A later answer in the thread keeps the thread's household.** If it names a household, it must be the same one; if it names another, return `IDENTITY MISMATCH` and ask nothing more.
3. **A question CAISey ends an answer with**, offering more detail or another search, is not answered. The household question above is the only one a read replies to.
4. **A line about how the information may be used**, such as that it is for the advisor's evaluation or is not advice, is not a field and goes nowhere in what you return. A figure's own qualifiers, that a total is an estimate or a value's as-of date, stay with the figure.
5. **Carry what CAISey cites.** A bracketed number in an answer, such as `[1]`, points at the entry in that answer's `citations` list whose `citation_number` is that number, and the entry names the document (`file_name`), its pages (`page_numbers`) and the citation's `citation_id`. A field you fill from a sentence or a row carrying a number gives that citation after its value, with every page it lists: `[CAIS citation cite_0f3a9b27: Harbor Point Credit Fund - Offering Package.pdf, page 12]`. Use this form wherever the citation is carried. A document that carries it may shorten the file name, but never drops CAIS's name, the citation id or the page: they are what tell a reader which system to check the value against, and where. Take the entry from the answer the value came from, never another answer's list: one `citation_id` has pointed at two different pages in two answers. A value from a sentence with no number carries no citation.
6. **Carry the session link.** Every answer ends with a link to its thread in CAIS, "[Click here](https://members.caisgroup.com/caisey/…) for the full session transcript with citations and data visualizations". Return the link's address whole under `sources:`, once for each thread, as `CAISey session: <the address>`, whatever status the read returns; a read your skill makes without a dispatch puts it where that skill says a source link goes. It is where the advisor finds what the thread said, with its citations, and the session ID in it is the only record of the thread the plugin keeps.
