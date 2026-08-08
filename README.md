# Train

![A steam locomotive pulling three carriages, drawn as a phosphor-green engineering blueprint on black](assets/social-preview.png)

A method for running coding agents in parallel on one repository without them
colliding, and for making what they tell you checkable.

It ships as a Claude Code skill (`skills/train/`), but most of it is discipline
rather than automation. The rules below hold whatever agent runtime you use.

---

## The metaphor, if it helps

The name is not decoration. The whole method maps onto a train, and most people
find it easier to hold that way.

| Rail | Here |
|---|---|
| **The train** | One batch of work, planned, run and shipped as a single unit. |
| **The conductor** | The lead agent. Plans the manifest, dispatches, integrates, ships. Never drives a carriage itself. |
| **The manifest** | The plan. What is aboard, which carriage carries it, in what order, and what each is allowed to touch. |
| **The tickets** | The work items. Each is a passenger with a seat reserved before departure, not one who wanders on later. |
| **The carriages** | The subagents. One per ticket, each on its own track, coupled only at the end. |
| **The track** | The isolated branch and worktree a carriage runs on until integration. |
| **The gate** | The inspection every carriage passes on its own before it is allowed to couple. |

The load-bearing part of the metaphor is that carriages couple **once, at the
end**, and the train leaves as a unit. Nothing ships carriage by carriage, and
nothing couples that has not passed inspection alone. When a carriage fails
inspection it is uncoupled and left behind rather than dragged along because it
is already built.

---

## Contents

- [Why this exists](#why-this-exists)
- [How this compares to Devin and similar platforms](#how-this-compares-to-devin-and-similar-platforms)
- [The problem it solves](#the-problem-it-solves)
- [Is this for you?](#is-this-for-you)
- [What your repo needs first](#what-your-repo-needs-first)
- [Installing](#installing)
- [The three commands](#the-three-commands)
- [What a train actually looks like](#what-a-train-actually-looks-like)
- [The three layers](#the-three-layers)
- [Files it creates](#files-it-creates)
- [Honest expectations](#honest-expectations)
- [Objections you will have](#objections-you-will-have)
- [Common questions](#common-questions)
- [What it is not](#what-it-is-not)
- [Further reading](#further-reading)

---

## Why this exists

It started with a wrong hypothesis, and the correction is the interesting part.

One operator, several codebases, and coding agents that were each individually
capable of good work. The constraint was never how good any single agent was. It
was that running more than one of them produced collisions, and that checking
what they reported back consumed exactly the attention that was supposed to be
freed up.

**The original hypothesis was simple: if agents can run in parallel without
colliding, throughput goes up.** That turned out to be the easy half. Git
worktrees solved collisions in an afternoon, the shared test database took a day
after that, and then the throughput gain arrived roughly as expected.

**What did not go away was the operator.** With several agents running, the
bottleneck moved to reading what came back and deciding whether to believe it.
An agent would state a cause it had never measured. A count would turn out to be
recalled rather than derived. A subagent's finding would get repeated by its
parent with the uncertainty stripped out, land in a document, and be discovered
two days later. None of that is a concurrency problem, and no amount of
isolation touches it.

So the method grew a second half nobody planned: **an evidence protocol.** Every
number carries the command that produced it. Absolutes require a population-wide
check. A figure not re-derived in-session is treated as recall. Any claim the
conclusion leans on gets a deliberate attempt to falsify it. That half turned out
to matter more than the parallelism.

Then a third problem surfaced. Lessons were being written down and never
enforced, so the same class recurred. One repository held 47 recorded lessons
with almost no guards, and one of those lessons described itself as the third
instance. That produced the retro loop and the rule that a lesson is not closed
until it has a guard or a written reason none is possible.

The measured outcome after fourteen batches across two codebases: throughput held,
rework attributable to a previous batch sat at zero every time it was measured,
and there were no reverts. **The velocity gain was real but modest. The
correctness gain was large, and the operator stopped being the verification
layer** - which was the actual constraint all along, just not the one anyone set
out to fix.

---

## How this compares to Devin and similar platforms

Worth being straight about, because the shape is genuinely similar.

From public descriptions, Cognition shipped parallel Devin sessions in early
2026 and then Managed Devins, where one session acts as a coordinator, breaks
work into pieces, delegates to child sessions each running in its own isolated
VM, monitors them, resolves conflicts and compiles the results into pull
requests. That is the same topology described here: a conductor, carriages,
isolated tracks, integration at the end. Anyone claiming to have invented that
shape is not paying attention. ([overview](https://www.marktechpost.com/2026/06/10/ai-coding-agents-development-platforms-2026/),
[orchestration](https://aidevsetup.com/insider/devin-agents-can-now-orchestrate-other-devins-what-it-means))

**Where the platforms are straightforwardly better.** A full VM per child is
stronger isolation than a git worktree, and it removes the shared-substrate
problem by brute force rather than by auditing for it. They are managed, so
there is no setup and no substrate diagnostic to run. They handle browser use,
team features, session replay and support. They scale past what one machine and
one operator's attention can hold.

**Where this is different, and it is one axis, not many.** These are platforms;
this is a method. It is a markdown file you drop into a skills directory, it runs
on the setup you already have, and you own it. More importantly, the layer it
adds is verification rather than orchestration:

- A carriage must prove its own new tests **can fail** before the suite runs, by
  reverting its fix and showing the test go red. Unfalsifiable tests were the
  single largest defect class measured here, and running your suite in a cleaner
  VM does not make a test that cannot fail able to fail.
- Every claim in a report carries the command that produced it, and absolutes
  require a population-wide check. Better isolation does not stop an agent
  asserting an unmeasured cause.
- A finding that recurs three times becomes an enforcement mechanism, and a
  lesson is not closed until it has one. Accumulated agent memory is not the same
  thing as accumulated enforcement.

**The honest summary is that these compose rather than compete.** The failure
modes instrumented here are agent-behaviour problems, not infrastructure
problems, and nothing about them depends on where the agent executes. If you are
already running a platform that gives you isolated agents and pull requests, take
the verification half and leave the orchestration half alone - it is the part
you are missing, and the part no sandbox provides.

One caveat on all of the above: this comparison is drawn from public
descriptions rather than from having run these platforms side by side against
the same backlog. Nobody has done that measurement, including me.

---

## The problem it solves

Running several coding agents on one codebase fails in three ways, and they
arrive in this order.

**First, they collide.** Two agents editing one working tree produce commingled
commits, reset stomps, branch-switch propagation and lost work. Git worktrees fix
that in an afternoon. Then you discover the real ceiling is shared *test*
substrate: one template database, one fixed port, one browser session, one
hoisted `node_modules`. In one adoption, two concurrent test runs against a
shared template database produced 37 unexplained failures that read as flakiness
rather than as a collision, because the harness dropped and rebuilt the template
under a process-local guard.

**Second, they assert things that are not true.** An agent states a cause it
never measured, a count it recalled rather than derived, or an absolute
quantifier drawn from a twenty-line sample. Subagent output gets laundered into
confident prose and lands in a document. You end up being the verification layer
yourself, which does not scale past one repo and one attention span.

**Third, nothing compounds.** Lessons get recorded and never enforced. One
repository audited during this work held 47 written lessons and almost no
guards. One of those lessons said, in its own text, "third-and-counting
instance" of a class the repo had already documented twice.

Train addresses all three. Worktree isolation plus a substrate audit for the
first. An evidence protocol for the second. A retro loop that converts lessons
into enforcement for the third.

---

## Is this for you?

**Good fit**

- One person, or a small team, directing several coding agents on a repo.
- You already have a test suite and a way to ship.
- Work arrives in batches: a handful of fixes and features at a time.
- You have been burned by an agent confidently reporting something wrong.

**Poor fit**

- A large human team with an existing merge queue and code-review culture. You
  already have most of this, in a form tuned to humans rather than agents.
- A repo with no tests and no gate. The foundation of the method is the sentence
  "the stream proved itself with the gate", and with no gate that sentence is
  worthless. Build one first. `init` will refuse to proceed without it.
- One-off scripts and throwaway prototypes. Below roughly three items of work the
  ceremony costs more than it returns.

**Partial fit, and worth knowing about**

A lot of the value is available at N=1. A single stream with a predicted file
set, a gate that actually runs, a review that names what it is hunting for, a
guard for every third recurrence and an evidence ledger *is* the quality engine.
Parallelism is a separate axis entirely. If your goal is fewer bugs rather than
more throughput, take the discipline and ignore the trains. There is a section
on exactly that in [docs/ADOPTING.md](docs/ADOPTING.md#scaling-it-down).

---

## What your repo needs first

Four things, and `init` checks all of them before it will write a config.

1. **A gate that runs.** One command that lints, type-checks and tests. It must
  be *executed* during onboarding, never read off a `package.json` or Makefile
  and assumed. A gate can be written and never have worked: one adoption found
  a task runner that had been exiting non-zero on a workspace-resolution error
  since the day it was added, with 1,116 real tests sitting unreachable from the
  build graph. Underneath that was a second defect where the runner stripped
  environment variables, so a test-isolation seam that had been proved by hand
  would silently not have worked under the gate.

2. **A green baseline.** A red gate on arrival trains everyone to re-run until
  green and drowns every real signal after it.

3. **Test isolation, or a plan for it.** Each concurrent stream needs its own
  database, port, or namespace. Check whether your CI already parameterises
  this. In one adoption the seam turned out to need zero source changes, because
  CI had been running the same suite against different infrastructure via
  environment variables for months.

4. **A clean tree.** No uncommitted work, no in-flight branches. Changing
  integration model on top of in-flight work is how repos end up with eight
  long-lived branches nobody can merge.

---

## Installing

Copy the skill into your Claude Code skills directory:

```bash
# user level, available in every repo
cp -r skills/train ~/.claude/skills/

# or project level, for one repo only
cp -r skills/train /path/to/repo/.claude/skills/
```

User level is recommended and deliberate. One copy of the method means it cannot
quietly fork per repository, which is the failure this design is most concerned
with. Everything repo-specific lives in that repo's `.train/` directory instead.

---

## The three commands

```
/train init onboard or retrain a repo, measuring its substrate first
/train run a train
/train retro fold the finished train's lessons back
```

**`/train init`** runs Freeze, Diagnose and Prescribe, then stops. It produces a
measured baseline and a `.train/config.md`. Its diagnostic step is not
skippable, and this is the most important thing in the whole method: an agent
handed a method without measuring its own baseline complies superficially and
drifts back within a week. An agent that measured its own eighteen-minute serial
review against three-minute builds, and saw that the bottleneck was itself, owns
the problem. That difference costs about an hour.

**`/train`** runs one train: measure, decompose, dispatch, verify, integrate,
ship, report.

**`/train retro`** classifies every lesson from the train into exactly one
destination, appends the priors, and reports the method's own change rate.

---

## What a train actually looks like

Concretely, from a manifest of six items.

**Measure.** Re-derive the hotspots from `git log` rather than reading a stored
list. Confirm the tree is clean and nothing is in flight. Read the config for
what cannot be derived: the gate command, isolation mechanism, migration system,
deploy invocation.

**Decompose.** Write a manifest: per item, the predicted file set, which hotspots
it claims, its migration slot, its dependencies. Two items touching the same
hotspot go in the *same* train, dispatched one after the other. Anything adding
an enforcement mechanism goes first and the rest branch off its merge. Anything
whose file set is unknowable becomes a read-only investigation instead. Then walk
the manifest yourself before dispatching: does every predicted file exist, does
every acceptance criterion name both the mutation and the failure it must
produce, is every figure a median of at least five runs.

**Dispatch.** Each stream gets its own worktree, a branch off the current train
head, a declared base sha it must assert before touching anything, its own
database port, and the lines from priors relevant to its risk surface. Every
input you will need from a human is requested now, in one batch, never
discovered mid-build.

**Stream exit.** The stream reverts each of its own fixes in turn, pastes its new
tests going red, and restores. Then it runs the full gate once, in its worktree,
against its own database. It returns a report capped at about forty lines: shas,
file list, gate output pasted, the pre-ship questions answered, the live evidence
that will prove it after deploy, anything it could not verify, and its claim
ledger. The bulk evidence goes to a gitignored file in the worktree.

**Review.** One reviewer per stream, spawned when that stream finishes, running
while others are still building. It does not re-run the green suite. It walks the
classes it declared it would hunt, replays the mutation proofs, checks that tests
fail for the reason claimed rather than merely failing, and rehearses any data
migration by executing it against a scratch database.

**Integrate.** Fast-forward each green stream onto the train branch in dependency
order. Never hand-resolve a source conflict; the losing stream rebases and
re-runs its own gate. Regenerate anything generated. Run the full gate once on
the merged result, because green-alone proves nothing about the train. Hunt
pairwise semantic conflicts explicitly. If the integrated gate goes red with no
obvious culprit, bisect by dropping streams rather than guessing.

**Ship and tear down.** One push, one CI run, one deploy. Remove every worktree,
delete every branch, and assert the counts by command with the output pasted.
Then report, then retro.

---

## The three layers

Mixing these is how the method rots.

| Layer | Lives in | Changes |
|---|---|---|
| **Method** | the skill | Rarely. Identical in every repo. Divergence between repos is debt, not diversity. |
| **Substrate** | `.train/config.md` | When the repo's infrastructure moves. Only facts that cannot be derived, because a stored fact is recall and recall goes stale silently. |
| **Priors** | shared store plus `.train/priors.md` | Every retro. Accumulated failure classes and calibration data. |

A new agent on a repo inherits the method, reads the config, and starts at the
priors' current competence rather than at zero. That is the compounding
mechanism and it is the entire point of keeping the layers apart.

---

## Files it creates

```
.train/
  config.md substrate facts, written by init, re-read every train
  priors.md failure classes and calibration, appended by every retro
```

Both are committed. Neither is large: a config runs to a couple of hundred lines
and a priors file grows by a handful of lines per train.

Your contributor guide gets exactly one line added, saying that every batch of
work runs a train. The method body stays in the skill so the guide does not
bloat.

---

## Honest expectations

Drawn from roughly fourteen trains across two repositories.

**Your first train will produce about ten changes to the method itself.** By the
sixth it should produce zero or one. That decline is the readiness signal, and it
is the gate for rolling the method to a second repository: do it before the rate
flattens and you are propagating a draft. Spikes back to one after a zero are
normal and nearly always come from newly-exercised surface, where a rule fires
for the first time and turns out to be under-specified. That is the method
working rather than decaying, and it is worth saying so out loud rather than
letting the number be flattered.

**Sharply-briefed streams are much faster than intuition predicts.** Same
codebase, same model tier: 37 to 75 minutes unbriefed, against 3 to 13 minutes
with a goal, acceptance evidence and a predicted file set. The predicted file set
is doing most of that work, because it bounds exploration.

**The bottleneck migrates, and that is the shape of the work.** Shared tree, so
you fix it with worktrees. Then review becomes the longest pole, so you
parallelise it. Then the orchestrator becomes the constraint, because it is
reading reports and writing briefs. Each fix is real and each exposes the next
one. Budget for the migration rather than reading it as the previous fix having
failed.

**The orchestrator's context, not your infrastructure, is the first ceiling.**
Capping stream reports and widening trains buys more than adding hardware does.

**Two metrics matter and they measure different things.** Brief gaps count what a
brief failed to name, which is omission. Capacity burned counts what it named
wrongly, which is commission. Watching only the first will read as solved while
the second runs unmeasured, and that happened here for three consecutive trains.
Green-first-time tracks work difficulty rather than process health, so do not
chase it upward.

**Rework across trains should sit at zero.** If it does not, the gate is the
problem, not the parallelism.

### What it costs

The intuition is that this burns far more tokens than working normally. One
repository's actual accounting says otherwise, and the shape of the answer is
more interesting than the size.

Mean tokens per active day, conductor and carriages combined, comparing 52
active days before adoption against 10 after:

| | Before | After | |
|---|---|---|---|
| Output tokens | 1,678,576 | 1,452,048 | **down 13%** |
| ...of which the conductor | 1,376,005 | 587,953 | down 57% |
| ...of which the carriages | 302,571 | 863,860 | up 186% |
| Cache writes | 15,561,044 | 30,876,627 | **up 98%** |
| Weighted total (indicative) | - | - | **up ~23%** |

Generation got **cheaper**, not more expensive. A carriage with a predicted file
set does not read the whole repository to work out where to start, and that
exploration was the largest single line of the old bill. The conductor's own
spend more than halved because it stopped building and stopped authoring.

The overhead is real but it is somewhere unintuitive: **context priming.** Cache
writes roughly doubled, because N carriages each need their own context
established rather than one long-running agent amortising a single context all
day. That is what pushes the weighted total up by roughly a quarter, and it is
the honest cost of the method.

So: budget around 20 to 25% more, not 2x or 3x. And weigh it against what the
same accounting shows on the other side - rework attributable to a previous
train measured at zero every time, and zero reverts across the whole period.
Rework is pure re-spend, and one repo's pre-adoption baseline had roughly a
third of all commits classified as fixes.

**Caveats, because this is one measurement and not a study.** Ten post-adoption
days against fifty-two before. One repository. The price weighting is indicative
rather than a real invoice. The work mix changed at the same time, so this is
not a controlled comparison. Measure your own before believing any of it.

---

## Objections you will have

**"This is a lot of process for an agent."** Most of it costs seconds and runs
once. The manifest walk is three commands. The mutation proof is a revert and a
restore in a worktree you already have. The expensive parts of the method are the
parts that replaced something more expensive: a review that replays proofs is
cheaper than one that discovers them.

**"My agent already writes tests."** So did these. One train measured tests that
could not fail as its dominant defect class: nine or more instances across seven
streams. A test written alongside its fix and never proven red is decoration that
reads as coverage. That single change, moving mutation proof to build time, is
probably the highest value-per-adoption-cost item in the method and it needs no
worktrees, no parallelism and no orchestrator.

**"Why not just open a pull request per stream?"** Because it depends entirely on
your runners, which is why that fact lives in the substrate layer rather than the
method. On a single self-hosted runner, five parallel branches meant roughly 34
minutes of queue for the fifth, so PRs created a bottleneck where none existed.
On elastic hosted runners the constraint disappears and PRs are affordable again.
Measure yours.

**"We are too small for this."** Then run it at one or two streams and drop the
ceremony. The bug-reduction half of the method is independent of the parallelism
half. One repo measured 13% of its entire recorded bug history as self-inflicted
by agent-process machinery rather than by the product, so cutting ceremony there
removed a measurable source of bugs rather than being a compromise.

**"Our agent is reliable, we do not need an evidence protocol."** Two figures in
one repo's first diagnostic were wrong when independently checked, and that
diagnostic was otherwise excellent work. The protocol is not an accusation; it is
what makes good work checkable, and the same agent later caught three of its own
false findings before reporting them.

---

## What it is not

**It is not a merge queue and it does not replace CI.** Streams prove themselves
locally and the integrated result is what CI sees. If you already run a merge
queue with speculative batching, you have solved the integration half and should
take the verification half only.

**It has nothing to say about work whose shape is not yet known.** Train
executes; it does not chart. If a question cannot be stated precisely yet, no
stream should be dispatched to find out. That is a planning problem and it wants
a planning method, used before this one.

**It is not a substitute for knowing your own codebase.** Every genuinely useful
thing the method did in adoption came from measurement of that specific repo:
which files are hotspots and why, what the test substrate actually shares, what
its own bug history says its failure classes are. The method supplies the
questions. Your repo supplies every answer, and borrowing another repo's answers
is the one shortcut guaranteed not to work.

---

## Common questions

**How do I run multiple Claude Code agents in parallel on the same repository?**
Give each one its own git worktree and branch, never the shared checkout, and
have each verify its base commit before editing. That removes file collisions.
Then find what they still share: a test database, a fixed port, a browser
session, a hoisted `node_modules`. That shared substrate, not git, is the real
ceiling on how many can run at once.

**Why do my AI agents keep conflicting in git?**
Because they are editing one working tree. Two agents in one checkout produce
commingled commits, reset stomps and branch-switch propagation. Worktrees fix it
structurally in an afternoon; discipline alone does not, and leaks.

**How many coding agents can I run at once?**
The lower of two numbers: what your test substrate can isolate, and what your
orchestrating agent can actually read. The second usually binds first, because N
agents times a report is a context budget.

**How do I stop an AI agent from claiming something it did not verify?**
Require every number to carry the command that produced it and the output
containing it, ban absolute quantifiers without a population-wide command, and
treat any figure not re-derived in-session as recall rather than measurement. A
hedge on a countable quantity ("4,600+", "roughly") is the reliable tell.

**Why do agent-written tests pass when the code is broken?**
Because a test written alongside its fix is rarely proven able to fail. One
measurement found unfalsifiable tests to be the single largest defect class:
nine or more instances across seven parallel agents in one batch. The fix is for
the agent to revert its own change and show the test going red before the suite
runs.

**Is this a merge queue?**
No. A merge queue serialises integration on your CI. This batches work, isolates
each piece, verifies each alone, then integrates and ships once. If you already
run a merge queue you have the integration half and should take the verification
half only.

**Does it only work with Claude Code?**
It ships as a Claude Code skill, and that is the fastest way to adopt it. The
method itself is a set of rules about isolation, verification and evidence, and
nothing in it depends on a particular agent runtime.

**What does it cost in tokens?**
Roughly 20 to 25% more on a weighted basis in the one repository that was
measured, and the shape is counterintuitive: output generation fell 13% because
briefed agents stop exploring, while cache writes doubled because each agent
primes its own context. See [What it costs](#what-it-costs).

---

## Further reading

- [docs/CONCEPTS.md](docs/CONCEPTS.md) - the vocabulary, and the failure behind
  each rule.
- [docs/ADOPTING.md](docs/ADOPTING.md) - the full onboarding sequence, the
  readiness gate, and how to scale the method down.
- [docs/PATTERNS.md](docs/PATTERNS.md) - the recurring failure shapes it guards
  against, each with the evidence that produced it.
