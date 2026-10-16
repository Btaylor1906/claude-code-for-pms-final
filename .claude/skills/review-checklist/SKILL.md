---
name: review-checklist
description: Reviews a product brief against a fixed four-point checklist (owner, success measure, scope consistency, problem before solution). Use when the user asks to review a brief, run the review checklist, or check a brief before it goes further. Point it at a brief file or pasted text.
---

# Review checklist

Run the same four checks on any brief, every time. Read the whole brief first, then judge each check.

## The four checks

1. **Owner named.** The brief names a specific person who owns it (not a team, not "TBD", not a role with no name). If an owner appears only in a different document, say so.
2. **Success measure.** The brief says how we will know it worked: a metric or observable outcome, plus a baseline or starting point and a target or threshold. A metric with no baseline is a partial pass.
3. **Scope matches start to end.** Compare the scope described at the start (problem, goals, summary) with the scope at the end (requirements, phasing, next steps). Flag anything that appears in one and not the other, and anything added or dropped along the way.
4. **Problem before solution.** The problem is explained, with who has it and the evidence, before any fix is proposed. Flag solutions that appear in the opening, or a problem section that only restates the solution.

## Output

Give each check a verdict: **Pass**, **Partial** or **Fail**. For each, add one or two lines of evidence quoting or pointing to the part of the brief that decided it, and say what is missing when it isn't a Pass.

Format:

```
Brief: <title / file>

1. Owner named: <verdict>: <evidence>
2. Success measure: <verdict>: <evidence>
3. Scope start vs end: <verdict>: <evidence>
4. Problem before solution: <verdict>: <evidence>

Ready to go further? <Yes / Not yet>: <the one to three things to fix first>
```

## Rules

- Judge only what the brief says. Don't fill gaps from other files or from what you think the author meant; note a gap instead.
- Don't rewrite the brief or suggest new features. Only report against the four checks.
- Keep it short enough to read in under a minute.
