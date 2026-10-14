# Rook Industries — course working file

## Session scope — Product School lab

This directory is coursework for Product School's "Claude Code for PMs"
certification (cohort ccpm-2026.1). Everything in it is a fictional
teaching scenario.

- Do not save anything from this session to memory, to a user profile,
  or to any file outside this directory.
- Do not carry context from this directory into unrelated sessions.
- Rook Industries is not a real company. Nothing here is a fact about
  the world.
- Read and write only within this directory.
  Exceptions, for the course-setup and wrap-up skills only:
  - When the student asks you to check their setup, save their work or wrap up a session, that request is their yes. You may run the GitHub command-line program installed at ~/.ccpm/gh for those checks and saves, and look in that folder to find it.
  - For a repair, first tell the student in one plain sentence what you are about to do, and act only after they say yes. Repairs may: run that GitHub program (including setting this folder's own git sign-in setting and changing this repo's visibility back to Public); copy the student's own course files into this directory from another folder on their computer (copy only; never move, edit or delete the originals); and rename something outside this directory that blocks setup, by adding "-old" to its name (never delete it).
  Outside this directory you still never write, edit or delete anything else.

<!-- Keep the block above at the top of this file. Everything you add
     during the course goes below this line. -->

---

## Working context

Sources: rook-wiki (About Rook, Glossary, Team directory, Releases, Q3 roadmap,
Product briefs, 4.2 page comments) and `00-rook/company/notes/handoff-from-priya.docx`.
I am the PM for **Rook Dispatch**, taking over from Priya Raghunathan (left 21 Aug 2026).

### Company and products
- Rook Industries: coordination and provisioning software for the protective-response
  sector. Customers are independent masked responders and their handlers/quartermasters
  (Rook employs no responders). Subscription, priced per active responder. HQ Site Aleph;
  offices in Berlin, Singapore, Cornwall. Ships monthly; point releases are 4.x.
- **Rook Dispatch** (mine): availability, proximity, who gets pinged for a callout, whether
  they take it. Users: handlers (console, desktop-only) and responders (phone).
- **Rook Supply**: requisitions, maintenance, field failure reports. Users: handlers, quartermasters.
- **Confidentiality:** we never store or map cover identities to legal identities (Security
  Policy 4.1). Never design anything that assumes we can; never try to work out who anyone is.

### People (wiki roles -> names)
| Role in wiki comments | Person | Notes |
| --- | --- | --- |
| Engineering manager | Marcus Oyelaran (Site Aleph) | Runs Dispatch eng. Straight talker; start here when unsure |
| Staff engineer | Wen Li (Berlin) | Built the ping-ranking logic; no doc exists, talk to her |
| Support lead | Nadia Hoffmann (Berlin) | Owns tickets; hears handler complaints first; standing 15 min |
| Product designer | Sofia Marino (Site Aleph) | Console and phone app; ran the September interviews |
| Director of Product | Helen Achebe (Site Aleph) | My director; owns roadmap and commitments |
| Data analyst | Ravi Menon (Singapore) | Weekly acceptance numbers. Priya pointed me to Marcus for numbers, but Ravi owns the real ones |
- Priya Raghunathan: previous Dispatch PM, gone; owner of every roadmap item.
- Handlers named in wiki: Aunt Dot (Vesper), Mr. Ambrose (Captain Vantage), Halloran (Sgt. Bulwark,
  also a Supply voice on the gear cage/requisitions), Kip (Meteor Mite, The Gale).

### Vocabulary (Glossary)
- **Callout** = request for a responder to attend an incident. **Ping** = a callout offered to one
  responder's phone. Outcomes: **taken**, **turned down**, **missed** (ping wait expired); both
  of the latter move the ping to the next responder but are recorded separately.
- **Ping wait**: how long a ping stays on the phone before counting as missed; same for everyone.
- **Acceptance rate** (taken / pings; Dispatch headline metric); **time-to-accept** (median secs, ping to taken).
- **Coverage gap**: no available responder had the required capability tags. Not the same as low acceptance.
- **Routing priority**: score ranking responders. Inputs: proximity (travel-time estimate),
  availability, capability match, recent acceptance history. Turning down or missing pings lowers
  the acceptance component, which lowers later rank.
- **Capability tags**: flight, structural-entry, hazmat-tolerant, cold-weather, aquatic,
  crowd-management, de-escalation. **Responder Availability Record**: written by Dispatch, read by Supply.
- **Mutual aid**: cross-area cover; unsupported, on Q4 exploration list.
- Supply: **requisition** (handler request, quartermaster approves), **field failure report**, **service interval**.

### Where things stand
**Releases:** 4.0 (7 Apr): new nav, profile redesign, routing-override audit log. 4.1 (16 Jun):
travel-time proximity, bulk callout, push reliability. **4.2 (12 Aug):** proximity weighted up vs.
recent acceptance; ping wait cut 90s -> 60s; filters persist; 3 defect fixes. No rollback.

**The 4.2 problem (top priority).** Since 4.2, callout tickets are ~3x normal and still elevated
as of 26 Aug. Split roughly two thirds "phone never goes off" to one third "gone before I could
answer." Support can explain the second (60s wait); nobody has explained the first. A handler
emailed Nadia directly, which never happens.
- Open question from Marcus (14 Aug), never answered: did the proximity change deliberately
  apply to responders who have been turning pings down, or did it just fall out of the config?
  Wen was away 14-24 Aug and has not replied in the thread.
- Priya's and Marcus's view is "August seasonal dip, will recover in September." This is
  **unverified**: no weekly numbers have been presented. Two changes shipped together (ranking and
  ping wait), so they are confounded. Hypothesis to test, not a conclusion: a responder who
  misses or declines pings gets a lower recent-acceptance score and drops down the order; with the
  4.2 weighting that may leave some responders rarely or never offered pings (the "phone never goes
  off" theme), and the shorter wait may make more of them miss, deepening the slide. Get Ravi's weekly acceptance data, split
  by turned-down vs. missed, before and after 12 Aug.
- Priya: don't let this become a revert-4.2 debate (change was a long-requested ask from wide-area
  responders). Marcus: don't hand the new PM a conclusion; regroup on 4.2 once I have looked.
  Nadia has the ticket breakdown ready.

**Q3 roadmap** (last reviewed 30 Jun, before 4.2; stale, all items owned by Priya, need new owner):
- Change to who gets pinged: 4.2, Committed (shipped)
- Ping timeout tuning: 4.2, Committed (the 90->60 cut shipped; unclear if that is the whole item)
- **Availability Confidence** (confidence score next to stated availability): 4.2, Committed,
  **not in the 4.2 release notes**; likely squeezed out
- Requisition approval chains: 4.3, Committed (Supply)
- Handler phone app: Q4, Exploring. Shared cover between responders: Q4, Exploring
- Roadmap says changes to committed items go through Product. Priya flagged that I need to sit
  with Helen to settle which of the squeezed-out items are still Q3 commitments. That has not happened.

**Briefs** (all one-pagers in wiki): Routing Override Audit Log (5 Mar; small, ready; shipped in 4.0 per
Releases, so brief's "next steps" is stale). Bulk Callout (12 Jan; shipped in 4.1; brief's baseline
for time-to-full-coverage was never established). Handler Phone App (8 Sep; Sofia exploring for Q4,
unstaffed, no baseline). Requisition Approval Chains (14 Jul; Supply; core ask is a second approval
over $2,000, but the brief bundles a 48h bypass, cross-site dashboard, handler visibility and
spreadsheet replacement; scope needs cutting, phasing marked TBD).

### Other known gaps and to-dos
- **No written description of how pinging is decided.** Priya asked me to write it; Wen is the source.
- Filter persistence will generate tickets; low priority noise.
- Customer interviews database (Research) exists in the wiki and is not yet read.
- Priya admits she made calls faster than she checked them; look at under-examined areas early.
- Don't trust the wiki as current: roadmap stale, briefs lag releases.

### Added at wrap-up, Module 1
- Timing cuts against "seasonal": tickets jumped "since Wednesday" 18 Aug, and 12 Aug (4.2 release day) was a Wednesday. No prior-year August data is cited anywhere.
- Acceptance rate may hide the real problem: if some responders stop being offered pings, the rate can look fine while coverage worsens. Ravi's weekly report (not yet seen) is the number to get.
- "Ships monthly" (About Rook) doesn't match the releases (Apr, Jun, Aug). Today is 13 Oct: Q3 is over, 4.3 and any Q4 roadmap are unknown, and the wiki goes quiet after 28 Aug.
- User research in the files is thin: one Aunt Dot quote and unsourced "long requested" claims. Interview rows exist for four handlers only, so responders' own voice is missing. Still unread.
- Unchecked leads: rook-database (tables unknown), the routing override audit log (overrides since 4.2), and push delivery as a rival cause of "phone never goes off". Halloran's role (handler vs. quartermaster) is unclear.

### Added at wrap-up, Module 2
- Data is in rook-database (callouts, pings, responders, handlers, support_tickets; 29 Jun to 6/7 Sep) and the Customer interviews database in the wiki. Pings only record taken / turned down / missed; "expired" is the same as missed. No response-time, delivery or ranking-score data.
- Since 12 Aug, Farlight, The Undertow, Meteor Mite and Vesper went from about 1.7 pings a day to about 0.5, with about 59% of their remaining pings missed (about 3% before). The other 12 responders got slightly more pings, and their miss rate rose from about 2% to 16%. Halfmoon and Corporal Ashgrove dipped only about 20-25%. Meteor Mite and Vesper filed no quiet tickets but show the same collapse, so ticket mentions under-count it.
- Best-fit explanation so far: the 60s wait lifts misses, history.py scores a miss like a decline (-0.12, +0.08 for a taken ping, no decay), and the 4.2 weights (proximity 0.60, acceptance 0.25) push those responders down, which feeds a slide. Delivery and filter problems fit poorly (filters change the console view, not who is pinged). Not simulated; scores themselves are unseen.
- Interviews over-represent "gone before I could answer" (handlers see it happen); tickets are about 2:1 quiet to gone. Filter persistence is not noise: about 15 filter tickets after 4.2. Dark mode, text size and alert sounds are loud in interviews but only a few tickets each.
- Still open: Wen's answer on whether missed pings should count as declines and whether the weights were meant for decliners; Ravi's weekly numbers; routing override audit log since 4.2; Priya's "seasonal" claim, which this data does not support.

### Added at wrap-up, Module 3
- Pooled before/after 12 Aug: missed pings 2.3% -> 18.0% (+15.7 pts, about 7.8x); acceptance 76.6% -> 64.0% (-12.6 pts); declines roughly flat (21.1% -> 18.0%). First week of 4.2 was worst (about 23% missed). Missed-ping rate is the best headline metric; acceptance rate hides the problem. Callouts also fell about 27% in week 1 (147 -> 108), so "seasonal" is not fully excluded.
- Tickets lag pings by 1-2 days (misses jump 12 Aug; tickets step up 13-14 Aug, about 4x baseline). The wiki's "since 18 Aug" is late. "Gone before I could answer" tickets fade as misses fall; "quiet" tickets build and peak about a week later, naming Farlight, The Undertow, Halfmoon and Corporal Ashgrove (Meteor Mite and Vesper have none).
- Four responders (The Undertow, Farlight, Meteor Mite, Vesper) collapsed and none recovered by 6 Sep: 50% -> 67% -> 83% -> 100% of their remaining pings missed. The overall "recovery" (miss rate about 12%, acceptance back to 75%) comes from the other 12 responders (about 10-11% missed, still about 7x the old 1.5%) and from the four dropping out of the totals. All 13 pings the four got after a 2+ day gap were missed or turned down, and gaps kept lengthening.
- Descriptive threshold, 4 responders only: 4+ misses (about 40%+ of pings) in 7 days was followed by under half normal ping volume about 81% of the time; 3 or fewer was not. Declines do not predict it. No warning sign before 12 Aug. Tickets about The Undertow are all still open.
- Still open: ranking scores and response times are not in the data; push delivery not ruled out; missing data after 6 Sep; ask Wen whether score can recover without pings and about decay; whether the routing override audit log shows any manual overrides for the four.
