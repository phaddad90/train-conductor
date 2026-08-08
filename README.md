# Train

A method for running coding agents in parallel on one repository without them
colliding, and for making their claims checkable.

It is packaged as a Claude Code skill (`skills/train/`), but the method is
tool-agnostic — most of it is discipline, not automation.

---

## The problem it solves

Running several coding agents on one codebase fails in three ways, in this
order:

1. **They collide.** Two agents editing one working tree produce commingled
   commits, reset stomps, and lost work. Worktrees fix that in an afternoon —
   and then you discover the real ceiling is *shared test substrate*: one
   template database, one fixed port, one browser session, one hoisted
   `node_modules`. In one adoption, two concurrent test runs against a shared
   template DB produced 37 unexplained failures that read as flakiness.

2. **They assert things that aren't true.** An agent states a cause it never
   measured, a count it recalled rather than derived, or an absolute quantifier
   from a twenty-line sample. Subagent output gets laundered into confident
   prose. You end up being the verification layer, which does not scale.

3. **Nothing compounds.** Lessons get recorded and not enforced. In one
   adoption, a repository had 47 written lessons and almost no guards — one of
   them literally said *"third-and-counting instance"* of a class the repo had
   already documented twice.

Train addresses all three: worktree isolation and a substrate audit for the
first, an evidence protocol for the second, and a retro loop that converts
lessons into enforcement for the third.

---

## Is this for you?

**Good fit**

- One person (or a small team) directing several coding agents on a repo.
- You already have a test suite and a way to ship.
- Work arrives in batches — a handful of fixes and features at a time.
- You have been burned by an agent confidently reporting something wrong.

**Poor fit**

- A large human team with an existing merge queue and code-review culture. You
  have most of this already, in a form tuned to humans.
- A repo with no tests and no gate. Train's foundation is *"the stream proved
  itself with the gate"* — with no gate, that sentence is worthless. Build one
  first; `init` will refuse to proceed without it.
- One-off scripts and throwaway prototypes. The ceremony costs more than it
  returns below roughly three items of work.

**Partial fit, and worth knowing about**

Much of the value is available at **N=1**. A single stream with a predicted file
set, a gate that actually runs, a named-hunt-class review, a guard for every
third recurrence, and an evidence ledger *is* the quality engine. Parallelism is
a separate axis. If your goal is fewer bugs rather than more throughput, adopt
the discipline and ignore the trains.

---

## What a train is

A batch of work, decomposed into **streams**. Each stream:

- runs in its own git worktree on its own branch,
- verifies its base commit before touching anything,
- builds one predictable slice,
- proves its own new tests can fail, then runs the full gate itself,
- returns a ~40-line report, not a diff.

An **integrator** decomposes, dispatches, integrates and ships. It never builds
and never writes prose that could be a template. It merges green streams onto a
train branch, regenerates anything generated, runs the gate **once** on the
merged result, then ships once.

Review runs **per stream, concurrently**, and is adversarial only — it never
re-runs a suite the stream already ran green.

The train ends with a retro that classifies every lesson into exactly one
destination, which is what stops the method forking per repo.

---

## The three layers

Mixing these is how the method rots.

| Layer | Lives in | Changes |
|---|---|---|
| **Method** | the skill | Rarely. Identical in every repo. Divergence is debt. |
| **Substrate** | `.train/config.md` | When the repo's infrastructure moves. Only facts that *cannot be derived* — everything derivable is re-measured every train, because a stored fact is recall and recall goes stale silently. |
| **Priors** | shared store + `.train/priors.md` | Every retro. Accumulated failure classes and calibration. |

A new agent on a repo inherits the method, reads the config, and starts at the
priors' current competence rather than at zero. That is the compounding
mechanism, and it is the whole point.

---

## Using it

```
/train init     # onboard or retrain a repo — measures its substrate first
/train          # run a train
/train retro    # fold the finished train's lessons back
```

`init` is not optional and its diagnostic step is not skippable. An agent handed
a method without measuring its own baseline complies superficially and drifts
back within a week; an agent that measured its own bottleneck owns it.

Full adoption sequence: [docs/ADOPTING.md](docs/ADOPTING.md).
Vocabulary and rationale: [docs/CONCEPTS.md](docs/CONCEPTS.md).
The failure shapes it guards against: [docs/PATTERNS.md](docs/PATTERNS.md).

---

## Honest expectations

From roughly fourteen trains across two repositories:

- **Your first train will produce ~10 changes to the method itself.** By the
  sixth it should produce zero to one. That decline is the readiness signal —
  do not roll the method to a second repository until it flattens, or you are
  propagating a draft. Spikes after zero are normal and almost always come from
  newly-exercised surface: a rule fires for the first time and turns out to be
  under-specified.

- **Sharply-briefed streams are much faster than intuition predicts.** Same
  codebase, same model: 37–75 minutes unbriefed against 3–13 minutes with a
  goal, acceptance evidence and a predicted file set. The predicted file set is
  what bounds exploration.

- **The bottleneck migrates, and that is the shape of the work.** Shared tree →
  fix with worktrees → review becomes the longest pole → parallelise it →
  *the orchestrator* becomes the constraint, because it is reading reports and
  writing briefs. Each fix is real and each exposes the next. Budget for it
  rather than reading it as the previous fix having failed.

- **The orchestrator's context, not your infrastructure, is the first ceiling.**
  Capping stream reports and widening trains buys more than adding hardware.

- **Two metrics matter and they measure different things.** *Brief gaps* count
  what a brief failed to name (omission). *Capacity burned* counts what it named
  wrongly (commission). Watching only the first will read as solved while the
  second runs unmeasured. Green-first-time tracks work difficulty, not process
  health — do not chase it upward.

---

## What it is not

It is not a merge queue, and it does not replace CI. On a single self-hosted
runner, opening a pull request per stream *creates* a queue where none existed —
one adoption measured five parallel branches as ~34 minutes of wait for the
fifth. Streams prove themselves locally; the integrated result is what CI sees.
If your runners are elastic, that constraint disappears and PRs are affordable
again. This is exactly the kind of fact the substrate layer exists to hold.

It also has nothing to say about work whose *shape* is not yet known. Train
executes; it does not chart. If the question cannot be stated precisely yet, no
stream should be dispatched to find out — that is a planning problem and wants a
planning method.
