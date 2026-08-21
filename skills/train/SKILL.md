---
name: train
description: Run a batch of work as a merge train - N streams building concurrently in isolated worktrees, each self-verifying to green, integrated once and shipped once. Use for ANY multi-item batch of work in ANY repo, in place of building serially in the shared tree. Modes - "init" to onboard or retrain a repo (measures its substrate first), "retro" to fold a finished train's lessons back into the method. Triggers - "run a train", "train init", "train retro", "parallel streams", "batch this work".
---

# Train

One method, every repo. Three layers, never mixed:

- **METHOD** - this file. Universal. Identical everywhere. Resist per-repo
  variation; divergence between repos is debt, not diversity.
- **SUBSTRATE** - `.train/config.md` in the repo. ONLY facts that cannot be
  derived. Everything derivable is re-measured at §1 of every train, because a
  stored fact is recall and recall goes stale silently.
- **PRIORS** - the Second Brain first, `.train/priors.md` only for the
  genuinely repo-local. Read before dispatch. Appended by retro.

**A rule has exactly one authoritative home, and correcting it means enumerating
its READERS.** The layers above say where a rule LIVES; this says what happens
when it changes. Every document that restates a rule is a reader - a checklist
that executes it, an agent definition loaded at start-up, a reference doc - and a
correction that updates the nearest copy leaves the rest live. Two copies of one
rule do not diverge loudly. They diverge silently, and the divergence stays
invisible until someone follows the copy that is wrong.

## Modes

| Invocation | Does |
|---|---|
| `/train` | Run a train. Requires `.train/config.md`. |
| `/train init` | Onboard or retrain a repo. Read `references/init.md`. |
| `/train retro` | Fold the last train's lessons back. Read `references/retro.md`. |

**No `.train/config.md`? Stop and run init.** Never guess the substrate.

**One repo, one remit.** A train never spans repositories, and an agent is never
tasked with another repo's init, config, priors or backlog - each repo's agent
owns its own. The method is shared; the work never is. The Second Brain is the
only channel between repos, and it carries lessons, never instructions.

## Answer protocol - governs every output below

Applies to the manifest, Stream Reports, QA verdicts, train reports, and
anything said to the operator.

- **A1 Ledger.** Every number and factual assertion carries the exact command
  that produced it and the literal output containing it. Cannot paste output
  containing the figure? Label it UNVERIFIED. Ledger at the end, not inline.
- **A2 Scope.** State the window measured and the window claimed. Absolute
  quantifiers - never, always, zero, none, every, "returns nothing" - require a
  population-wide command. Measuring 20 commits and claiming 90 days is the
  canonical failure. **And a population-wide command must prove it SAW the
  population**: state the denominator, show it is non-empty, and name the AXIS it
  covers. A zero, a green or an "all paths" from a scan whose scope is unstated
  is UNVERIFIED. An enumeration bounded to one direction of a state transition
  has not enumerated the transition - if you fixed every path that ADVANCES, say
  so, and say that the paths that reverse or cancel were not examined.
- **A3 Recall.** Any figure not re-derived this session is recall. A hedge on a
  countable quantity - "4,600+", "~", "over", "roughly" - is the tell.
  Countable things get counted. **A3 governs what is SAID; this governs what is
  COMMITTED.** A countable figure written into a source file, docstring or
  comment must be recomputed by the build, or it does not go in - state the
  RELATION the tests enforce and point at the test. A published CONSTANT is
  exempt, because a standard's own threshold cannot drift, and that exemption is
  the boundary rather than a loophole.
- **A4 Falsify.** Take the three claims the conclusion leans on hardest and try
  to prove each wrong. Say what was tried.
- **A5** Subagent claims inherit zero trust; run A1-A4 on them yourself.
- **A6** Assume every number will be re-derived downstream.
- **A7 Account.** A response to an enumerated instruction answers every item BY
  NAME - including no-ops, already-done items and items deliberately dropped -
  and states the commit it is true at. A crossing revision is recognised by
  checking that commit's parent, not by re-reading the instruction. An item that
  appears in no line of the answer is an unanswered item, not an implied yes.
- **A8 Live figures.** Against a running system a baseline is re-measured
  immediately before the change, never carried from build time - a figure that
  was true four hours ago is recall. A figure that moves against prediction is
  attributed by ORIGIN, by joining records to the change's own marker, never by
  the size of the delta. A fix is verified through the CONSUMING surface's own
  predicate, never a raw field that looks equivalent.

**Refer to work by name, not by bare id.** "#150, #151, #128" is a wall the
operator has to look up one at a time; "#150 (never-reject violation)" reads at
a glance. The id rides inside the name, never stands in for it.

Absence-of-token claims about source must exclude comments or parse structure -
a grep for a token inside a file that documents that token matches its own
comment.

## 1. Measure - every train, never recalled

```
git log --since="90 days ago" --no-merges --name-only --pretty=format: \
  | grep -v '^$' | sort | uniq -c | sort -rn | head -20     # hotspots
git log --since="90 days ago" --oneline --no-merges | wc -l  # denominator
git branch --list; git worktree list                         # in-flight state
git status --porcelain                                       # dirty tree
```

Hotspot share = count / denominator. Anything above ~5% of commits is a
hotspot. Read `.train/config.md` for what cannot be derived: gate command,
test-isolation mechanism, migration system, deploy invocation, generated
artifacts, CI triggers and runner count.

A derived index - a code graph, symbol index, embedding store, generated map -
is a stored fact under another name: recall goes stale silently, so it is
ASSERTED fresh, never read fresh. Assert against the newest thing the repo
changed (`git log -1 --name-only`), never against the tool's own status line,
and treat a miss as staleness until proven otherwise - a stale index degrades
into a plausible capability limit ("unsupported file type"), which is exactly
why it survives. `.train/config.md` names the index and its refresh command.
(method_ext, operator-approved 2026-08-17.)

**Refuse to start if the tree is dirty or work is in flight.** Land or park it.
Changing integration model with work in flight is how eight-branch pile-ups
happen.

**Enter a train with headroom for a whole train.** The integrator's own
context is an entry condition budgeted here exactly as §3 budgets spawns:
compact at the boundary, where every input to the manifest is durable and
re-readable, so the compaction costs nothing. Where a train is large enough
that mid-train compaction becomes unavoidable, that is expected - and is
precisely why verdicts, the sha mapping, dispatch rationale and operator
rulings are written to files as they are produced rather than held in
context (§5). (method_ext, operator-approved 2026-08-17.)

## 2. Decompose - the step most often skipped, and where rework comes from

Produce the train manifest before any dispatch: per item, the predicted file
set, hotspots claimed, migration slot, dependencies, and its **blast radius** -
whether being wrong here costs money, data, access, or an irreversible external
effect. Most items are cheap to get wrong; the few that are not should be
visible at manifest time, not discovered at review.

- **Below about three items, it is not a train.** One urgent fix runs solo (§7);
  two items in a shared tree cost less than the manifest, the worktrees and the
  integration. The ceremony has to be earning something.
- **What makes a correct slice is not its size.** A slice is right when its
  predicted file set is disjoint from its siblings AND its acceptance evidence is
  self-contained - provable without waiting for another stream to land. Size
  (~30 min) is a consequence of getting those two right, not the criterion. A
  slice that needs a sibling's work to demonstrate itself is one stream, not two.
- **Predict the file set from co-change history, not from the ticket.** The
  ticket says what to change; §1's ranking says what historically gets changed
  with it. A prediction drawn from reading the ticket alone systematically misses
  the test file, the fixture, the registry entry and the doc that always move
  together, and those misses are what spend the +2 allowance.
- **Dispatch order is not manifest order.** Enforcement mechanisms go first
  (they bind everyone). Then longest-pole first, then descending, because the
  train's wall clock is its slowest chain and starting that last idles it.
  High-blast-radius items go early too, so a failure surfaces while there is
  still time to react rather than at integration. The concurrency cap is the
  LOWER of the substrate limit and what the integrator can actually read: N
  streams times a report is a context budget, and that ceiling usually arrives
  first.
- **Dropping a stream mid-train is normal.** If an item turns out wrong, pull it
  from the merge, let teardown delete its branch, and return it to the backlog.
  Do not carry a doubtful stream into integration because it is already built.

- **Hotspots co-queue, they do not exclude.** Two items touching the same
  hotspot belong in the SAME train, dispatched sequentially inside it - the
  second branches off the train head after the first merges. Pushing the second
  to a later train adds delay and buys no safety; same-scope changes need to be
  tested *together*, which is where semantic conflicts surface.
- **Semantic dependency = one stream.** If B's correctness depends on A's
  behaviour, they are one stream run sequentially, never two. Two streams green
  alone can be wrong together.
- **Sequenced resources couple like hotspots.** A single-head migration chain,
  an ordered registry, any monotonic sequence - two claimants cannot verify
  their link into it from inside their own worktrees, because the sibling's
  entry does not exist there. Two claimants on one sequence co-queue: a declared
  dispatch order at manifest time, the second based on the first, and that order
  IS the integration order - declared up front, not discovered at integration.
  (operator-approved.)
- **A stream that adds an enforcement mechanism constrains every sibling.** A
  ratchet, tripwire, lint rule, schema constraint or ACL retroactively binds all
  other work in the same train - one-to-many, not pairwise, so the dependency
  rule above does not catch it. Such a stream is dispatched FIRST with siblings
  branching off its merge, or held to the next train. Never concurrent with the
  work it will constrain: no sibling's brief can carry a requirement that does
  not exist yet at dispatch. **The concurrency exemption is judged on PRACTICE,
  not file overlap** (observed across two consecutive trains, operator-approved): a ratchet policing a practice
  - how tests assert, how errors render, how connections resolve - touches every
  sibling with zero file overlap, so file-set disjointness never exempts it.
  One train invoked the file-overlap reading and paid three same-train recurrences of
  the exact class the ratchet banned. **Its brief must also enumerate the recognition
  axes** - what object, spelled how, named or set which ways - and the house
  idioms per axis, gathered by grepping before dispatch. Skip this and QA
  discovers the shapes one at a time: observed as a 7-round ladder costing ~2.5h
  against a 14.9-minute build, 6 rounds of which were greppable upfront. **And
  it must try to evade its own detector** - a guard keyed on a name rather than
  a binding is bypassed by shadowing. This is §5's mutation-check applied to a
  detector, where the mutation is a bypass shape, not a new rule. **And it must
  report its PRECISION on the current codebase** - every flag it currently
  raises, triaged true or false positive. Evading itself tests false negatives;
  this tests false positives, and both are acceptance criteria. A detector that
  fires 5 times and is wrong 5 times gets ignored within a week, after which the
  class it guards is silently unenforced while still looking enforced - worse
  than no detector. If it flags by a hand-maintained list, ask what the
  vocabulary IS: one that evolves inside this repo (schema columns, route names,
  registry entries) must be DERIVED from its source of truth, because a hand
  copy of it is the mechanism's own decay. One that is external and fixed (a
  language's builtins, another engine's function set) has no in-repo source to
  derive from and a hand list is correct - but write down which case you are in
  and why, or the next reader cannot tell a justified list from a rotting one.
- **An investigation is not a build stream.** The test is whether you can state
  the question precisely NOW - not whether you can answer it. A sharp question
  with an unpredictable file set is an investigation; a question you cannot even
  phrase sharply is not ready to dispatch at all, and saying so is the honest
  answer rather than dispatching a stream to find out. Its file set is unknowable
  at dispatch, so §3's predicted-set hard stop and the hotspot rule cannot
  protect it. Dispatch it to produce a FINDING plus a proposed brief - read-only,
  no fix - and the fix becomes a properly predicted stream in this train or the
  next. If it must fix in flight, it declares its file set UNKNOWN and may not
  run concurrent with anything touching plausible surface. **A manifest cannot
  claim "no hotspot collisions predicted" while any
  stream's file set is unknown** - that is an unfounded absolute, not a
  prediction. **A ticket's diagnosis is a hypothesis, not a spec.** Any stream
  whose brief inherits a diagnosis - from a ticket, a prior finding, or the
  integrator - verifies that premise against evidence as its first analytical
  act, before building on it. A4 covers the stream's own conclusions; this
  covers what it was handed. Observed: a "box config" diagnosis was wrong
  (nothing had ever seeded the config anywhere), and the correct fix differed;
  a brief once named an edit site that was dead code on the live path.
  **So is the integrator's own prescription.** When the integrator directs a
  MECHANISM change rather than a defect fix - replace this parser, use that
  library's own output, derive it from the source of truth - the brief must
  require the stream to enumerate what the OLD mechanism covered and prove the
  new one still covers it. A stream complies exactly and loses coverage doing so,
  and it cannot easily refuse a direction that arrives with a rationale and a
  bound. Observed: replacing a hand-rolled manifest parser with the build tool's
  metadata output was sound reasoning and a net -55 lines, and silently stopped
  scanning the workspace root manifest, losing coverage in two layers at once.
   **AND THE INTEGRATOR'S OWN FIGURES CARRY THE LEDGER.** The two rules above bind
   the STREAM: verify what you were handed. They work - which is exactly why every
   stream in one train corrected the ticket that dispatched it, four of them
   correcting facts the integrator had supplied as established. That is the method
   catching the defect at the last possible moment, at the cost of every stream
   doing correction work that should never have been needed. So the obligation runs
   upstream too: **every figure in a manifest or brief is either re-derived this
   session with its command pasted, or labelled RECALL**, and a stream may refuse a
   brief whose numbers are unlabelled. The population ledger binds what a stream may
   CLAIM; nothing bound what the integrator may ASSERT, and that asymmetry pointed
   the discipline away from the party with the most reach. When every stream in a
   train corrects its own dispatching ticket, that is a decomposition defect, not a
   spec-quality problem.
- **Decisions serialise; execution parallelises.** A decision changes what the
  remaining work IS, so two decisions resolved in one pass means the second was
  made against a stale picture - the same one-to-many shape as the ratchet rule.
  Anything whose output is a decision rather than a diff runs one at a time, and
  where that decision is the operator's it is never resolved by inference, no
  matter how blocked the train is. An agent answering on the operator's behalf
  because waiting is expensive has broken the thing the rule protects.
- **Slice to ~30 min of build.** Sharply-briefed streams run far faster than
  intuition predicts (observed 3-13 min against 37-75 min unbriefed); the
  predicted file set is what bounds exploration. Widen the train, not the slice.
- **Walk the manifest before dispatching anything.** QA walks streams and §8
  enumerates integrator SIDE-EFFECTS; nothing otherwise walks integrator
  JUDGEMENT, and judgement errors propagate with full authority because streams
  comply exactly. Three mechanical checks, costing minutes:
  1. **Every predicted file exists**, or is explicitly marked to-be-created.
     `git ls-tree -r HEAD --name-only | grep -f predicted` takes a second.
     Observed: a stream dispatched against an asset directory that was not in the
     repo at all - it arrived with a later phase's handoff - and halted correctly
     having cost a full dispatch.
  2. **Every acceptance criterion names the mutation AND the failure it must
     produce.** A criterion that cannot be falsified is not a criterion. Observed:
     "all six tests must go red" was impossible for one of the six, and a
     compliant stream would have contorted a correct test to satisfy it.
  3. **Every figure is a median of >=5 runs with min and max, or is labelled
     UNVERIFIED.** A single sample may raise a question; it may never settle one.
     Observed: one timing sample taken on a machine that was compiling was 7x
     wrong and nearly bought an unnecessary optimisation programme. The method
     manufactures that contention itself by running N streams on one box.
- **A tracker's status is not evidence the work is undone.** Before an item
  becomes a stream, verify it is still open against the SYSTEM, not against its
  status field - the defect still reproduces, the row still exists, the path is
  still reachable. An item that cannot be shown still open is an investigation at
  most, and possibly a close. The premise check further down fires INSIDE a
  stream, which is after the stream is paid for.
- Show the operator the manifest with its ledger. Do not wait for approval
  unless a hotspot claim is contested or an item needs an operator input.

## 3. Dispatch - every stream gets all of this or it does not launch

- **Agent spawns are a consumable session resource - budget them.** A train
  costs roughly 2N+ spawns (a carriage and a walker per stream, plus ratchet,
  classifier and content roles), and runtimes cap spawns per SESSION
  cumulatively, not concurrently. Reaping idle agents does not help: the counter
  counts spawns, never live agents. `.train/config.md` records the cap and the
  observed per-train cost; when a session's remaining budget is under two trains'
  worth, start a fresh session rather than degrading mid-train.
  **Prefer resuming a finished agent over spawning a fresh one** for repeat roles
  - a walker already matched to a surface has performed as well or better than a
  new spawn. Never force a context-exhausted agent; retire it and start one.
  **Resuming spends the agent's context instead of the spawn odometer** - the two
  are currencies for the same purchase, and a resume economy that works perfectly
  hides the exhaustion until it fails all at once. Six consecutive trains ran on
  zero fresh spawns and then walked into a wall with no warning.
  **The runtime cap is a runaway backstop, never the budget.** Set it high enough
  that it cannot bind, and do the budgeting in config where it is visible and
  measured. A ceiling you are steering by is a ceiling that will surprise you.
- **THE POPULATION LEDGER - declare the slots BEFORE the work.** Folded after the train that measured it
  (operator, 2026-08-20) after E2 failed six times in one train on the class it
  was written for. You cannot detect a missing check by inspecting a report,
  because the omission removes the claim and its evidence together - so the slot
  must exist before the report does. The brief enumerates every population claim
  this stream will have to make:

  | trigger | slot |
  |---|---|
  | the stream ADDS A CHECK | denominator + axis |
  | the stream CENSUSES or claims completeness | one slot per search route, **minimum two of DIFFERENT KIND** |
  | the stream WRITES EXISTING ROWS | dry-run query + its output |
  | the stream returns a NULL RESULT | the population that WOULD have contained the thing |
  | the stream asserts any ABSOLUTE | denominator + axis |

  **"Different KIND" is load-bearing and the brief must say so in these words.**
  Two greps against the same index is ONE route wearing two hats, and that is
  exactly how that missed file was missed. The census that worked used three
  genuinely different kinds - ORM attribute, raw SQL string, and
  instance-reads-by-holder - and only the third found the file no prior list
  named.

  **The brief must also state, in the brief itself, that UNKNOWN is blameless.**
  Not in the method, not in a prior - in the text the stream reads. A mechanism
  that punishes empty slots without offering an honest unknown manufactures
  evidence, and the first stream that cannot reach live will invent a number
  rather than write that it could not determine one.

  EMISSION is required everywhere. **IDENTITY is preferred where the stream
  controls both sides** - make the claim and the check the same statement, as a
  migration whose dry-run and write share one population constant does, or a rule
  with one implementation and several callers rather than a restatement in a
  second language. Identity cannot diverge; emission can only be checked.
- Its own git worktree and a local branch off the current train head. Never the
  shared tree. Never a push to origin unless config says otherwise.
- **Verifying the base is the stream's FIRST act, before any edit.** Worktree
  tooling spawns at a main-line sha, not necessarily the branch you declared -
  observed failing 3 of 3 times. The brief states the expected base sha; the
  stream asserts it, ff/resets if wrong, and says so in its report. Without
  this, co-queueing silently builds the second claimant on the wrong file state
  and every "clean merge" is luck.
- **A base that was AMENDED or rewritten - not merely advanced - rebases with
  `rebase --onto <new-base> <old-base>`.** A plain rebase replays the superseded
  base commit itself into the stream. When the integrator rewrites a base, the
  grant states BOTH shas and the stream asserts the old one is its ancestor
  before rebasing. (operator-approved.)
- Brief: goal, acceptance evidence required, predicted file set, migration slot
  or "none", isolated substrate handles (test DB port/namespace per config, and
  any singleton resource - browser, REPL, shared session, shared dev or staging
  database - budgeted exactly like a port), model tier, the relevant lines
  from `.train/priors.md`, and **a statement that a SUBSTANTIATED NULL RESULT -
  already fixed, premise false, defect unreachable - is an acceptable
  deliverable.** A stream that manufactures a fix to fill its slot has done worse
  than nothing, because the fix will be walked, merged and believed.
- **Every operator input - screenshot, ruling, credential, sign-off - is
  requested at manifest time in one batched list.** Never discovered mid-build.
- Hard stops; report and halt rather than work around: config-owned data the
  operator owns, a production write, a side-effectful external retry, the same
  test failing three times, the file set exceeding predicted + 2, **running out
  of working context** - stop honestly with a clean tree and an explicit list of
  what remains (observed: two streams did exactly this and a fresh agent completed
  each first try; a stream that pushes on with spent context ships unverifiable
  work), or **needing a new dependency** - installed packages are near-always shared state (a symlinked
  virtualenv, a hoisted node_modules, a vendor dir), so one stream's install
  silently changes what every sibling's gate is testing against. Nothing fails;
  the results just stop meaning what they say.

## 4. Stream exit - the gate is mechanical, runs ONCE, by the stream

Run the repo's exact gate command from `.train/config.md`, in the worktree,
against its own test substrate. Do not run a second overlapping suite behind it.

**Mutation proof runs LEFT, at build time, by the builder** (observed across two consecutive trains,
operator-approved): before the gate, the stream reverts each of its fixes in
turn and pastes its own new tests going red, then restores. QA replays rather
than discovers; a stream whose tests were never proven red ships tests that
cannot fail, which one train measured as the dominant defect class (9+ instances,
7 streams). A mutation must vary exactly ONE thing - changing two validates
the test instead of the code (a stream's false proof). A red that fails for the
wrong reason (syntax error, import break) is not a proof; report failed
mutation attempts as what they were.

The Stream Report is what QA and the integrator read instead of the diff. **Cap
it at ~40 lines**: branch and sha, file list with line counts, gate commands
with pass/fail counts pasted, the repo's pre-ship questions answered with
evidence, the live evidence that will prove it after deploy, **what any embedded tunable
constant selects against live data at the value being shipped** - a threshold,
cutoff, rate limit, retry count or page size is a judgement call encoded as a
number, and nobody can review a number without seeing what it picks; paste the
read-only result so the decision is operator-visible BEFORE it ships, not
inferred from the diff - anything
UNVERIFIED, any file outside the predicted set, **every population-ledger slot
the brief declared, each filled FILLED / UNKNOWN / N-A** - FILLED means the
literal command and its literal output, never a summary of the output; UNKNOWN
means could-not-determine with the reason, and is blameless; N-A means it does
not apply, with the reason - and the claim ledger. Full
evidence goes to `STREAM-REPORT.md` in the worktree, **at a path the repo
gitignores**, with the path cited. Report artifacts never ride a merge - two
streams writing the same tracked path is an add/add conflict at integration.
Untracked-by-convention is not enough; `git add -A` defeats it, so the ignore
line is the enforcement and `.train/config.md` names the path.

**An investigation or design stream writes its finding to the FILE first and
returns a summary second - the reverse of a build stream.** A build stream's work
survives in its commit whatever happens to the agent. An investigation stream has
no commit: the finding IS the return message, so an idled session loses the
entire result. The durable artifact must never be the ephemeral one - the same
reasoning as the population ledger and the QA verdict file, and the same rule as
§7's detector clause: the thing that proves the work has to survive independently
of the party reporting it.

**Every stream report also answers "what did you notice that the brief did not
decide?" - answered, or explicitly NONE.** Same three fills as the population
ledger. The ledger forces a stream to emit its EVIDENCE so a claim cannot outrun
it; this forces a stream to emit its UNASKED QUESTIONS so a brief's blind spot
surfaces while the author is still in the code. It is the one question a stream
can always answer and a reviewer usually cannot.

## 5. QA - adversarial only, concurrent, one instance per stream

**QA GATES EVERY PUSH. NO STREAM SHIPS UNWALKED, EVER.** Operator law, restated
2026-08-20 after it lapsed for two consecutive trains. It is not a per-train
judgement and does not lapse because a batch is small, urgent, bug-only or
obviously fine. **If you find yourself reasoning that a particular stream does
not need a walk, that reasoning is the defect.**

**The gate is not a substitute.** The repo-level rule this mirrors used to read
"run a QA pass (or the full suite)", and that parenthetical is how the law
lapsed: the suite ran, so the rule read as satisfied while nothing independent
looked at the diff. A gate proves the code does what its tests say. QA asks
whether the tests say the right thing - which is the only question that catches
a stream shipping exactly to a brief that was wrong.

**QA WRITES ITS OWN VERDICT FILE. A verdict that exists only as an agent's
return message does not exist.** One file per stream, on disk, at a path the
train names before dispatch. This is the same rule as the durable-artifact rule
for investigation streams, and it exists because two entire trains' QA verdicts
were lost: they had only ever been chat turns.

**A train cannot close until the per-stream verdict count is asserted BY COMMAND
and its output pasted**, exactly as section 7 requires of worktree and branch
counts. UNKNOWN is an allowed and visible value; a missing file is not. Without
this, QA's absence is undetectable - which is precisely how it went unnoticed
across two trains, two close-outs and two retros.

- **Spawned by that stream's completion**, not batched by the integrator. QA
  for a finished stream runs while other streams are still building. Each QA
  instance holds its own test substrate handle.
  **Walkers dominate the spawn bill** - typically 9-11 of a ~15-spawn train. When
  budget is tight, one walker may take several streams SEQUENTIALLY: independence
  survives, because a walker is independent of the builders whichever streams it
  takes, and only concurrency is lost. That is usually cheap, since walks run
  under build time. Declare it when you do it, and retire the walker before its
  context degrades - a walker on its fourth stream is not the walker that did the
  first. Never consolidate a walker across a stream it built.
- Does NOT re-run a suite the stream already ran green. That is duplicated
  compute and a collision risk.
- **The QA brief NAMES its hunt classes**, derived at brief-generation time from
  `.train/priors.md` plus the stream's own risk surface - the priors file is the
  standing class list, so seeding from it is mechanical, not judgement. A defect
  found in round 2 whose class was NOT named is a brief gap, not a QA win: that
  is what makes the §8 split falsifiable instead of self-flattering. A
  first-occurrence exemption - "no prior existed to seed from" - holds only if
  the class was underivable from the stream's RISK SURFACE too, since hunt
  classes come from both sources; absent-from-priors alone is not enough, or
  every gap becomes a first occurrence. One exemption per class, and the class
  enters priors the same day. A hunt list
  that names everything is not a hunt list - padding it to avoid gaps is itself
  a finding.
- **QA is held to the same evidence standard as the stream: a check that was
  attempted but did not take effect is not a check.** Three specifics follow
  from that one rule, and every one of them was learned the hard way -

  - *Walk the named classes* in the diff. Declaring them beforehand is what
    makes "QA caught it" falsifiable after.
  - *Mutation-check* that new tests fail without the fix, scoped to the tests
    the stream added. For a guard written after an incident, the mutation is
    **RESTORING THE DEFECT IT WAS WRITTEN FOR, in the shape that defect actually
    had** - never a synthetic restatement, never by breaking a helper. To prove a
    guard is INVOKED, delete the CALL: breaking the helper proves only that the
    helper is reachable from somewhere. The guard must fail INDEPENDENTLY for
    every member of the population it claims to cover, restoring clean between
    each - one red proves one member, never the set. And assert the LOCATION of
    an effect, not only its occurrence: a test proving something was written
    passes when the write lands in the wrong place. **Verify the mutation LANDED
    before its result counts**: re-read the value, re-import the module, check the diff. A probe
    that silently no-ops (an edit landing in a docstring instead of a table
    value, a patch applied to a copy) reads as "fix confirmed" and once nearly
    closed a real finding. **Commit before mutating** - reverting with a
    working-tree checkout against uncommitted changes destroys the FIX rather
    than the mutation, observed twice in one sitting.
  - *Mutation proves a test CAN fail. It does not prove the test fails for the
    REASON CLAIMED*, and that gap has let two defects through. Assert the
    **expected failure**, not merely a failure. Two mechanical halves:
    - **Proof of arrival.** A test must demonstrate it reached the code it
      claims to exercise. If it can pass without the function under test being
      entered, it is testing something else. Observed: a test failed under
      mutation because a validation layer IN FRONT of the target refused the
      input, so the target was never entered - it was exercising the guard in
      front of the guard.
    - **Failure specificity.** Assert the reason, never a substring that a
      generic or default message also contains. Observed: a permanent guard kept
      passing because a different error message happened to contain the asserted
      substring, so the guard survived the thing it documented being removed.
    - The same applies one level up to any test justified as "this proves we
      chose X over Y": ask what it would report if Y were implemented. If the
      answer is "the same", it never entered the discriminating region and is
      decoration. Observed: 2 of 3 tests passed against the rejected
      implementation until one asserted it had entered that region.
  - *Rehearse data migrations by EXECUTING them* against a scratch database
    built at the pre-merge head. A read-only SELECT proves which rows would be
    selected, never that the migration runs - driver type-binding, transaction
    semantics and the migration's own code are only exercised by running it.
    Production-shaped data where a backup restore is available; an empty schema
    proves syntax, not behaviour. A corrective sweep's cutoff derives from the
    DEFECT'S WRITE WINDOW - the last moment before the fix's first possible
    write - never from the deploy or run moment, which re-destroys post-fix
    truth on any re-run. Rehearsal executes the migration TWICE: a sweep that
    is not idempotent against post-fix rows is not finished. (operator-approved,
    operator-approved.)

  Then: run A4 against the stream's ledger, and hunt the absence cases.
- For claims of the class that keeps failing (absolutes), prefer 2-3 verifiers
  each given a *different slice* of context, with disagreement as the trigger,
  over one verifier with everything.
- Verdict is PASS or a specific failure with file:line. Fixes are re-walked as
  a delta - never a "smalls" round that lands unwalked. **The verdict is
  written to `.train/t<N>-verdicts.log` the moment it is produced** (gitignored;
  one line: stream, sha, PASS/FAIL, walker) - a verdict that exists only in
  conversation is destroyed by compaction and by any session boundary, and the
  loss is silent because integration proceeds on a remembered PASS.
  (method_ext, operator-approved 2026-08-17.)
  **The same holds for EVERY artifact handed between agents** - a patch, a
  finding, a census, not only a verdict. It is written to a file at the moment it
  is produced; conversation is not storage, and neither is a scratch directory
  the session owns. The reason was never specific to verdicts, and a verified
  patch was lost exactly this way.
- **When the integrator carries a departed stream's work itself, it declares
  that** exactly like an inline walk, and proves the transplant by CONTENT
  EQUALITY of the diff at source and destination - never by inspection.
- If a QA walk exceeds its stream's build time, say so in the train report: the
  walk is doing work the gate should own.
- **At-cap protocol, and it must be declared.** Out of spawn budget: first resume
  a finished walker for a bounded delta it already owns; only as a last resort
  does the integrator walk inline. An inline walk is DEGRADED review - the same
  party that dispatched the work is now judging it, which is the independence the
  method exists to preserve - so the report names every stream walked inline
  rather than letting it count as a proper walk. A silently degraded walk is
  worse than a skipped one, because it still reads as reviewed.

## 6. Integrate - the integrator's only build-adjacent job

- **An EMPTY population-ledger slot blocks the merge.** The check is mechanical -
  for each slot the brief declared, does the report carry one of the three fills?
  That is a lookup, not attention, and it is the whole conversion: "did you
  verify" is unanswerable from absence, "is slot 3 empty" is not. A stream that
  discovers a claim the brief did not anticipate ADDS a slot and says so; that
  addition is a brief gap and is counted as one.
- Merge order: dependencies first, hotspot-carriers last.
- ff-merge each green stream onto the train branch. **Never hand-resolve a
  source conflict** - mechanical resolution is how duplicate imports ship. A
  generated-file conflict is discarded and regenerated; a source conflict means
  the losing stream rebases and re-runs its own gate.
- **The integrator owns every `down_revision` at merge.** A stream cannot resolve
  its migration's parent from inside its own worktree, because the parent exists
  only in a sibling's - the migration tool then fails, the DB suites need a
  migrated database, and so **the stream cannot gate AT ALL** with the correct
  parent in place. Left unstated, every stream invents the same workaround (gate
  against the real head, flip the line before handover), which ships a commit
  containing a line no gate ever ran: the gated-tree-is-not-the-committed-tree
  class arriving by construction rather than by carelessness. So: the stream
  commits with the parent that exists in ITS OWN worktree and marks the line with
  its allotted slot and intended parent; the integrator repoints at merge, where
  the integrated gate proves it immediately. The stream's commit stays valid
  whichever way the sibling's migration goes.
- **Two streams that share no files can still collide.** File-level disjointness
  is necessary, not sufficient: streams also contend over SHARED OBJECTS no diff
  shows - a constraint or enum whose full membership each re-declares, a registry
  both enumerate, a linear namespace both draw from. Where each rebuilds a set
  from its own snapshot, whichever runs second silently deletes the other's
  additions, with no error until first use, and per-stream substrate guarantees
  neither stream's own gate can see it. Only the integrated gate can. **A rebase also re-runs the
  derived-artifact and staleness checks over the files the stream did NOT
  write** - that is the moment foreign changes arrive under its assumptions, and
  its own gate does not interrogate them.
- Regenerate the generated layer once, on the merged tree.
- **Run the full gate once on the integrated result.** This is the only run that
  proves the batch; green-alone proves nothing about the train.
- Semantic-conflict hunt: for each pair of streams whose work COUPLES - shared
  symbols, shared TABLES or stores, shared sequences and registries, or a shared
  PRACTICE one of them now enforces - name the specific way they could break
  together (new caller plus renamed callee, registry entry plus registry
  tripwire, template plus copy ratchet, two writers on one table, a ratchet that
  retroactively binds a sibling's diff) and check that specific thing. §2 already
  recognises sequenced resources and enforcement mechanisms as coupling; this
  hunt covers every form §2 names, not the symbol form alone. Symbols were the
  only form when this line was written and §2 grew two more without it
  following - two streams sharing a TABLE are green alone and wrong together,
  and the hunt would not have looked.
- **Red integrated gate with no obvious culprit: bisect by dropping streams**
  (halve, re-gate, repeat). At N=4 that is ~2 extra runs. Do not guess.
- Speculative option when substrate handles are free: gate (S1), (S1+S2),
  (S1+S2+S3) concurrently as streams land, taking integration off the critical
  path.

## 7. Ship

One push, one CI run, one deploy per the repo's deploy procedure. Close-outs
run once per train, not once per stream. Record the stream-sha to
integrated-sha mapping - rebase rewrites shas and QA verdicts reference the old
ones; the integrated gate is the correctness proof, the mapping is provenance.

**Close-out tears the train down.** Remove every stream worktree and delete
every stream branch once its content is on main under the mapped sha - the
pre-rebase commits linger as `ahead=N` residue and prune nothing on their own.
**The close-out is not complete until the counts are asserted by command and the
output pasted** - `git worktree list | wc -l` and `git branch --list | wc -l`,
both expected to be 1. **Close-out also reconciles the MANIFEST, by command:**
every manifest item is DISPATCHED, FOLDED-INTO-<n>, DEFERRED-with-reason or
DROPPED-with-reason, and the list is pasted. An item appearing in no ledger line
is a close-out failure, exactly as an unasserted worktree count is - three items
once sat on a manifest and appeared nowhere in 3,269 lines of ledger, and nothing
required anyone to notice. Stating them from memory is discipline and discipline
failed this twice; the pasted command is the enforcement, exactly as an ignore
line beats an untracked convention. Skipping this accumulates exactly the branch pile-up that makes a
later base check ambiguous, and it is how a clean repo turns into eight
long-lived branches nobody can merge.

**AND THAT RULE IS THE NARROW CASE OF A GENERAL ONE.** Worktree and branch counts
got a pasted-command enforcement because their absence burned someone twice. The
same argument applies to every step this method mandates, and stating it only for
the two that happened to burn someone is how a mandated step goes missing without
anyone noticing:

> **Any artifact that is the ONLY evidence a step ran is itself a DETECTOR.
> Retiring or replacing it silently removes a check.**

So: **every mandated step leaves an artifact whose ABSENCE is countable at
close-out**, and a convention change must carry each artifact's detector role
across or NAME what it is dropping. An artifact that is only on disk is not
durable enough to be a detector - it must be committed, because a file matched by
an ignore rule disappears without leaving a diff to notice.

This is the §2 absence question - *for every effect deferred, queued or skipped,
which surface SHOWS it and which action RECOVERS it?* - turned on the METHOD'S OWN
EXECUTION rather than on the product's data. Every absence class found until now
was about a ROW: 202-but-nothing-written, done-but-skipped, an idempotency key
consumed by a skip. A mandated STEP that silently never runs has the identical
shape, and nothing was looking for it.

**Measured, and it is why this rule exists.** QA is required by §5 for every
stream. It ran for many trains and then did not run for two consecutive trains, both of which
shipped to a live system taking real customer orders. Nobody noticed across two
close-outs and two retros, for two compounding reasons, each an instance of the
rule above:

    earlier   tNN-verdicts.log   held integrator notes AND QA verdicts
    later     tNN-reports/       stream reports only - zero QA files, ever

The convention change carried the stream half across and dropped the QA half, so
QA's output had nowhere to land. And the verdicts log had never been tracked
anyway - a `.gitignore` line matched `t*-verdicts.log`, so five trains of QA
evidence lived on one laptop and their disappearance produced no diff. The only
evidence the step ran was neither carried forward nor durable.

**An urgent single fix is not a train, and does not get to skip the gate.** It
runs solo in the shared tree with the full gate and a QA walk - urgency changes
the batching, never the verification. Do not start one while a train is in
flight: a hotfix landing under active worktrees is the shared-tree collision
this method exists to remove, so land or park the train first. The fix then
appears in the next train report's integrator side-effect enumeration, so
nothing ships unaccounted just because it was fast.

## 8. Report, then retro

One line each, plus a ledger - with verdicts and provenance read from
`.train/t<N>-verdicts.log`, not from memory: streams dispatched; green first
time; **and of
those needing a second round, how many were QA catching a real defect versus a
brief that was missing something** - only the second kind is a problem; wall
clock for longest stream, QA, integration, CI, deploy; estimate vs actual per
stream; files outside predicted sets; hotspot collisions hit; rework
attributable to the previous train.

**THE WALKER THAT FOUND THE DEFECT CLASSIFIES THE BOUNCE, not the integrator.**
The brief-gap split is the one number that falsifies the integrator's own
decomposition, and letting the integrator grade it is the same structural
conflict the retro names for the METHOD count - where it has now been measured
producing an eight-point gap. So the QA instance (or the stream, where the stream
found it) states in its verdict which of the two a bounce was, and the report
copies that verdict rather than re-deriving it. Cheaper than classifying in
flight and structurally sound rather than procedurally hopeful. A bounce whose
verdict does not say is UNVERIFIED, never zero (operator ruling, 2026-08-18).

**Enumerate every integrator side-effect that did not go through a stream** -
endpoint calls, direct DB writes, board changes, deploys, anything touching
state outside git - and name the sanctioned path each used. QA walks streams;
nothing otherwise walks the integrator, and an integrator writing raw where a
sanctioned endpoint exists silently skips whatever that endpoint also does
(notifications, audit rows, cache invalidation). This list is the only place
that class surfaces.

### Telemetry - one machine-readable row per train

The prose report above is for the operator reading THIS train. `.train/telemetry.tsv`
is for anyone asking whether the method works across many. Prose does not
aggregate, and hand-typed counts have been wrong in practice - append one
tab-separated row at close-out, same moment as the teardown assertion.

Header, and it does not change - a stable core is what makes a trend survive:

```
train  date  streams  spawns  walked_inline  green_first  brief_gaps
       cap_burn  files_out  rework  method_own  method_ext  tripwires  wall_min
       longest_min  tok_out  tok_cache_w  extra
```

**DERIVED - computed by command, never typed.** A number git or CI already holds
is recall the moment you type it (A3). Define the train range as
`<previous train tip>..<this train tip>`, then:

```
files_out    git diff --name-only PREV..TIP | wc -l      # vs predicted sets
wall_min     first author-time in range -> deploy completion
longest_min  slowest stream's dispatch -> its report
tok_out      output tokens over the train's date window
tok_cache_w  cache-creation tokens over the same window
spawns       agents spawned this train, against the session cap
```

Token accounting lives in the agent runtime's own transcripts; the path and the
deploy-workflow name are repo-specific, so `.train/config.md` records the two
commands that produce them. Cost is the first thing a sceptical adopter asks
about and the easiest number to hand-wave, so it is derived or it is UNVERIFIED.

**REPORTED - judgement, so each needs a written definition and a blind check.**
`green_first` (streams needing no second round), `brief_gaps` (omission - a class
the brief failed to name), `cap_burn` (commission - a factual error in the brief,
caught by the premise check), `rework` (defects traceable to a prior train),
`method_own` and `method_ext` (method changes from this train's discoveries
versus contributed from outside - never merge these, the readiness gate depends
on the split), `tripwires`, `blast_max`.

**`extra` is free-form `key=value` pairs** and it is where a new metric lives
while it proves itself. Never widen the core to trial something.

### Retiring a metric

**At every retro, each core column must name a decision it informed within the
last five trains, or it is dropped.** A column in `extra` that informs a decision
twice earns promotion to the core; one that never does is deleted without
ceremony.

This rule is not optional politeness. Readings that nobody acts on are exactly
the process machinery this method exists to remove - one repo measured 13% of
its entire recorded bug history as self-inflicted by agent-process overhead. A
metric you have explicitly stopped chasing has to justify its row as context or
lose it.

Then run `/train retro`. A train that does not feed its lessons back is a train
that will be run identically next time.

## Integrator discipline

The integrator decomposes, dispatches, integrates, ships. **It does not build,
and it does not author.** QA briefs are generated from (dispatch brief + Stream
Report), and must **restate the dispatch brief's declared deviations and
substrate budget verbatim** - generation that silently drops them is derivation
loss, and it burns QA capacity re-deriving what was already known. Attribute
every brief gap to the artifact that lost the information - dispatch brief or QA
brief - and record capacity-burned separately from defect-missed; they have
different causes and different fixes. A fix brief is the QA verdict handed back verbatim. Read the ledger
and verdict FROM DISK (`.train/t<N>-verdicts.log`), never from remembered
context; open a report body only when flagged, QA failed, or a hotspot was
crossed. Anything written that could be a template is latency on the critical
path - and the integrator's context, not infrastructure, is the first ceiling
you will hit.

**The integrator declares and fills its OWN population-ledger slots** (operator,
2026-08-20). Port grants, merge conclusions, session-state conclusions and any
census the integrator runs are population claims, and in one train six of the twenty-two
instances were the integrator's - including a rule it had authored into three
briefs hours before breaking it twice. Same three fills, same wording.

**ENFORCEMENT AND MEASUREMENT ARE SPLIT ON PURPOSE, and the split is not a
concession.** No merge-blocking on the integrator half for one train; the slots
are declared and filled so the compliance data exists, at no design cost. **The
blind classifier grades the integrator's slots at the next retro, not the
integrator.** The reason is structural: the party a mechanism constrains must not
be the only party judging whether it works, and deferring a constraint on the
party responsible for most of the instances - on that party's own recommendation
- is that conflict rather than a way around it. External judgement is the answer
to that conflict; delay is not.

Never block a stream on an operator answer: park it, dispatch the next.
Questions batch into one message, multiple choice, max three options,
recommendation first and labelled.
