# Product brief: Quiet Responders — "Still in the system?"

**Product:** Rook Dispatch  **Author:** PM, Dispatch  **For:** Helen Achebe (Director of Product); then Marcus Oyelaran, Wen Li, Sofia Marino, Nadia Hoffmann, Ravi Menon
**Status:** Draft for discussion, 16 Oct 2026. Nothing here is committed. It proposes what we build; it does not authorise a code change.
**Primary persona:** The Undertow (responder, Harborside), handler Desmond Okafor

> **Read this first: data caveat.** Every number below comes from rook-database (pings and tickets, 29 Jun to 7 Sep 2026) and the replay of the routing code. The data ends five weeks ago. We have not seen real ranking scores, and we have no response-time or push-delivery data. Ravi's weekly numbers and Wen's answers (see Open questions) should be confirmed before anything is scoped as final.

---

## 1. Executive summary

Since 4.2 (12 Aug), some responders have stopped being offered callouts, and nobody, including them, their handlers and Support, can see why. The Undertow went from about 12 pings a week for six straight weeks to **1 ping in the week of 31 Aug**. His handler, Desmond Okafor, filed **11 tickets** about it between 20 Aug and 7 Sep. All are still open, and Support has no answer to give him.

The cause is a design choice in the routing code that nobody has revisited. `history.py` treats a **missed** ping (the ping wait ran out) exactly like a **turned down** one, costs 0.12 each time, gives back only 0.08 for a taken ping, and has **no decay**. 4.2 cut the ping wait from 90s to 60s, so misses jumped from 2.3% to 18% of pings. Four responders (The Undertow, Farlight, Meteor Mite, Vesper) slid to a score of 0.00. Because they rank last, they are rarely offered anything, so they have almost no chance to earn the score back. Wen left a 2019 TODO next to that code asking whether this should ease back on its own. It was never answered.

**Proposal:** three connected changes, built in this order.
1. **Routing: a quiet responder can always climb back.** A missed ping costs less than a turned-down one, the score drifts back toward neutral over time, and an available responder who has gone unusually quiet is guaranteed an early place in the offer order.
2. **Handler console: make quiet visible.** A "quiet" flag and a plain-words reason on the coverage card, so Desmond and Kip can see what is happening and Support has an answer.
3. **Responder phone: tell the responder they are still in the rotation.** A short status line, so The Undertow does not have to guess whether his account is still active.

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

## 5. Proposed solution

### 5.1 Routing: a quiet responder can always climb back *(Engineering, with Wen Li)*
1. **Misses are not declines.** Keep the 0.12 penalty for a **turned down** ping. Use a smaller penalty for a **missed** one. Starting hypothesis only: 0.04–0.06. This must be tested by replay, not picked by hand.
2. **The score eases back toward neutral over time.** A fixed amount per day (not past `NEUTRAL_SCORE`, 0.5), so a bad run cannot be carried forever. This answers the 2019 TODO. Its "against" argument (someone who stopped taking work should stay low until they take work again) is met by stopping at neutral and by keeping declines costly. The four did not stop taking work: The Undertow took 9 of 12 in the week before 4.2.
3. **Quiet-protection: guaranteed re-entry.** If a responder is **available** and has the **right capability tags**, but their pings over the last 7 days are below a set share of their own 6-week baseline (starting point: 50%), they are placed in the top two of the offer order for matching callouts within the proximity horizon until they have had a fair number of offers. Ranking stays one score for everyone; this is a bounded bump, not an exemption. A responder who then keeps declining still ranks down.
4. **Persist the score.** If the score lives only in memory (the code says it does), a restart wipes it. Wen to confirm.

### 5.2 Handler console: make quiet visible *(Design: Sofia Marino)*
On each responder's coverage card, next to availability:
- A **"Quiet" badge** when the responder is below the baseline (same rule as 5.1.3).
- A one-line plain-words reason: *"1 ping in the last 7 days (usual: 12). Last 4 pings were missed. Ranked lower after recent missed pings. Working back up: next offers are prioritised."*
- A **"Last pinged" time** and the share of pings **missed vs turned down** this week, so Desmond can tell "ranked low" from "broken".
- The same wording available to Support (Nadia) to paste into replies, so the 11 open tickets can be closed with an answer.
This responds directly to Kip's two cards "that might as well be two different products." Nothing on the card exposes a cover identity or a legal one.

### 5.3 Responder phone: you are still in the rotation *(Design: Sofia Marino)*
A short status on the responder's phone home screen: *"You're available. You're in the offer list. Last offer: 9 days ago."* After a long quiet spell, the first re-entry ping looks the same as any other. The status never shows a score or rank. It exists to answer *"am I still even in the system?"* in the product rather than in a support ticket.

### 5.4 What the Director asked for
Helen asked for something a handler like Kip would notice and a responder who has gone quiet would feel differently about. Kip notices the badge and reason. The Undertow feels it on his phone. A clickable prototype of 5.2 and 5.3 (Sofia) is the next deliverable if Helen wants one; this document does not include it.

---

## 6. What changes once it ships? (Question 2)

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
