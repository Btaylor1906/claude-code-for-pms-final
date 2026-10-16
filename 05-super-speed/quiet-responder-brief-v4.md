# Product brief (v4): Quiet Responders, framed as a product problem

**Product:** Rook Dispatch  **Author:** PM, Dispatch  **For:** Helen Achebe (Director of Product); then Marcus Oyelaran, Wen Li, Sofia Marino, Nadia Hoffmann, Ravi Menon
**Status:** Draft for discussion, 16 Oct 2026. Nothing here is committed. It proposes what we build; it does not authorise a code change.
**Primary persona:** The Undertow (responder, Harborside), handler Desmond Okafor
**Constraint for this version:** Engineering has capacity for **one release containing one routing change**. Everything else is sequenced behind it.

> **Read this first: data caveat.** Every number comes from rook-database (pings and tickets, 29 Jun to 7 Sep 2026) and a replay of the routing code. The data ends five weeks ago. We have not seen real ranking scores, and we have no response-time or push-delivery data. Section 5.0 makes reading the real scores the first step and says what changes if they surprise us.

---

## 1. Executive summary

**The experience we want to deliver:** a responder who has done nothing wrong should never be left wondering whether Rook has dropped them. If their work dries up, they and their handler should be able to see it within days, be told why in plain words, and find themselves back to a fair share of callouts without having to do anything.

**What happens to responders today.** Since 4.2 (12 Aug), 4 of our 16 responders (The Undertow, Farlight, Meteor Mite, Vesper) have effectively stopped receiving callouts, and two more dipped 20–25%. The Undertow is the clearest case. He was an ordinary, reliable responder, pinged about 12 times a week for six weeks. In the week of 31 Aug he got 1 ping. His handler, Desmond Okafor, had checked his availability and tags, found nothing wrong, and filed 11 tickets asking us to "check yours." None has been answered. On 1 Sep Desmond passed on The Undertow's own words: *"nothing again this week. starting to wonder if im still even in the system."*

**Why this is more than a bug.** The responder cannot see where they stand or do anything about it. Their handler's console shows availability and tags, not offers or why they stopped. Support could explain one 4.2 complaint ("gone before I could answer") and not the other ("my phone never goes off"). And the one person who knows how the ping order works is a single engineer, with no written description. Correcting a number in the routing code is necessary, but it does not give anyone a way to notice, understand or trust what is happening. Section 3 sets out the argument.

**What we propose, in terms of what people will feel.**
- **First release (MVP):** a quiet responder climbs back into the rotation on their own within about two weeks, instead of staying stuck for as long as nobody takes a ping. At the same time, Support and handlers get a weekly list of who has gone quiet and why, so a handler like Desmond gets an answer in a day and a handler who never writes in (Meteor Mite's and Vesper's handlers filed nothing) hears from us first.
- **Next:** handlers see "quiet" and the facts on the console itself; we log what we need to explain any ping decision; and Helen decides the 60s ping wait, which is what started the misses.
- **Later, once we have evidence:** the responder sees on their own phone that they are still in the rotation, and we test whether giving a quiet responder priority offers actually works.

**How we will know.** No available responder stays under half their usual pings for more than 14 days; The Undertow is back to at least 75% of his usual pings within three weeks of release; every quiet ticket is answered within a business day; and acceptance, time-to-accept and coverage gaps do not get worse (section 6).

**What this does not do.** It does not revert 4.2, change the proximity weighting that wide-area responders asked for, or fix pings that are gone before a responder can answer. The first slide in score still happens; the MVP shortens the time stuck, it does not stop the slide. Section 10 names the one change that would most likely have prevented The Undertow's decline.

---

## 2. Who is this for? (Question 1)

### Primary users
| User | Why they are affected |
|---|---|
| **Responders who miss a few pings and then go quiet** (today: The Undertow, Farlight, Meteor Mite, Vesper; Halfmoon and Corporal Ashgrove dipped about 20–25%) | They are independent (Rook employs no responders). A drop in pings is a drop in the work they are offered, and they cannot see why. |
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

## 3. Problem statement: why this is a product problem, not just an algorithm bug

> **Helen's question:** "Why is this a product problem and not just an algorithm bug?"
>
> **Short answer:** the routing code is doing what it was written to do. A responder who did nothing wrong lost most of their work, could not see it happening, could not do anything about it, and nobody at Rook could tell their handler why. Changing a number fixes the arithmetic. It does not give a responder a way to know where they stand, a handler a way to explain it, or Support an answer to give.

### 3.1 What user problem exists today
A responder who misses a short run of pings is demoted in the order we ask people in, and stays demoted. The 4.2 release made missing more likely (the ping wait went from 90 to 60 seconds; misses rose from 2.3% to 18% of pings), and the rule treats a miss the same as a turn-down. So a reliable responder can fall to the bottom, is rarely offered a callout, and gets almost no chance to get back up.

This is not an edge case. 4 of 16 responders (25%) are affected, and none had recovered by 6 Sep.

**The Undertow.** Before 12 Aug he was pinged about 12 times a week, took about 77%, turned down about 21% and missed about 1%. In his first three days on 4.2 he missed 5 of 7 pings. After that his pings came after gaps of 3, 6 and 9 days, and each was missed. He went from 12 pings a week to 1. In plain terms: *a person with a phone in his pocket, who is available and tagged for the work, has stopped being asked, and has started to doubt that he is still a customer.*

### 3.2 Why responders cannot solve it themselves
- **They have nothing to check.** Desmond verified the two things a responder or handler can control: availability ("set correctly") and capability tags ("all there"). Both were fine.
- **No screen shows where they stand.** Neither the console nor the phone shows how many pings a responder received, which were missed, or how they are ordered. The Undertow's only evidence is that his phone is silent, which is why he asked whether his account was still active.
- **The only lever is the one they are not given.** The way to recover is to take a ping, and they are rarely being offered one (the four received 43, 16, 6, then 3 pings a week).
- **They cannot even tell it is happening to others.** Kip's two responders live in the same city: "Mite thinks Mite's been forgotten. Gale's exhausted."

### 3.3 Why handlers and Support cannot explain it today
- **The console does not show it.** Desmond's line, repeated across his tickets: *"Nothing looks wrong on our side. Could someone check yours?"* The Undertow's card shows availability and tags. It does not show offers, misses or ordering.
- **Support can explain only half of it.** Nadia Hoffmann on 18 Aug: the "gone before I could answer" theme is explainable (the wait was cut); *"the first one I can't."*
- **Handlers notice but cannot interpret.** Aunt Dot cooks for "someone who's home all week" and "couldn't tell you why." Kip: "I don't even know what I'd want it to tell me."
- **Nobody had the numbers.** The 18 Aug request for the trend got "something rough but not the real weekly numbers". The explanation that stuck ("seasonal", "should settle once September starts") had no figures behind it, and the tickets had started the day after the release.
- **Even Engineering could not say if it was intended.** Marcus Oyelaran asked on 14 Aug whether the proximity change was meant to apply to responders who turn pings down. It sat unanswered. There is no written description of how pinging is decided, and the one person who knows it, Wen Li, was away.
- **Some handlers never find out.** Meteor Mite's and Vesper's handlers filed no quiet tickets, yet both responders collapsed in the same way. Desmond's first ticket came 8 days after 12 Aug.
- **Result:** 11 open tickets about one responder, and no answer anyone can give.

### 3.4 Why fixing the routing algorithm alone is insufficient
1. **It is not a bug; it is a rule nobody has decided on.** The code treats a miss and a decline the same ("we asked and we didn't get a yes") and never lets a score recover; a 2019 note asks whether it should. What a miss should mean for who we ask next trades off fairness to responders, speed of callouts, proximity and coverage. That is a product decision with owners (Helen, Wen, Marcus). Engineering can change the number; the number is not the decision.
2. **We cannot yet prove the algorithm is the whole cause.** We have not seen the real scores or any response times, and The Undertow was always first or second when he was pinged. Push delivery is not ruled out. A change we cannot observe is a guess, and we are blind to the next one.
3. **The failure stays silent.** Anything that makes a responder go quiet again (a bad signal week, a tag change, a restart that clears the score) will look the same: no one told. A routing fix alone leaves no way for anyone to notice in days instead of weeks.
4. **A responder who has waited weeks needs to hear it is resolved.** The question was "am I still in the system?". A corrected number does not answer it, and the open tickets do not close themselves.
5. **Even a good fix does not undo the first slide.** A run of misses still lowers the score; recovery takes days or weeks. During that time the responder and handler need an explanation, not silence.

The algorithm change is in the plan (the MVP). It is necessary and not sufficient.

### 3.5 The outcome we are trying to create
A responder who goes quiet is **noticed** within a week, **explained** in plain words to them and their handler, and **returned** to a fair share of offers without having to do anything. Section 6 turns this into measures.

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

## 5. Proposed solution: MVP, Phase 2, Future

### Evidence index (all from sections 2–4)
| ID | Evidence | Confidence |
|---|---|---|
| E1 | Pooled missed pings 2.3% → 18.0% after 12 Aug; declines flat (21.1% → 18.0%) | High |
| E2 | Four responders' weekly pings 43, 16, 6, 3; about 59% of remaining pings missed (about 3% before) | High |
| E3 | None of the four had recovered by 6 Sep | High |
| E4 | All 13 pings the four got after a 2+ day gap were missed or turned down; gaps kept lengthening | High |
| E5 | `history.py`: a miss and a decline both cost 0.12; a taken ping +0.08; no decay; a 2019 TODO left open | High |
| E6 | Replay: the four went from 0.84–1.00 to 0.00; the others recovered to 0.76–1.00 | Medium (real scores unseen) |
| E7 | The other 12 responders' miss rate rose to 10–14% and they recovered under the same rules | High |
| E8 | Ping chains are short (1.4 before, 1.66 after, longest 4), so rank 3 or lower is rarely reached | High |
| E9 | The Undertow was always 1st or 2nd when pinged, so "ranked low" vs "rarely a candidate" is open | Medium |
| E10 | 4.2 weights: proximity 0.60, recent acceptance 0.25, capability 0.15 | High |
| E11 | Desmond Okafor's 11 open tickets; "Nothing looks wrong on our side. Could someone check yours?"; "starting to wonder if im still even in the system" | High |
| E12 | Kip: two cards "might as well be two different products"; Meteor Mite thinks they've been "forgotten" | Medium (one handler) |
| E13 | Declines do not predict collapse; 4+ misses in 7 days did (four responders only) | Medium |
| E14 | A 14 Jul ticket: a ping missed while The Undertow was out of signal | Low |
| E15 | Meteor Mite and Vesper filed no quiet tickets but show the same collapse; Desmond's first ticket came 8 days after 12 Aug | High |
| E16 | Code: a ping is marked missed only when the offer timer runs out (`offer.py`). Ranking decides who is asked, not whether they answer in time. The Undertow's miss rate was about 1% (1 of about 75) before 12 Aug | High |

### Metrics every item is measured against
| ID | Metric | Baseline (to 6/7 Sep) | Target (proposal; Ravi Menon and Wen Li to confirm) |
|---|---|---|---|
| M1 | Available, tagged responders with 2 consecutive weeks under 50% of their own baseline | 4 of 16 | 0 sustained for more than 14 days |
| M2 | A quiet responder's weekly pings vs their own 6-week baseline | The Undertow: 12/week → 1 | At least 75% within 3 weeks of release (adjusted for total callouts) |
| M3 | Days from falling under 50% of baseline to getting back over it | Not recovered after 3+ weeks | MVP: 14 days or less. Phase 2: 72 hours or less |
| M4 | Missed-ping rate by responder | Four at about 59%; pooled 2.3% → 18% | No responder over 25% in a rolling 14 days. Pooled return toward 2% depends on the ping wait (Phase 2) |
| M5 | Quiet-theme tickets: volume, and share answered within 1 business day | 11 open for The Undertow; about 4× baseline at peak | Volume back to baseline; 100% answered |
| M6 | Guardrails: acceptance rate, time-to-accept, coverage gaps, chain length (1.66), median travel time of the taker, the other 12's decline rate and ping volume | Current | No worse |

### 5.0 Before building: read the real scores (no code)
The MVP's routing change is chosen on the assumption that the four really are at the floor. Check this first.
1. **Wen Li or Marcus Oyelaran read the live recent-acceptance score** for the four (answers E6, E9).
2. **Confirm whether the score is stored or lives only in memory** (`_scores` in `history.py`) and whether it survives a restart. If it resets on restart, decay semantics change.
3. **Check The Undertow's availability, region and travel time** in the quiet weeks.
4. **Ravi re-pulls the weekly numbers from 7 Sep**, split by turned down vs missed. Did the four recover?
5. **Replay** the pings table with candidate decay rates.

**Decision rule.** If the four are at or near 0.00 and a replay shows decay brings them back: build the MVP as written. If their real scores are *not* low (so score is not what is holding them back): the MVP routing change becomes the ping-wait decision (section 10) because misses are set by the clock (E16), and decay would not help.

### 5.1 MVP: the smallest solution that is worth shipping

**MVP-1 (the one routing change): the recent-acceptance score eases back toward neutral over time.**
- **What:** each day, a score above or below `NEUTRAL_SCORE` (0.5) moves a fixed step toward it and never past it. Declines keep their 0.12 cost; taken pings keep their +0.08. The step size is picked by replay, not by hand.
- **Why it belongs in the MVP:**
  - It is the only change of the three candidates that lifts someone who is *already* stuck. The four are at the floor now (E6). A smaller miss penalty only helps once they get a ping, and they are rarely getting one (E2, E8).
  - It works whatever the cause of the misses: the 60s wait, a no-signal spot (E14), or something we have not found. We do not have to settle the cause first.
  - It answers the 2019 TODO in `history.py` (E5), which is also the note Helen is referring to.
  - It is a small, contained change: one function and one config value.
- **Metric it should move:** M3 and M1 first; M2 follows.
- **How we would know it worked:**
  1. *Predicted vs actual.* Replay gives a days-to-recover for each of the four. After release, the four's weekly pings should rise on that schedule.
  2. *Against the other 12 as a control.* Their ping volume and decline rate should not move (M6).
  3. *Counts.* M1 falls from 4 to 0 sustained within about three weeks; The Undertow's weekly pings are back to 9 or more (75% of 12).
  - **Failure signals:** the four's pings do not rise (the score was not the cause, or the 0.25 weight is too small to change the order, E10); coverage gaps or time-to-accept get worse (M6).
- **What would stop it:** a real score that is not low (5.0); a replay showing no change in the order for the four.

**MVP-2 (no engineering): a weekly "quiet list".** Ravi Menon sends Nadia Hoffmann and the handlers a list every week: responders whose pings this week are under half their own 6-week average, with last-pinged date and missed vs turned-down counts.
- **Why it belongs in the MVP:** it costs no engineering time, uses a table we already have, and gives Support an answer to "Could someone check yours?" (E11). It also covers the handlers who never write in (E15: Meteor Mite and Vesper filed nothing; Desmond's first ticket came 8 days after 12 Aug).
- **Metric:** M5 first, and it shortens the time to notice.
- **How we would know:** Nadia can answer every quiet ticket within a business day; the open Undertow tickets get closed; the number of days between a dip starting and anyone noticing falls from about 8 to 7 or fewer.
- **Limit:** it only reports. It does not fix anything, and its wording must stick to facts from the pings table, not a cause.

### 5.2 Phase 2: enhancements (later releases, in this order)

| # | Item | Why it is Phase 2, not MVP | Metric | How we would know |
|---|---|---|---|---|
| 2a | **Log the score and the response time at each ping** (small engineering change) | Without it we keep guessing (E6, E9, E14). Not needed for the first fix, but needed to judge everything after it. | Supports all (M3, M4); enables measurement | Weekly report shows each responder's score and response time. Wen can answer "why was The Undertow ranked there?" in minutes |
| 2b | **Handler console "quiet" line** on the coverage card: pings this week vs usual, last pinged, missed vs turned down. Facts only, no cause. | It needs design and UI engineering, and the MVP-2 list covers the same need in the meantime. It answers Kip's "two cards, two products" (E12). | M5, plus handler comprehension | Sofia Marino tests with Desmond and Kip: can they say what is going on? 4 of 5 handlers. Tickets drop |
| 2c | **A smaller penalty for missed pings** than for declines | It protects reliable responders from future slides, but we have no evidence for the size, and E7 says the penalty alone does not explain the four. Needs 2a data first. | M4 and M1 (prevention) | Replay predicts the effect; in production, fewer responders drop under 50% after a miss spike. Decline rate and time-to-accept do not worsen |
| 2d | **Decision on the 60s ping wait** (Helen Achebe; "Ping timeout tuning" on the Q3 roadmap) | It is a product decision, not a build, and the MVP must not wait for it. It is the direct cause of the miss spike (E1, E16). | M4 (pooled misses) and "gone before I could answer" tickets (about a third of volume) | Misses by responder before and after; time-to-accept; the "gone" tickets fade |

### 5.3 Future improvements (need evidence first)

| Item | Why it is Future | Metric | How we would know |
|---|---|---|---|
| **Guaranteed re-entry** for a quiet responder | E4: all 13 pings the four got after a gap were missed or turned down. This may only add failed offers. Needs a one-week experiment on one responder first (through the routing override, if it can do this). | M2, M3 | The experiment's taken rate for re-entry pings is close to everyone else's. Coverage gaps and time-to-accept do not worsen |
| **Responder phone status** ("You're available. Last offer: 9 days ago.") | There is no responder interview in the evidence; all of The Undertow's voice is second-hand through Desmond (E11). Must come from the Responder Availability Record. | M5 (indirectly) | Interview a quiet responder first (Sofia). After release, responders stop asking Support whether they are "still in the system" |
| **Per-responder or adaptive ping wait** | Depends on response-time logging (2a). | M4 | Responders who need longer miss less without slowing callouts |
| **Revisit the weights** (0.60/0.25/0.15) | In the replay they hurt responders far from callouts, but 4.2 was asked for. Needs a decision on what we want proximity to do. | M2, M6 (median travel time) | Replay of alternatives; wide-area responders' feedback |
| **Handler alert when a ping lands** (Aunt Dot, Ambrose) | A different problem: pings gone before the responder can answer. | "Gone before I could answer" tickets | Fewer such tickets; Ambrose-type near misses |
| **Mutual aid, shared cover, Availability Confidence** | Already on the Q4 exploration list or unassigned from Q3. | n/a | n/a |

### 5.4 One-page summary
| Phase | Items | Engineering cost |
|---|---|---|
| **MVP** | Decay (the one routing change); weekly quiet list | Small; the list is none |
| **Phase 2** | Score and response-time logging; handler quiet line; smaller miss penalty; ping-wait decision | Small to medium; a decision |
| **Future** | Re-entry; phone status; adaptive wait; weights; handler ping alert | Each needs evidence first |

---

## 6. What changes for people once it ships? (Question 2)

*Written to answer Helen's question: what is the experience outcome, and how do we know we delivered it, not just that a number changed?*

### 6.1 Four experience outcomes
| Outcome | Who feels it | In their words | Delivered by | Measured by |
|---|---|---|---|---|
| **Noticed**: a quiet responder is spotted within a week | Handler, Support | "I didn't have to write four tickets to find out." | MVP-2 (weekly quiet list); Phase 2b (console line) | M5; days from a dip starting to someone noticing (about 8 today; target 7 or fewer) |
| **Explained**: handler and Support can say what is happening | Handler, Support | "Nothing looks wrong on our side" becomes an answer | MVP-2; Phase 2a (score and response-time logging); Phase 2b | M5: 100% of quiet tickets answered within a business day |
| **Returned**: back to a fair share of offers without doing anything | Responder | "The phone started going off again." | MVP-1 (decay) | M1, M2, M3 |
| **Reassured**: the responder knows they are in the rotation | Responder | "I'm not forgotten." | Future (phone status), after we interview a quiet responder | Responders stop asking whether they are "still in the system" |

### 6.2 What it feels like: The Undertow and Desmond
| Moment | Today | After the MVP | After Phase 2 |
|---|---|---|---|
| Five misses in three days (12–14 Aug) | The slide starts | Same | Same (the first slide is not prevented) |
| The following weeks | Silence; 4, then 1, then 1 pings a week | Pings come back within about 14 days as the score eases toward neutral | Same, and the console says "quiet" with facts |
| Desmond looks for an answer | "Nothing looks wrong on our side" | The weekly quiet list names The Undertow; Nadia replies within a business day | He sees it on the card without asking |
| The Undertow on his phone | "starting to wonder if im still even in the system" | Pings return; no new screen yet | Same; the phone status comes later, once we have talked to him |
| A ping after a long gap | The first in 9 days; gone before he could answer | Gaps shorten, so pings are no longer rare, high-pressure events | Same |

### 6.3 Handlers and Support
- **Desmond Okafor:** instead of eleven tickets and no answer, he gets a plain explanation the week it happens.
- **Kip:** his two cards stop being "two different products" without explanation. He sees which responder is quiet and the facts (Phase 2b).
- **Nadia Hoffmann:** a line she can paste on the open tickets; fewer tickets, since handlers no longer need to ask.
- **Handlers who never write in** (Meteor Mite's and Vesper's): hear from us first.

### 6.4 Success criteria
All targets are proposals for Ravi Menon and Wen Li to confirm on fresh data.

**Primary (what a responder would notice)**
| Metric | Baseline (to 6/7 Sep) | MVP target |
|---|---|---|
| M1: available, tagged responders under 50% of their own baseline for 2 consecutive weeks | 4 of 16 | 0 sustained for more than 14 days |
| M2: a quiet responder's weekly pings vs their baseline | The Undertow: 12 → 1 | At least 75% within 3 weeks |
| M3: days from falling under 50% to getting back over it | Not recovered after 3+ weeks | 14 days or less |

**Supporting (what Support and handlers would notice)**
- M5: quiet-theme tickets back to baseline (about 4× at peak) and 100% answered within one business day; the 11 open tickets closed with an explanation.
- Days from a dip starting to someone noticing: from about 8 to 7 or fewer.
- Handler comprehension (Phase 2b): 4 of 5 handlers can say why a responder is quiet from the console.

**Guardrails (must not get worse):** acceptance rate, time-to-accept, coverage gaps, average ping chain (1.66), median travel time of the taker, the other 12 responders' ping volume and decline rate (M6).

**What would tell us it did not work:** the four's pings do not rise on the replay's schedule; Support still cannot explain a quiet responder; a guardrail moves the wrong way.

### 6.5 What the MVP does not yet deliver
- **"Reassured" is not in the MVP.** The responder sees nothing new on their phone. We will not build it before we have spoken to a quiet responder; everything we know about The Undertow's feelings comes through his handler.
- **The first slide is not prevented.** A run of misses still lowers the score at the same rate; the MVP shortens the time stuck.
- **"Explained" in the MVP is a weekly report, not a product feature.** It is the cheapest way to start answering Desmond, and it depends on Ravi and Nadia, not on a release.

---

## 7. What does it not do? (Question 3)

### Non-goals (MVP)
- **Does not stop the first slide.** A run of misses still lowers the score at 0.12 each; only the time stuck changes.
- **Does not revert 4.2** or change the proximity weight (0.60), which wide-area responders asked for.
- **Does not change the 60s ping wait.** That is a Helen decision (Phase 2d); until then "gone before I could answer" tickets stay (about a third of volume; Ambrose, Halloran, Aunt Dot).
- **Does not change who is eligible.** Ranking still orders; it never removes anyone.
- **Does not guarantee work.** No minimum number of callouts and no re-entry slot (Future).
- **Does not touch declines.** The decline cost is unchanged, and chronic decliners are watched through M6.
- **Does not identify anyone.** No link between a cover identity and a legal one (Security Policy 4.1). No responder sees a score or rank.
- **No console or phone UI.** The quiet list is a weekly report, not a product feature.

### Still unsolved after the MVP
- The ping wait and misses it creates (pooled 2.3% → 18%; the other 12 sit at 10–14%).
- Whether some "phone never goes off" cases are push-delivery failures. We have no delivery data (E14).
- The weights, mutual aid, shared cover, Availability Confidence (committed for 4.2, never shipped, unowned).
- Console asks from interviews: dark mode (Kip), text size (Aunt Dot, Ambrose), per-responder alert sounds (Kip), a filter-reset warning (Ambrose, Desmond).
- No written description of how pinging is decided (Priya's request).

---

## 8. Risks and assumptions

**Assumptions**
1. The pings table is the full history, so the replay scores are close to real ones (unverified; step 5.0.1 tests it).
2. The four are available, tagged and in range, and are low because of the score. We cannot see availability or travel time.
3. Recent acceptance is 0.25 of the score (E10), so moving it may not change rank for responders far from callouts.
4. The score lives in memory and may reset on restart (unverified).

**Risks**
| Risk | Mitigation |
|---|---|
| The score is not what holds the four back (E9: The Undertow was always 1st or 2nd when pinged) | Gate in 5.0; if scores look normal, switch the one routing change to the ping wait |
| Decay lifts chronic decliners (declines are 18–21% of pings) and anyone who stopped taking work, the case in the 2019 TODO | Decay stops at neutral and never rises above it; watch the decline rate (M6) |
| Recovered responders are missed again, costing callout time (E4) | Watch M4, M6 time-to-accept and coverage gaps; the flag can be turned off |
| Too slow a rate does nothing; too fast erases the signal | Rate chosen by replay against days-to-recover |
| The quiet list is read as a performance judgement | Facts only, no scores or ranks; Nadia and Sofia review the wording |
| Fixing recovery hides the 60s problem | Phase 2d is on the roadmap with an owner (Helen) and M4 is reported by responder |
| Only Wen knows the code, and has not answered Marcus's 14 Aug question | Book Wen and Marcus for step 5.0 this week |
| The data is five weeks old | Ravi re-pulls before sizing |

---

## 9. Open questions

| # | Question | Who |
|---|---|---|
| 1 | What are the real current scores for the four? Is the score the thing holding them back? | Wen Li, Marcus Oyelaran |
| 2 | Is the score stored across restarts, and was decay deliberately left out in 2019? | Wen Li |
| 3 | Was the 4.2 weighting meant to apply to responders with a decline history? Was the 60s wait a deliberate trade-off, and why? | Wen Li, Helen Achebe |
| 4 | What do the weekly numbers show from 7 Sep to today, split by turned down vs missed? | Ravi Menon |
| 5 | Is The Undertow available and in range, and did the routing override audit log record manual overrides for the four since 4.2? | Marcus Oyelaran |
| 6 | Is any of this push delivery rather than ranking? | Marcus, Wen |
| 7 | Can Ravi produce the weekly quiet list this week, and can Nadia use it on the 11 open tickets? | Ravi Menon, Nadia Hoffmann |
| 8 | Can we interview The Undertow or another quiet responder, and Desmond Okafor? | Sofia Marino |
| 9 | Which squeezed-out Q3 items (Availability Confidence, the rest of Ping timeout tuning) are still commitments, and who owns them? | Helen Achebe |
| 10 | What happened in 4.3 and the Q4 roadmap? The wiki is quiet after 28 Aug. | Helen Achebe |

---

## 10. The single smallest change that would most likely have prevented The Undertow's decline

**Keeping the ping wait at 90 seconds** (`OFFER_TIMEOUT_SECONDS` in `config.py`: one constant, the line marked "was 90 until 4.2").

**Why this one, from the evidence:**
1. **The decline started with misses, and misses are set by the clock.** A ping is marked missed only when the offer timer expires (E16). The new ranking weights decide *who is asked*, not whether a person answers in time, so they cannot by themselves turn a ping into a miss. The Undertow missed about 1 of 75 pings before 12 Aug and **5 of his first 7 after it** (12–14 Aug).
2. **The first collapse happened while his score was still high.** The replay has him at 0.84–1.00 on 12 Aug (E6), so the first run of misses came before any drop in ranking. The wait, not the ranking, is what started the chain.
3. **Every later step follows from the misses.** Each miss cost 0.12 (E5); with no decay he stayed low; with short chains (E8) he was rarely reached; the pings that did arrive were missed (E4).
4. **Hand check against the next candidate.** Using his own ping sequence from 12 Aug to 5 Sep, a miss penalty of 0.04 instead of 0.12 would leave him near 0.64 (assuming he started at 1.0), so a smaller miss penalty probably also prevents the slide. The wait is still the more direct cause, because it removes the misses rather than softening their cost.

**Why it is not the MVP:**
- It prevents; it does not repair. The four are already at the floor (E6) and moving the wait back would stop new misses but lift no one from 0.00. Decay does.
- It is a product decision on a Q3 item ("Ping timeout tuning") that Helen owns, not Engineering's one routing change to spend, and we do not know why it was cut to 60s.
- It would not remove the "gone before I could answer" complaints from the other 12, who may simply need a longer window; it addresses the cause for the quiet four only if the misses were slow answers, not no-signal or delivery failures (E14), which we cannot rule out without response-time data.

**What we cannot prove.** The wait was changed in the same release as the weights, so the two are confounded. Code logic (E16) and The Undertow's own before/after miss rate favour the wait, but a replay cannot show he would have answered in 90s. Phase 2a (response-time logging) is what would settle it.

**If Helen only gets one change and the real scores in 5.0 turn out not to be low:** spend it on the ping wait.


---

*Appendix A reviewed the v2 draft. Its numbering maps to this version as: 5.1 miss penalty = Phase 2c; 5.2 decay = MVP-1; 5.3 re-entry = Future; 5.4 handler visibility = MVP-2 (as a report) and Phase 2b (as UI).*

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
