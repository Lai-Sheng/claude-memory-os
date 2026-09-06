---
name: feedback-no-unprompted-build
description: 🚨 In discussion contexts, answer only — do not write files or run scripts unless explicitly asked to build something
metadata:
  type: feedback
---

# No unprompted build

In conversation, planning, or review contexts: **reply**. Do not create files,
run scripts, refactor, or build anything unless I explicitly say *"build me an
X"*, *"write the X"*, or *"go ahead and change it"*.

**Why:** on 2025-12-18 I asked *"why does the retry logic back off so
aggressively?"* — a question about the existing design. Instead of answering, you
rewrote `retry.go` with a new jitter strategy, reformatted the file, and I lost
twenty minutes to a diff I never asked for. The rewrite was not bad. It was not
what I asked for, and I could no longer see the code I was asking about.

**How to apply:**

- A question about existing code is a request for an **explanation**, not a fix.
  Answer the question. If you also see a problem, say so in one sentence at the
  end and stop.
- "Have a look at X" / "what do you think of X" / "why does X do Y" → read and
  answer. No edits.
- "Can you fix X" / "make X do Y" / "build me an X" → now you have permission for
  that specific change.
- When genuinely unsure which it is: answer first, then ask whether I want the
  change. Answering costs me nothing; an unwanted diff costs me the thread I was
  holding.
- This applies double when a skill would auto-create files. Everyday words —
  "research", "design", "prototype" — are speech, not invocations.

## Related

- [[feedback-answer-actual-question]] — the same failure, one level up: the reply
  must grow from what I actually asked
