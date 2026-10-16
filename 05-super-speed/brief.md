# Quiet Responders: one-pager

**Rook Dispatch · PM, Dispatch → Helen Achebe · 16 Oct 2026 · Draft, nothing committed**
*Data: rook-database to 7 Sep 2026 plus a replay of the routing code. Real ranking scores, response times and push delivery are unseen.*

## The problem
A responder who did nothing wrong can lose most of their work after 4.2, with no way for anyone to see it or fix it. **The Undertow** (handler Desmond Okafor) went from about 12 pings a week to **1**. His handler checked availability and tags, found nothing wrong, and filed **11 tickets** asking us to "check yours." All are open. On 1 Sep: *"starting to wonder if im still even in the system."* 4 of 16 responders (The Undertow, Farlight, Meteor Mite, Vesper) are affected, and none had recovered by 6 Sep.

## Why it's a product problem, not just an algorithm bug
- **Responders can't see or fix it.** No screen shows offers, misses or where they stand. The only way back is to take a ping, and they aren't being offered any.
- **Handlers and Support can't explain it.** Nadia Hoffmann could explain "gone before I could answer" but not "my phone never goes off." Nobody had weekly numbers, and Marcus Oyelaran's 14 Aug question is still unanswered. Meteor Mite's and Vesper's handlers filed nothing at all.
- **The code does what it was written to do.** A missed ping costs 0.12, the same as a decline. A taken ping earns back 0.08. The score never recovers on its own. Wen Li's 2019 TODO asks whether it should. What a miss should mean is a product decision.
- **A number change alone leaves the failure silent**, and it can't be verified without scores we haven't seen.

## What triggered it
Misses rose from **2.3% to 18%** of pings after 12 Aug, when the ping wait was cut from 90s to 60s. The Undertow missed about 1 of 75 pings before and 5 of his first 7 after. Declines were flat. Replay: the four fall from 0.84–1.00 to 0.00. They then rank low, rarely get offered a ping, and can't climb back. All 13 pings they got after a long gap were missed or turned down.

## The experience we want
A responder who goes quiet is **noticed** within a week, **explained** in plain words, and **returned** to a fair share of offers without doing anything.

## Plan (Engineering: one release, one routing change)
| Phase | What | Moves |
|---|---|---|
| **MVP** | **Score decay:** the recent-acceptance score eases toward neutral (0.5), never past it; rate set by replay. **Weekly quiet list** from Ravi Menon to Nadia and handlers (no engineering). | Returned; noticed |
| **Phase 2** | Log score and response time per ping; "quiet" line on the handler console; smaller penalty for missed pings; **Helen decides the 60s wait** | Explained; fewer misses |
| **Future** | Responder phone status (after interviewing a quiet responder); guaranteed re-entry (needs a test first); adaptive ping wait; revisit weights | Reassured |

**First step, no code:** Wen or Marcus reads the real scores for the four. If they aren't low, the one routing change becomes the ping-wait decision.

## Success (proposed; Ravi and Wen to confirm)
- No available responder stays under 50% of their usual pings for more than **14 days** (today 4 of 16).
- The Undertow back to **75%+** of his usual pings within 3 weeks.
- Every quiet ticket answered within **1 business day**.
- Guardrails flat: acceptance rate, time-to-accept, coverage gaps, other responders' volume.

## Not doing
No revert of 4.2 or the proximity weight. No fix for pings gone before a responder can answer. No stopping of the first slide: the MVP shortens the time stuck. No link between cover and legal identity, and no scores shown to responders.

## Risks and open questions
- Score may not be what holds the four back (The Undertow was always 1st or 2nd when pinged). Does the score survive a restart? *(Wen)*
- Decay also lifts chronic decliners. Watch the decline rate.
- Was the 60s wait deliberate? *(Helen, Wen)* Is any of this push delivery? *(Marcus)*
- Data is five weeks old. *(Ravi to re-pull)* No responder has been interviewed. *(Sofia Marino)*

## Smallest change that would most likely have prevented it
**Keep the ping wait at 90s** (one constant in `config.py`). A ping is marked missed only when the timer expires, so the weights can't cause misses. The Undertow's score was still high when his misses began. It prevents but doesn't repair, and it's Helen's decision, so decay is the MVP.
