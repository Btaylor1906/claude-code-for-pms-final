# Product brief: Quiet Responders — "Still in the system?"

**Product:** Rook Dispatch  **Author:** PM, Dispatch  **For:** Helen Achebe (Director of Product); then Marcus Oyelaran, Wen Li, Sofia Marino, Nadia Hoffmann, Ravi Menon
**Status:** Draft for discussion, 16 Oct 2026. Nothing here is committed. It proposes what we build; it does not authorise a code change.
**Primary persona:** The Undertow (responder, Harborside), handler Desmond Okafor

> **Read this first: data caveat.** Every number below comes from rook-database (pings and tickets, 29 Jun to 7 Sep 2026) and the replay of the routing code. The data ends five weeks ago. We have not seen real ranking scores, and we have no response-time or push-delivery data. Ravi's weekly numbers and Wen's answers (see Open questions) should be confirmed before anything is scoped as final.

---

## 1. Executive summary

Since 4.2 (12 Aug), some responders have stopped being offered callouts, and nobody, including them, their handlers and Support, can see why. The Undertow went from about 12 pings a week for six straight weeks to **1 ping in the week of 31 Aug**. His handler, Desmond Okafor, filed **11 tickets** about it between 20 Aug and 7 Sep. All are still open, and Support has no answer to give him.

The cause is a design choice in the routing code that nobody has revisited. `history.py` treats a **missed** ping (the ping wait ran out) exactly like a **turned down** one, costs 0.12 each time, gives back only 0.08 for a taken ping, and has **no decay**. 4.2 cut the ping wait from 90s to 60s, so misses jumped from 2.3% to 18% of pings. Four responders (The Undertow, Farlight, Meteor Mite, Vesper) slid to a score of 0.00. Because they rank last, they are rarely offered anything, so they have almost no chance to earn the score back. Wen left a 2019 TODO next to that code asking whether this should ease back on its own. It was never answered.

**Proposal (revised after a devil's-advocate review, Appendix A):** read the real scores first, then build in order of evidence strength.
1. **Handler console: make quiet visible.** Best supported (11 open tickets, Kip's interview). A factual line on the coverage card: pings this week vs usual, last pinged, missed vs turned down. No routing change.
2. **Routing, gated by replay:** a smaller penalty for missed pings and a score that eases back toward neutral are plausible but unproven. Each is built only if replaying the pings table and reading Wen's real scores supports it. Guaranteed re-entry is **not** in the first release: all 13 pings the four received after a gap were missed or turned down.
3. **Responder phone status** is conditional on talking to a quiet responder first.

It does not revert 4.2, change the 60s ping wait, or remove proximity weighting. Those decisions stay separate (section 9).

---

## 2. Who is this for? (Question 1)

### Primary users
| User | Why they are affected |
|---|---|
| **Responders who miss a few pings and then go quiet** (today: The Undertow, Farlight, Meteor Mite, Vesper; Halfmoon and Corporal Ashgrove dipped about 20–25%) | They are independent and paid per callout they take. A drop in pings is a drop in work, and they cannot see why. |
| **Their handlers** (Desmond Okafor for The Undertow; Kip for Meteor Mite; Aunt Dot for Vesper) | They watch the console, answer the responder's questions and file the tickets. They currently have nothing on screen to explain a quiet week. |
| **Support (Nadia Hoffmann)** | Eleven open tickets for one responder alone, with no answer to give. |

Secondary: the other 12 responders, who now get slightly more pings and a higher miss rate (about 2% before, about 10–14% after). This brief should not make their experience worse.

### How it affects them today, through The Undertow
The Undertow was an ordinary, reliable responder. Before 12 Aug he was pinged about 12 times a week, took roughly 77% of them, turned down about 21% (a normal decliner) and **missed about 1%**. After 4.2:

| Date (2026) | What happened to The Undertow |
|---|---|
| 12–14 Aug | 7 pings, **5 missed**, 2 taken. His score drops fast: with the 60s wait, he missed pings he would have answered before. |
| 17 Aug | His **last taken ping** (as of 7 Sep). |
| 18 Aug | Missed. |
| 20 Aug | Desmond files the first ticket: "six pings in the past seven days… availability is set correctly and his tags are all there." |
| 21 Aug → 27 Aug → 5 Sep | Pings arrive after gaps of 3, 6 and 9 days. **Every one is missed.** |
| 1 Sep | Desmond passes on The Undertow's own words: *"nothing again this week. starting to wonder if im still even in the system."* |
| 5 Sep | The ticket "First one in 9 days and he missed it": it landed and was gone before he could answer. |
| 7 Sep | One ping in the past seven days. All 11 tickets are open. |

Desmond's repeated line is "Nothing looks wrong on our side. Could someone check yours?" He is right, and nobody on our side can show him anything. Replaying the pings through the routing rules puts The Undertow at 0.84–1.00 on 12 Aug (as high as anyone) and **0.00 by the end**.

**The same pattern, in the handlers' own words:**
- **Kip:** *"Meteor Mite's been texting me all week asking if something's broken because it's so quiet… The Gale hasn't stopped… Mite thinks Mite's been forgotten."* He wants it to "look less like a coincidence."
- **Aunt Dot:** she cooks for "someone who's home all week instead of half of it", and had never put the quiet weeks next to the fast-vanishing ones. She notices the pattern but cannot explain it.
- **Mr. Ambrose** and **Halloran** (for Captain Vantage and Sgt. Bulwark) describe the other 4.2 symptom, pings that are "gone" before the responder can answer. That is the 60s wait, which this brief does not change.

---

## 3. Problem statement

When a responder who has been reliable gets a short run of **missed** pings, Dispatch's routing logic ranks them last. Because ping chains are short (average 1.4 pings per callout before 4.2, 1.66 after, longest 4), someone ranked third or lower is rarely reached. So they are offered fewer pings, which gives them almost no way to raise their score, and the score has no decay and no reset. The responder and handler see only silence. Support sees tickets. Nobody sees the reason.

**This is not what the 4.2 notes describe.** 4.2 was framed as "proximity weighted up", and Marcus asked on 14 Aug whether it was meant to apply to responders who turn pings down. The data says that was not the group that was hurt: The Undertow was a normal decliner before 12 Aug. **The group hurt was responders who missed**, and the mechanism is the miss penalty plus the shorter wait, not the weights alone.

---

## 4. Supporting evidence and findings

| Source | Finding | Confidence |
|---|---|---|
| **pings** | Pooled missed pings 2.3% → 18.0% before/after 12 Aug (about 7.8×). Declines roughly flat (21.1% → 18.0%). | High |
| **pings** | The four quiet responders went from about 1.7 to about 0.5 pings a day; about 59% of their remaining pings were missed (about 3% before). Weekly volumes for the four: 43, 16, 6, 3. | High |
| **pings** | None of the four had recovered by 6 Sep. The overall "recovery" in the average comes from the other 12 responders and from the four dropping out of the totals. | High |
| **pings** | All 13 pings the four received after a gap of 2+ days were missed or turned down, and the gaps kept lengthening. | High |
| **pings** | Descriptive threshold: 4+ misses in 7 days was followed by under half normal volume about 81% of the time (3 or fewer, not). Declines do not predict it. There was no warning sign before 12 Aug. | Medium (4 responders) |
| **tickets** | Tickets about quiet pings lag the pings by 1–2 days. "Gone before I could answer" fades as misses fall. "Quiet" builds and peaks about a week later. About 2:1 quiet to gone. | High |
| **tickets** | Meteor Mite and Vesper filed no quiet tickets but show the same collapse, so ticket counts under-count the problem. | High |
| **routing code** | `history.py`: a miss and a decline both cost 0.12; a taken ping gives +0.08; no decay; a 2019 TODO asks about decay and says "Leaving it as-is for now." | High |
| **routing code** | `config.py` 4.2: proximity 0.45 → 0.60, recent acceptance 0.40 → 0.25, ping wait 90 → 60s. `routing.py`: nobody is removed from the list, only reordered. | High |
| **code replay** | The four were at 0.84–1.00 on 12 Aug and ended at 0.00; the others recovered to 0.76–1.00. | Medium: assumes the table is the full history; real scores unseen |
| **interviews** (4 handlers, 2–5 Sep) | Kip, Aunt Dot: quiet stretches with no explanation. Ambrose, Halloran: pings gone before the responder can respond. | Medium: no responder was interviewed |
| **wiki** | Priya's "seasonal dip" claim is unsupported: tickets jumped the day after the release, with no prior-year data cited. Callouts did fall about 27% in week 1, so seasonality is not fully excluded. | Medium |

**Gaps in the evidence:** no interview with The Undertow or any quiet responder (his only words are second-hand, via Desmond's tickets); no ranking scores, response times or push-delivery data; nothing after 7 Sep.

---

## 5. Proposed solution (revised: every feature tied to evidence)

**Revision note.** The first draft proposed three routing changes with invented numbers (a 0.04–0.06 miss penalty, a 50% quiet threshold, "top two" re-entry) and a handler reason line that asserted a cause we have not seen. A devil's-advocate review (Appendix A) found that several of those rested on assumptions, and one piece of evidence in this brief cuts against re-entry. This section now says, for each feature, which evidence supports it, what is **not** proven, and the gate it must pass before anyone builds it.

### Evidence index (all from sections 2–4 of this brief)
| ID | Evidence | Confidence |
|---|---|---|
| E1 | Pooled missed pings 2.3% → 18.0% after 12 Aug; declines flat (21.1% → 18.0%) | High |
| E2 | Four responders' weekly pings 43, 16, 6, 3; about 59% of remaining pings missed (about 3% before) | High |
| E3 | None of the four had recovered by 6 Sep | High |
| E4 | All 13 pings the four got after a 2+ day gap were missed or turned down; gaps kept lengthening | High |
| E5 | `history.py`: a miss and a decline both cost 0.12; a taken ping gives +0.08; no decay; a 2019 TODO left open | High |
| E6 | Replay: the four went from 0.84–1.00 to 0.00; the others recovered to 0.76–1.00 | Medium (assumes the pings table is the full history; real scores unseen) |
| E7 | The other 12 responders' miss rate rose from about 2% to 10–14% and they recovered under the same rules | High |
| E8 | Ping chains are short (1.4 before, 1.66 after, longest 4), so rank 3 or lower is rarely reached | High |
| E9 | The Undertow was always 1st or 2nd when pinged, so "ranked low" vs "rarely available or in-region" is open | Medium |
| E10 | 4.2 weights: proximity 0.60, recent acceptance 0.25, capability 0.15 | High |
| E11 | Desmond Okafor's 11 open tickets about The Undertow; "Nothing looks wrong on our side. Could someone check yours?"; "starting to wonder if im still even in the system" | High |
| E12 | Kip: two cards "might as well be two different products"; Meteor Mite thinks they've been "forgotten" | Medium (one handler) |
| E13 | Declines do not predict a collapse; 4+ misses in 7 days did (4 responders only) | Medium |
| E14 | A 14 Jul ticket: a ping missed while The Undertow was out of signal | Low (one ticket, before 4.2) |
| E15 | Meteor Mite and Vesper filed no quiet tickets but show the same collapse | High |

### 5.0 Build order: read first, build second
Nothing in 5.1–5.3 gets built until these read-only checks are done. None needs a code change.
1. **Wen Li or Marcus Oyelaran read the live recent-acceptance score** for the four responders (answers E6, E9).
2. **Confirm whether the score is stored or lives only in memory** (history.py's `_scores` dictionary) and whether it survives a restart.
3. **Check The Undertow's availability, region and travel time** during the quiet weeks, so we know whether he is ranked low or never a candidate (E9).
4. **Ravi Menon re-pulls the weekly numbers from 7 Sep** to today, split by turned down vs missed. Did the four recover?
5. **Replay** the pings table through each candidate rule (below) and compare against the real scores from step 1.

The order after that: visibility first (5.4, supported by E11 and E12, and it does not touch routing), then one routing change chosen by the replay.

### 5.1 Missed pings cost less than turned-down ones
- **Problem it solves:** a missed ping is treated as a decline, so the 60s wait sends reliable responders down the order.
- **Evidence for:** E1 (misses 7.8×, declines flat), E2, E5 (miss = decline = 0.12), E6, E13 (declines don't predict collapse, misses do).
- **Not proven:**
  - Real scores are unseen (E6). If the four are not actually near 0.00, this fix does nothing.
  - E7 shows the other 12 had a 10–14% miss rate under the same rule and recovered. The penalty alone does not separate the four from the rest; something else differs (E9, volume, region).
  - We have no response-time data, so we cannot say whether the misses are slow answers to a 60s clock or phones that were never reachable (E14).
  - The size of the new penalty has **no evidence**. The earlier 0.04–0.06 was a guess and is withdrawn.
  - The code's own comment ("we asked and we didn't get a yes") may be deliberate: a responder who often does not answer costs the callout time.
- **Gate (smallest experiment, no build):** replay the pings table with the miss penalty at 0.12 (current), 0.08, 0.04 and 0. Report, for each: do the four reach rank 1–2 again, how long does it take, and does the order change for the other 12? If the four still sit near the bottom at 0.04, the penalty is not the main cause and we stop.
- **Build only if:** the replay shows the four recover with a penalty smaller than the decline penalty **and** Wen confirms there is no reason a miss should be treated as a decline.
- **Guardrails to watch:** missed-ping rate, time-to-accept, coverage gaps, average chain length.

### 5.2 Recent-acceptance score eases back toward neutral over time (decay)
- **Problem it solves:** with no decay, a low score is permanent unless the responder takes a ping, and a low-ranked responder is rarely offered one.
- **Evidence for:** E3 (no recovery), E4 (gaps lengthen), E5 (no decay; a TODO from 2019 asks for one), E6 (ends at 0.00).
- **Not proven:**
  - Whether the score persists (5.0 step 2). If it resets on restart, the "stuck" story is weaker.
  - Whether the score is really the cause. E10: acceptance counts for 0.25 of the score. Moving a score from 0.00 to neutral 0.50 adds at most 0.125, while proximity has 0.60. For a responder far from the callout, decay may not change the order.
  - The rate has **no evidence**. Too slow does nothing; too fast erases the signal for responders who really do decline.
- **Unintended consequence:** decay helps everyone, including chronic decliners (declines were 18–21% of pings in aggregate) and anyone who has stopped taking work, which is the case the 2019 TODO warned about.
- **Gate (smallest experiment, no build):** replay with decay at three rates (for example 0.01, 0.02 and 0.05 a day, stopping at 0.50) and report days-to-recover for the four. Then check for each responder whether a +0.125 score change would alter the rank at all, using the real travel times from 5.0 step 3.
- **Build only if:** the replay shows recovery in a time we can accept (the proposed success metric is 14 days), the score is shown to move rank for the four, and it does not lift responders with declines above those who accept.

### 5.3 Guaranteed re-entry for a quiet responder: **conditional, not in the first release**
- **Problem it solves:** E8. With chains this short, a low-ranked responder is rarely reached, so they cannot rebuild their score.
- **Evidence for:** E2 (volumes 43, 16, 6, 3), E8, E3.
- **Evidence against:** **E4: all 13 pings the four got after a 2+ day gap were missed or turned down.** The best evidence we have says a re-entry ping to a quiet responder does not get taken. A guaranteed top-two slot would deliver more of them, at the cost of callout time.
- **Not proven:**
  - Whether the four are available, tagged and in range (E9; the availability code is a stub).
  - The 50% threshold and "top two" are invented. E13 comes from four responders, and Halfmoon and Corporal Ashgrove dipped 20–25% without collapsing, so we cannot tell how a threshold would treat them.
  - Why post-gap pings fail: a slow score, 60s, a locked phone or a bad signal? All four causes are open.
- **Unintended consequences:** each failed offer costs the callout up to a full ping wait; a bump beats a closer responder (against the wide-area request behind 4.2, E10); a second rule on top of the score makes "who gets pinged" harder to explain (no written description exists today).
- **Gate (smallest experiment):** if the override tool allows it (the routing override audit log exists since 4.0; confirm with Marcus), for **one** responder (The Undertow) for **one** week, handlers or Dispatch manually offer him nearby, matching callouts first. Measure taken vs missed. If most are missed again, the problem is in the response, not the ranking, and 5.3 is dropped.
- **Build only if:** the experiment shows re-entry pings are taken at a rate close to the other responders' (not just the 14 Jul no-signal type of miss).

### 5.4 Handler console: show that a responder is quiet, with facts only
- **Problem it solves:** Desmond and Kip cannot tell a quiet responder from a broken one, and Support has no answer (E11, E12, E15).
- **Evidence for:** E11 (11 open tickets, none answered), E12, E15 (two collapsed responders filed no tickets, so some handlers do not notice at all).
- **What it shows:** only things we can read from the pings table today: **pings in the last 7 days against the responder's own usual**, **last pinged**, and the **share missed vs turned down** this week. Example: *"1 ping in the last 7 days (usual: 12). Last 4 pings: all missed."*
- **Changed from the first draft:** the draft added "Ranked lower after recent missed pings." That states a cause we have not verified (E6, E9) and is **removed** until Wen confirms the ranking is actually the reason. A "Quiet" badge is allowed only if the rule is simple and shown in the line above.
- **Not proven:** that handlers act on it or find it useful (Kip says he does not know what he would want it to tell him); that the 7-day count is the right window; whether this helps responders who file no tickets (E15).
- **Gate:** Sofia Marino tests a mock with Desmond Okafor and Kip: can they say, in their own words, what is going on? Nadia Hoffmann tries the same text on the open tickets.
- **Unintended consequences:** the badge may read as a performance label; the line may prompt handlers to ask Support for a fix we cannot offer yet.

### 5.5 Responder phone: availability status: **conditional**
- **Problem it solves:** a responder who hears nothing wonders if their account is still active.
- **Evidence for:** E11 (The Undertow, via Desmond: "starting to wonder if im still even in the system"; "asked me this morning whether his account is still active").
- **Not proven:** there is **no responder interview** in our evidence. Everything we know about The Undertow's feelings is second-hand through his handler. We do not know if responders would read or want this.
- **Constraint:** the line must come from the Responder Availability Record. If a responder is not available, it must not say "You're in the offer list". It never shows a score or rank.
- **Gate:** interview The Undertow or another quiet responder (Sofia) before design starts.

### 5.6 What was dropped from the first draft
| Dropped | Why |
|---|---|
| Miss penalty "0.04–0.06" | No evidence for any number; replay decides |
| "50% of baseline" quiet threshold | One group of four; E13 is descriptive only |
| "Top two" re-entry as a first-release feature | E4 is evidence against it |
| "Persist the score" as a feature | It is a question for Wen, not a feature (5.0 step 2) |
| "Ranked lower after recent missed pings" on the card | An unverified cause |
| Phone status as committed | No responder voice in the evidence |

## 6. What changes once it ships? (Question 2)

*The journey below describes the full set of features if every gate in section 5 passes. Rows that depend on 5.1-5.3 may not ship; the visibility rows (5.4) are the first release.*

### User journey: before and after (The Undertow)
| Moment | Before (today) | After |
|---|---|---|
| **A short run of misses** (12–14 Aug) | Five misses in three days; score falls steeply. | Misses cost less than declines. He falls, but not to the floor. |
| **The week after** | His score is at or near 0.00, he ranks last, and offers dry up (4, then 1, then 1 pings a week). | His pings fall below 50% of his baseline, so the "Quiet" rule kicks in and he is placed near the top for matching callouts nearby. |
| **Desmond looks at the console** | Availability and tags are fine. "Nothing looks wrong on our side." | **Quiet** badge with the reason and last-pinged time. He knows it is ranking, not a broken account. |
| **The Undertow looks at his phone** | Nothing. *"starting to wonder if im still even in the system."* | "You're available. You're in the offer list." |
| **A ping arrives after a long gap** | Gone before he could answer (5 Sep, first in 9 days). | He is offered pings at a regular rate again, and a taken ping now earns +0.08 as before. |
| **Weeks later** | No route back: score stuck at 0.00, no decay. | Score drifts back toward neutral on its own; a few taken pings return him to normal. |
| **Support** | Open tickets with no answer. | A clear explanation to paste. |

### Measurable success criteria
All targets are **proposals** for Ravi Menon and Wen Li to confirm against fresh data.

**Primary**
| Metric | Baseline (to 6/7 Sep) | Target |
|---|---|---|
| Responders **available and tagged** with 2 consecutive weeks under 50% of own baseline | 4 of 16 (25%) | **0 sustained for more than 14 days**; any that dip are back above 50% within 14 days |
| Weekly pings for The Undertow-like responders vs own 6-week baseline | The Undertow: 12/week → 1 | Back to **at least 75% of baseline** within 3 weeks of launch (adjusted for total callout volume) |
| **Time-to-first-ping** after a responder falls below 50% of baseline | Not recovered after 3+ weeks | **Within 72 hours** |
| **Missed-ping rate, by responder** | Four responders at about 59% | No responder over 25% in a rolling 14 days |

**Supporting**
- "Phone never goes off" tickets per week: from a peak of about 4× baseline to baseline.
- Open quiet tickets resolved with an explanation: from 11 open for The Undertow to 0.
- Handler comprehension (Sofia, 5 handlers): can they say why a responder is quiet from the card? Target: 4 of 5.

**Guardrails (must not get worse)**
- **Acceptance rate** and **time-to-accept** (median seconds, ping to taken).
- **Coverage gaps** (no available responder with the tags). Not the same as low acceptance.
- Average ping chain length (1.66 after 4.2).
- Median travel time of the responder who takes a callout. Wide-area responders asked for 4.2; we are not undoing that.
- The other 12 responders' ping volume and decline rate.

---

## 7. What does it not do? (Question 3)

### Non-goals
- **Does not revert 4.2** or lower the proximity weight (0.60). Proximity was long requested by wide-area responders. We keep it.
- **Does not change the ping wait (60s).** The "gone before I could answer" theme (Ambrose, Halloran, Dot, and about a third of tickets) stays unresolved by this brief. The 60s wait is the cause of the misses that started this. Whether to set it back, vary it per responder, or add a warning is a separate decision (section 9).
- **Does not change who is eligible.** Ranking still orders; it never removes anyone. A responder with no matching tags, or not available, is not offered.
- **Does not guarantee a minimum number of callouts** or equal work. It prevents a permanent slide, not variation.
- **Does not penalise or reward declining.** Declines keep their current cost.
- **Does not identify anyone.** Nothing in here links a cover identity to a legal one (Security Policy 4.1). We never show a score or rank to responders.
- **Does not build a general coaching, feedback or "nudge" system** for responders.

### Still unsolved after this release
- **Pings gone before a responder can answer** (60s wait). A handler cannot tell a responder how long is left. Ambrose wants the live callout "harder to miss." A handler alert for incoming pings (Aunt Dot's ask) is also not in scope.
- **Push delivery.** We cannot rule out that some "phone never goes off" cases are delivery failures, and The Undertow's 14 Jul ticket mentions a no-signal spot. We have no delivery data.
- **The weights (0.60/0.25/0.15)**, which in the replay hurt responders far from callouts. Not revisited here.
- **Mutual aid, shared cover, Availability Confidence** (committed for 4.2, never shipped, still unassigned) and the Q4 roadmap.
- **Other console asks** from the interviews: dark mode (Kip), text size (Dot, Ambrose), per-responder alert sounds (Kip), a filter-reset warning (Ambrose; Desmond ticket on 4 Sep). Those belong in the console redesign.
- **No documentation** of how pinging is decided (Priya's open request; this brief is not it).

---

## 8. Risks and assumptions

**Assumptions**
1. The pings table is the complete history, so the replay scores are close to the real ones. (Unverified.)
2. The four quiet responders are available, in-region and tagged, and are low because of the score, not for another reason. We cannot see availability, region or travel time (stubs).
3. A guaranteed re-entry does not cost callouts. The first pings to a stuck responder might simply be missed again, since all 13 post-gap pings were missed or turned down, and each missed ping costs the callout time.
4. Handlers will read a "Quiet" badge as information, not as a warning about the responder's performance.

**Risks**
| Risk | Mitigation |
|---|---|
| **Re-entry pings are missed again**, delaying callouts while the responder warms up. | Limit re-entry to callouts within the proximity horizon with a matching tag; cap per week; shadow-test by replay first; watch coverage gaps and time-to-accept. |
| **Proximity gains lost:** the bump puts a far-away responder ahead of a closer one. | Only within the horizon, only top-two, only when quiet; track median travel time. |
| **A "quiet" label reads like a penalty** to handlers and responders. | Wording and test with Sofia; no scores or ranks shown. |
| **Fixing the miss penalty hides the real 60s problem.** | Keep the ping-wait decision on the table; report missed rate by responder alongside. |
| **Score persistence unknown**, so decay or re-entry resets on restart. | Wen to confirm before build. |
| **The data is old** (5 weeks stale). | Re-run the numbers with Ravi before scoping. |
| **Wen is the only source** and has not answered Marcus's 14 Aug question. | Book time with Wen this week; Marcus to join. |
| **A one-afternoon number change** ships without a design. Engineering says the fix is small. | This brief's point is the experience, not just the constant: even if 5.1 ships first, 5.2 and 5.3 are what Desmond and The Undertow would notice. |

---

## 9. Open questions

| # | Question | Who |
|---|---|---|
| 1 | Was the 4.2 weighting meant to apply to responders with a decline history? (Marcus asked 14 Aug; mechanically it applies to everyone.) | Wen Li |
| 2 | Should a missed ping count the same as a decline? Why does the code treat them the same ("we asked and we didn't get a yes")? What penalty is right for a miss? | Wen Li, Marcus |
| 3 | Is the score persisted across restarts? Can it recover without pings, and should it decay (the 2019 TODO)? | Wen Li |
| 4 | What are the real current ranking scores for the four? Is The Undertow ranked low or just rarely available or in-region (he was always 1st or 2nd when pinged)? | Wen Li, Marcus |
| 5 | What do the weekly acceptance numbers show, split by turned down vs missed, for 7 Sep to today? Did the four ever recover? | Ravi Menon |
| 6 | Does the routing override audit log show manual overrides for the four since 4.2? | Marcus |
| 7 | Push delivery: is any of this delivery failure rather than ranking? | Marcus, Wen |
| 8 | Do we revisit the 60s ping wait (or make it vary) as a separate piece of work? This is a commitment under "Ping timeout tuning" on the Q3 roadmap. | Helen Achebe |
| 9 | Which squeezed-out Q3 items (Availability Confidence, the rest of Ping timeout tuning) are still commitments, and who owns them now that Priya has left? | Helen Achebe |
| 10 | What is the right "quiet" threshold and re-entry size? (4+ misses in 7 days came from only 4 responders.) | Ravi Menon, Wen Li |
| 11 | Can we interview The Undertow, Farlight or a quiet responder directly? Today we only have handlers. Also Desmond Okafor, who is not among the four interviewed handlers. | Sofia Marino |
| 12 | What is the state of 4.3 and the Q4 roadmap? The wiki is quiet after 28 Aug. | Helen Achebe |
| 13 | Can Support use the new reason text on the 11 open tickets before the build ships? | Nadia Hoffmann |

---

## 10. Suggested next steps
1. **This week:** Wen and Marcus walk through the code and answer Q1–Q4, Q7. Ravi re-pulls weekly numbers (Q5, Q10).
2. **Replay test:** run the pings table through the proposed rules (smaller miss penalty, decay, re-entry) before anyone picks a number.
3. **Sofia:** clickable prototype of the Quiet badge and the phone status line; short test with Desmond and Kip.
4. **Helen:** decide the ping-wait question (Q8) and the Q3 leftovers (Q9) in the same conversation.
5. **Interim, no code:** Nadia replies to the open tickets with what we know (the pattern is real, a fix is being designed).

---

## Appendix A. Devil's-advocate review of the routing changes

*Assumption for this review: the proposed solution is wrong. It uses only evidence already in this brief (E-numbers refer to the index in section 5).*

### A1. Reduced miss penalty
| | |
|---|---|
| **Tries to solve** | Misses cost the same as declines, so the 60s wait pushes reliable responders down. |
| **Supporting evidence** | E1, E2, E5, E6, E13. |
| **Missing evidence** | Real scores (E6 is a replay). Why the other 12 recovered under the same rule (E7). Any basis for the size of the penalty. Response times, so we cannot tell a slow tap from an unreachable phone (E14). |
| **The case against** | The rule may be doing its job. "We asked and we didn't get a yes" means a callout waited. E7 says 12 responders took the same rule and recovered, so the rule alone does not explain four collapses. E9 says The Undertow was always 1st or 2nd when pinged, which does not look like a responder pinned to the floor. We may be treating a symptom (the 60s wait creates misses) by weakening the signal. |
| **Unintended consequences** | Unreachable responders stay near the top, so more callouts wait through missed pings. A lower penalty hides the 60s problem: acceptance looks better and the cause is still there. |
| **Smallest experiment** | Replay the pings table at four penalty levels (0.12, 0.08, 0.04, 0) and check whether the four recover. Separately, ask Wen for their real scores. No code change. |

### A2. Score decay
| | |
|---|---|
| **Tries to solve** | A low score never recovers without a taken ping (the 2019 TODO). |
| **Supporting evidence** | E3, E4, E5, E6. |
| **Missing evidence** | Whether the score persists. The right rate. Whether score is what limits their volume at all. |
| **The case against** | E10: acceptance is only 0.25 of the score, proximity 0.60. Even a full recovery from 0.00 to 0.50 is worth 0.125, which may not outweigh a longer travel time. Wen's "against" argument in the TODO is real: someone who has stopped taking work should stay down until they take work again. |
| **Unintended consequences** | Decay lifts decliners as well as the four (declines are 18–21% of pings). It rewards inactivity. It could mask the cause for another year. |
| **Smallest experiment** | Replay at three decay rates and report days-to-recover. For each of the four, check whether a +0.125 change would move their rank given their real travel times. |

### A3. Guaranteed re-entry
| | |
|---|---|
| **Tries to solve** | A low-ranked responder is rarely reached (E8) so cannot rebuild a score. |
| **Supporting evidence** | E2, E3, E8. |
| **Missing evidence** | Whether the four are available, tagged and in range (stubs in the code). Any reason to expect the pings to be taken. The threshold and slot count. |
| **The case against** | **E4: every one of 13 post-gap pings to the four was missed or turned down.** Guaranteeing more such pings risks more failed offers. The threshold is derived from four responders. Re-entry overrides proximity, which wide-area responders asked us to weight up (E10). |
| **Unintended consequences** | Slower callouts; two rules instead of one score; the four could be failing for a reason unrelated to rank (signal, notifications, availability). |
| **Smallest experiment** | One responder, one week, manual priority for nearby, matching callouts through the existing override path (if Marcus confirms it can do this). Measure taken vs missed. |

### A4. Conclusion of the review
- **Best supported:** making quiet visible (5.4). E11 and E12 support it, it changes no routing, and its content can be drawn straight from the pings table.
- **Plausible, needs a replay and a real score first:** 5.1 and 5.2.
- **Weakest:** 5.3. The brief's own evidence (E4) says the pings it would add tend to fail.
- **Single biggest unknown:** the real scores for the four. We should read them before any of this is built.
