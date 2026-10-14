# 03 · Rewind — prompts

**Context:** You joined Rook two weeks ago as PM on Dispatch.
Release 4.2 shipped on 12 August, just before you arrived, and
landed badly.

Last session you read four conversations and every support ticket
since 4.2 — and found the two piles did not agree.

At the end of the session, type wrap up, and Claude Code saves the
prompts you wrote yourself below — not the starter prompt — updates
CLAUDE.md, and saves your work to GitHub. By Module 6 this file is a
prompt library built from your own questions.

---

### 1.

Use the rook-database connector. Call describe_table on pings, then run_query. Every row is one time we pinged a responder, and whether they took it, turned it down, or missed it. Release 4.2 shipped on 12 August. Tell me what changed after that date. Show me the weekly numbers before and after, and show me the rows you used to get them.

### 2.

If you had to choose between acceptance rate, missed-ping rate, total missed pings, or responder-level distribution changes as the single headline metric for the Director, which best isolates the impact of 4.2 and why? Rank them from strongest to weakest executive summary metric.

### 3.

Calculate the absolute and relative change in missed-ping rate and acceptance rate before vs. after 12 Aug. Which change is larger and easier to explain to an executive audience as the primary impact of 4.2?

### 4.

If acceptance fell because missed pings increased while declined pings remained stable, what is the clearest one-sentence causal narrative supported by the data? What metric best represents the underlying responder experience rather than just the business outcome?

### 5.

Use the context above to answer this question: If the Director of Product asked tomorrow what release 4.2 actually did to responders, what's the one number you'd put in front of her? Post it, and one line on where it came from.

### 6.

Pick one responder from the "quiet responder" group and reconstruct their journey through the data. Show their weekly metrics, identify the inflection point where behavior changed, and explain in plain English what a manager would have observed if they were watching that responder from July through September.

### 7.

Identify responders who went through the same pattern as The Undertow (miss spike followed by a sharp drop in pings) but later recovered. What changed immediately before they started getting pinged regularly again? Show the timeline and metrics before, during, and after recovery.

### 8.

For responders who became "quiet," compare their missed-ping rates in the two weeks before the drop-off with responders whose ping volume remained stable. Is there a threshold where misses are consistently followed by fewer future pings?

### 9.

For each responder who became quiet, examine the first ping they received after a prolonged gap. What happened next? Did they accept it, miss it, or decline it, and did their ping volume increase afterward?
