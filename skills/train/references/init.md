# train init - onboard or retrain a repo

Run when a repo has no `.train/config.md`, or when its substrate has moved
(new CI, new test harness, new deploy). This is also the **retraining** path for
an agent already working a live project in the old way.

## The sequence

**Freeze → Diagnose → Prescribe → Pilot → Retro → Adopt.** Do not reorder and
do not skip Diagnose.

Diagnose is load-bearing and will be the tempting one to skip. An agent handed a
method without measuring its own baseline complies superficially and drifts back
within a week; an agent that measured its own bottleneck owns it. **Refuse to
write a config without a completed diagnostic.**

## 0. Freeze

```
git status --porcelain; git branch --list; git worktree list
git log --all --not --remotes --oneline | head # unpushed work
```

Land or explicitly park everything in flight. Report any branch that is >20
commits ahead, or where several branches touch the same file - that is a
pile-up and it needs its own unwind plan before any train runs. Changing
integration model on top of in-flight work is how eight-branch conflicts form.

## 1. Diagnose

The answer protocol in SKILL.md applies to every answer. Anything unmeasurable
is UNKNOWN, never an estimate. Show the command with each figure.

**Baseline** - commits and merges over 90d; batch size (files, lines) between
ships; cycle time from first commit to shipped; rework (commits fixing something
shipped in the previous three batches, listed by sha, not as a percentage).

**Flow** - shared tree or isolated? Branch per task or straight to trunk?
How many builders run at once? Who runs the tests - builder, reviewer, or
integrator, and how many times per batch? What is the review/QA gate, what does
it run, how long does it take?

**Where the wall clock goes** - for the last three batches, split into building,
local test runs, CI, waiting on the operator, and rework. Name the longest pole
with its number.

**Concurrency blockers - be adversarial, assume parallel work fails:**

1. *Shared mutable resources - enumerate them all at once.* Not just what the
  tests touch: every fixed, mutable thing ANY stream command can reach. Test
  databases and ports, template/seed DBs, the shared dev or staging database a
  `migrate` would stamp, cache dirs, fixed file paths, named containers,
  singleton tools with session state (browser, REPL, tmux pane). Produce ONE
  list and mark each entry per-stream-isolated or integrator-owned - **and
  state the SCOPE at which each is isolated.** "Isolated per worktree" is not
  "isolated per test file": a path resolved from `__dirname` is genuinely
  unique per stream while still being shared by every test file inside one
  gate run, so a drop-and-recreate in one file destroys another's fixtures
  mid-run and surfaces as flakiness rather than collision. Check across
  streams AND within a single gate run; the right answer at the wrong scope
  reads as a clean bill of health. Scoping
  this question to "the tests" is how three separate shared-resource
  collisions get discovered as three separate incidents instead of one
  enumeration. Grep the test config and harness for hardcoded ports, DB
  names, and any drop-and-recreate guarded by process-local state. **This is
  usually the real ceiling, not the git tree.** Check whether CI already
  parameterises it - if CI runs the same suite against different infrastructure
  via env vars, that seam is your per-stream isolation and it is free.
2. *CI triggers and runner count.* What refs trigger a run? Is the concurrency
  group per-ref (queues) or global (cancels)? How many runners? Five parallel
  branches on one runner is a queue, not parallelism.
3. *Sequence-numbered artifacts.* Migrations, ordered fixtures, numbered configs
  - anything where two branches picking a number collide. What catches it and
  what does recovery cost?
4. *Files nearly every batch touches.* From §1 of SKILL.md. Separate authored
  from generated - generated files must never be merged, only regenerated.
5. *Order- or network-dependent checks* unsafe to run concurrently.
6. *Does the gate actually RUN?* Execute it. Do not read it off package.json,
  a Makefile or CI config and assume. A gate can be written and never have
  worked - exiting non-zero on a workspace-resolution error, or exiting ZERO
  having executed nothing, while a suite of real tests sits unreachable from
  the build graph. Record the baseline pass/fail counts from the actual run.
  **A gate that does not run, or a red baseline, is a blocking prerequisite** -
  every later claim in this method rests on "the stream proved itself with the
  gate", and that sentence is worthless if the gate is decorative. Same lesson
  as rehearsing a migration by executing it: reading proves intent, not
  behaviour.

**Lessons corpus, if one exists** - a postmortem log, bug-lessons table or
incident wiki. Audit it for GUARD COVERAGE: how many recorded lessons have an
enforcement mechanism, and how many are prose only? A repo can have a strong
recording culture and no enforcing culture, and the tell is a lesson whose own
text says "second/third instance". Recording without enforcing is why the same
class recurs; the rule that fixes it is that **a lesson is not closed until it
has a guard, or a written reason why no guard is possible** - the latter is a
legitimate answer and must be stated, not left blank.

Mine the rework commits as well, and expect the two sources to DISAGREE. A
lessons log is biased toward slow-to-discover bugs (they hurt, so they get
written up); a rework log is biased toward fast-to-discover ones (fixed in the
next commit, never recorded). Either source alone will point you at the wrong
class to instrument, and the two want different instruments - the invisible
classes want detectors, the visible ones want fixtures.

**Seams** - from the co-change data, name the natural independent slices and the
collision hotspots no scheme avoids. Of the last 10 pieces of work, how many
could genuinely have run in parallel without touching the same files? Name them.

**Own failure modes** - where did unverified claims get stated, where did the
documented process get skipped, and what in the repo's own guide is routinely
not done.

Finish with a table: metric, value, evidence source - cycle time, batch size,
CI wall and queue time, rework, and **max safe parallel streams with the
specific constraint that caps it.**

## 2. Prescribe

Write `.train/config.md` from the diagnostic. **Only what cannot be derived** -
if a `git log` one-liner produces it, it does not go in the file.

```markdown
# .train/config.md - substrate facts for <repo>
Diagnosed <date>. Re-run `/train init` when CI, test harness or deploy changes.

## Gate - EXECUTED during diagnosis, never read off package.json/Makefile
exact-command-here # the one the stream runs; must match CI's invocation
lint: ...
baseline: <pass/fail counts, pasted from the actual run>
Do NOT also run: <redundant suites already covered by the gate>

## Test isolation
mechanism: <per-stream ports | namespaced DBs | none needed | BLOCKED>
handles: <the env vars or ports allocated per stream>
max concurrent streams: <n> # and the constraint that caps it

## CI
triggers: <refs>
runners: <n, hosted or self-hosted>
stream branches push to origin: <yes/no> # no, unless runners are elastic

## Sequence-numbered artifacts
<migrations dir, allocation rule, what catches a collision>

## Integrator-owned - never merged; regenerated or restamped once on the merged tree
<generated paths + the command that regenerates them>
<mechanical stamps every stream would otherwise touch for reasons unrelated to
  its change: version counters, build numbers, changelog headers. A stamp bumped
  per-commit by convention looks like the repo's worst hotspot and is not one.>

## Hotspots requiring co-queueing
<top files by commit share, with the share; refresh at §1 each train>

## Pre-ship questions
<the repo's own checklist the Stream Report must answer, DERIVED from its own
  lessons and rework history - never borrowed from another repo, whose bugs are
  a different species. Shape it as class-level QUESTIONS with concrete tells
  nested underneath, not a flat list of specifics: a flat list accumulates past
  twenty items and stops being read, while a question with tells under it stays
  one page. Two repos arrived at this shape independently.>

## Deploy
<exact invocation, gates, timing constraints>

## Operator-owned - hard stop, never auto-change
<config the operator owns>
```

Anything the diagnostic found BLOCKED goes at the top as a prerequisite, and
the cap stays at 1 stream until it is fixed.

## 3. Pilot

One small train, 2 streams, real work, no exceptions to the method. Report §8
honestly including what went badly. The pilot's job is to find where the method
meets this repo's reality, not to succeed.

## 4. Retro

`/train retro` on the pilot. Its output seeds `.train/priors.md`.

## 5. Adopt

Skills are invoked; the repo guide loads every session - so **the law goes in
the repo's CLAUDE.md as one line** ("every batch of work runs `/train`"), and
the method body stays in the skill. Add nothing else to CLAUDE.md; the config
and priors carry the detail.

Seed `.train/priors.md` from the shared store's cross-repo notes before the
first real train, so a new agent starts at the ecosystem's current competence
rather than at zero.
