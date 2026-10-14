# 04 · X-Ray Vision — prompts

**Context:** You joined Rook two weeks ago as PM on Dispatch.
Release 4.2 shipped on 12 August, just before you arrived, and
landed badly. Last session the numbers showed you what happened: a
ping used to wait ninety seconds and now it waits sixty, people
missed pings they used to catch, and missing one counts the same as
turning one down — so four responders stopped hearing from us
altogether.

At the end of the session, type wrap up, and Claude Code saves the
prompts you wrote yourself below — not the starter prompt — updates
CLAUDE.md, and saves your work to GitHub. By Module 6 this file is a
prompt library built from your own questions.

---

### 1.

Open the folder 00-rook/code/dispatch-routing/. This is the part of our software that decides who gets asked to take a job. I have never read code before and I am not going to start now. Walk me through what happens from the moment something goes wrong somewhere to the moment a responder's phone buzzes, in plain English, no jargon. Then tell me which file each step lives in.

### 2.

Using the scoring logic in routing.py and history.py, simulate a responder who starts with a 0.5 acceptance score and then misses 1, 2, 5, and 10 consecutive pings. How much does their overall ranking score fall under the pre-4.2 weights versus the 4.2 weights? Show the calculation step-by-step.

### 3.

Trace the code path for a responder who has accumulated several misses. What mechanisms, if any, allow their acceptance score to recover without first receiving and accepting additional pings? Identify every place in the code where the score can increase, decrease, reset, or decay over time.

### 4.

Using the current routing algorithm, estimate how many additional declines or misses by higher-ranked responders would be required before a low-ranked responder such as The Undertow would receive a ping. Compare this before and after the 4.2 weighting change.

### 5.

answer this question: Based on what I found, the reason some responders are getting no pings at all is ___, because ___.

### 6.

Quick one, did the change to who gets asked first apply to responders who'd already been turning jobs down, or just new ones? Can't tell from the code and don't want to guess on this. Anyone know?

### 7.

Find me the part of this code that takes points off somebody when they miss a ping or turn one down. Show it to me and explain it in plain English. Then find me every single thing in this code that puts points back on.

### 8.

Somebody has been quiet for a month. Walk me through, step by step, exactly what would have to happen for them to start getting work again.

### 9.

answer this question: According to my findings in the code, for someone who's gone quiet, they would need to ___.
