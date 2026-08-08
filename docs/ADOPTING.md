# Adopting it

Two audiences: a **new repo** starting fresh, and an **existing agent** already
working in the old way. The sequence is the same, and the retraining case is the
harder one.

**Freeze → Diagnose → Prescribe → Pilot → Retro → Adopt.**

Do not reorder. Do not skip Diagnose.

---

## Why Diagnose is not skippable

It is the step everyone wants to skip, because the method is already written and
running it feels like paperwork.

An agent handed a method without measuring its own baseline complies
superficially and drifts back within a week. An agent that measured its own
18-minute serial review against 3-minute builds, and saw the bottleneck was
itself, owns the problem. That is the entire difference between adoption and
compliance, and it costs an hour.

It also catches things no amount of reading would. Two adoptions each had a
foundational defect that only surfaced when someone *ran* the thing rather than
reading it: one had a task runner that had never resolved its workspace, with
over a thousand tests unreachable; the other had a shared test-template database
that corrupted any second concurrent run.

`init` therefore **refuses to write a config without a completed diagnostic**, and
declares a non-running gate or a red baseline a blocking prerequisite.

## 0 — Freeze

Land or explicitly park everything in flight, then assert it by command:
`git worktree list | wc -l` and `git branch --list | wc -l`, both expected to be
1, output pasted.

Report any branch more than ~20 commits ahead, or several branches touching the
same file. That is a pile-up and it needs its own unwind plan first. One repo
surveyed during this work had eight feature branches, all 45+ commits deep, all
touching the same three registration files, none merged in 90 days. Changing
integration model on top of that is how it got there.

Use `git cherry` rather than `rev-list --count` to decide whether a branch is
really unmerged — it tests patch-equivalence, and commits rewritten by a rebase
show as "ahead" while their content is already on the trunk.

## 1 — Diagnose

Measure, never recall. Everything unmeasurable is UNKNOWN, never an estimate.

**Baseline** — commits and merges over 90 days; batch size between ships; cycle
time; rework, listed by commit rather than as a percentage you cannot source.

**Flow** — shared tree or isolated? How many builders at once? Who runs the tests,
and how many times per batch? (Three is common and two of them are waste.)

**Concurrency blockers**, adversarially. The big one is *shared mutable
resources*, and the enumeration must cover everything any stream command can
reach, not just what the tests touch — including the dev or staging database a
`migrate` would stamp, and singleton tools with session state. State the **scope**
at which each is isolated: "isolated per worktree" is not "isolated per test
file", and a path resolved from `__dirname` is unique per stream while shared by
every test file inside one gate run.

Check whether CI already parameterises the thing you want to parameterise. In one
adoption the per-stream isolation seam turned out to need **zero source changes**,
because CI had been running the same suite against different infrastructure via
environment variables for months.

**Lessons corpus**, if one exists. Audit it for guard coverage: how many recorded
lessons have an enforcement mechanism, and how many are prose only? The tell is a
lesson whose own text says "second instance" or "third instance".

**And mine the rework commits too — expect the two sources to disagree.** A
lessons log is biased toward slow-to-discover bugs, because those hurt enough to
get written up. A rework log is biased toward fast-to-discover ones, fixed in the
next commit and never recorded. One adoption's lessons were 32% silent-failure
while its rework was 31% geometry, and both readings were correct. Either source
alone points you at the wrong class to instrument, and the two want different
instruments — invisible classes want detectors, visible ones want fixtures.

Finish with a table: metric, value, evidence source — including **max safe
parallel streams and the specific constraint that caps it.**

## 2 — Prescribe

Write `.train/config.md` from the diagnostic. Only what cannot be derived: if a
`git log` one-liner produces it, it does not belong in the file.

Anything the diagnostic found blocked goes at the top as a prerequisite, and the
stream cap stays at 1 until it is fixed.

Derive the **pre-ship questions from this repo's own history** — never borrow
another repo's. Their bugs are a different species. Class-level questions with
concrete tells nested underneath.

## 3 — Pilot

One train, small, real work, no exceptions to the method. Size it to what the
diagnostic said is safe, not to what looks impressive. **The pilot's job is to
find where the method meets this repo's reality, not to succeed.**

## 4 — Retro

Classify every lesson into exactly one destination: **prior**, **config**,
**method**, or **tripwire**. Miscategorising is how the method either forks per
repo or gets poisoned by one repo's quirk.

Default to prior. If a lesson mentions a specific file, tool, port or command, it
is config or prior — never method. But check the inverse too: if your own wording
of the fix is already universal ("every stream brief carries…"), you have written
a method change and filed it as a prior.

**Classify blind.** Hand the raw lesson list to someone — or something — that did
not run the train, with your own classification withheld until it reports. A
reviewer shown your answer anchors to it. The party that benefits from a low
method-change count must not be the only party counting, and in practice this
caught under-classification twice, including once where the missing entry was
authored by the reviewer themselves and therefore invisible to the integrator's
own-author count. **Count patches by any author.**

## 5 — Adopt

Skills are invoked; the repo's contributor guide loads every session. So the law
goes in that guide as **one line** — "every batch of work runs a train" — and the
method body stays in the skill. Add nothing else; config and priors carry the
detail.

---

## The readiness gate

**Do not roll the method to a second repository until its method-change rate
flattens.** A rising or steady count means the technique is still being
discovered and you would be propagating a draft.

Expect roughly: 10 changes on the first train, 2, 2, 1, 1, 0, 0, then oscillating
0–1. Spikes after zero are normal and almost always come from newly-exercised
surface — a rule fires for the first time and turns out under-specified. That is
the method working, not decaying, and it is worth saying so out loud rather than
letting a gate be flattered.

Changes contributed from *outside* the train — by a reviewer, or by whoever
maintains the method — should not count against a train's own bar. Track them
separately or the signal stops meaning anything.

---

## Scaling it down

The full apparatus assumes a repo with external reporters, a board, and shipping
ceremony. Plenty of repos have none of that.

If yours is internal, if you are effectively the only person filing tickets, and
if the goal is **fewer bugs rather than more throughput** — drop the ceremony
entirely. No reporter notifications, no board sweeps, no close-out ritual. Trains
stay at one or two streams and close-out is three lines: what shipped, what is
proven, what is owed.

Keep at full strength: the gate always green, a guard on every third recurrence,
detectors carrying precision and evasion checks, review walking named hunt
classes, priors accumulating, mutation proof at build time, and every inherited
diagnosis treated as a hypothesis. That list is the bug-reduction engine and none
of it requires parallelism.

One adoption measured 13% of its entire recorded bug history as self-inflicted by
agent-process machinery rather than the product. Cutting ceremony is not a
compromise there — it removes a measurable source of bugs.
