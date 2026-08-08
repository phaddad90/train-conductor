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
- **PRIORS** - the shared knowledge store first, `.train/priors.md` only for the
  genuinely repo-local. Read before dispatch. Appended by retro.

## Modes

| Invocation | Does |
|---|---|
| `/train` | Run a train. Requires `.train/config.md`. |
| `/train init` | Onboard or retrain a repo. Read `references/init.md`. |
| `/train retro` | Fold the last train's lessons back. Read `references/retro.md`. |

**No `.train/config.md`? Stop and run init.** Never guess the substrate.

**One repo, one remit.** A train never spans repositories, and an agent is never
tasked with another repo's init, config, priors or backlog - each repo's agent
owns its own. The method is shared; the work never is. The shared knowledge store is the
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
  canonical failure.
- **A3 Recall.** Any figure not re-derived this session is recall. A hedge on a
  countable quantity - "4,600+", "~", "over", "roughly" - is the tell.
  Countable things get counted.
- **A4 Falsify.** Take the three claims the conclusion leans on hardest and try
  to prove each wrong. Say what was tried.
- **A5** Subagent claims inherit zero trust; run A1-A4 on them yourself.
- **A6** Assume every number will be re-derived downstream.

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

**Refuse to start if the tree is dirty or work is in flight.** Land or park it.
Changing integration model with work in flight is how eight-branch pile-ups
happen.

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
- **A stream that adds an enforcement mechanism constrains every sibling.** A
  ratchet, tripwire, lint rule, schema constraint or ACL retroactively binds all
  other work in the same train - one-to-many, not pairwise, so the dependency
  rule above does not catch it. Such a stream is dispatched FIRST with siblings
  branching off its merge, or held to the next train. Never concurrent with the
  work it will constrain: no sibling's brief can carry a requirement that does
  not exist yet at dispatch. **The concurrency exemption is judged on PRACTICE,
  not file overlap** (observed across two consecutive trains): a ratchet policing a practice
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
- Show the operator the manifest with its ledger. Do not wait for approval
  unless a hotspot claim is contested or an item needs an operator input.

## 3. Dispatch - every stream gets all of this or it does not launch

- Its own git worktree and a local branch off the current train head. Never the
  shared tree. Never a push to origin unless config says otherwise.
- **Verifying the base is the stream's FIRST act, before any edit.** Worktree
  tooling spawns at a main-line sha, not necessarily the branch you declared -
  observed failing 3 of 3 times. The brief states the expected base sha; the
  stream asserts it, ff/resets if wrong, and says so in its report. Without
  this, co-queueing silently builds the second claimant on the wrong file state
  and every "clean merge" is luck.
- Brief: goal, acceptance evidence required, predicted file set, migration slot
  or "none", isolated substrate handles (test DB port/namespace per config, and
  any singleton resource - browser, REPL, shared session, shared dev or staging
  database - budgeted exactly like a port), model tier, and the relevant lines
  from `.train/priors.md`.
- **Every operator input - screenshot, ruling, credential, sign-off - is
  requested at manifest time in one batched list.** Never discovered mid-build.
- Hard stops; report and halt rather than work around: config-owned data the
  operator owns, a production write, a side-effectful external retry, the same
  test failing three times, the file set exceeding predicted + 2, or **needing a
  new dependency** - installed packages are near-always shared state (a symlinked
  virtualenv, a hoisted node_modules, a vendor dir), so one stream's install
  silently changes what every sibling's gate is testing against. Nothing fails;
  the results just stop meaning what they say.

## 4. Stream exit - the gate is mechanical, runs ONCE, by the stream

Run the repo's exact gate command from `.train/config.md`, in the worktree,
against its own test substrate. Do not run a second overlapping suite behind it.

**Mutation proof runs LEFT, at build time, by the builder** (T13/T14,
operator-approved): before the gate, the stream reverts each of its fixes in
turn and pastes its own new tests going red, then restores. QA replays rather
than discovers; a stream whose tests were never proven red ships tests that
cannot fail, measured in one train as the dominant defect class (9+ instances,
7 streams). A mutation must vary exactly ONE thing - changing two validates
the test instead of the code (observed: a false proof from a two-variable mutation). A red that fails for the
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
UNVERIFIED, any file outside the predicted set, and the claim ledger. Full
evidence goes to `STREAM-REPORT.md` in the worktree, **at a path the repo
gitignores**, with the path cited. Report artifacts never ride a merge - two
streams writing the same tracked path is an add/add conflict at integration.
Untracked-by-convention is not enough; `git add -A` defeats it, so the ignore
line is the enforcement and `.train/config.md` names the path.

## 5. QA - adversarial only, concurrent, one instance per stream

- **Spawned by that stream's completion**, not batched by the integrator. QA
  for a finished stream runs while other streams are still building. Each QA
  instance holds its own test substrate handle.
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
    the stream added - **and verify the mutation LANDED before its result
    counts**: re-read the value, re-import the module, check the diff. A probe
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
    proves syntax, not behaviour.

  Then: run A4 against the stream's ledger, and hunt the absence cases.
- For claims of the class that keeps failing (absolutes), prefer 2-3 verifiers
  each given a *different slice* of context, with disagreement as the trigger,
  over one verifier with everything.
- Verdict is PASS or a specific failure with file:line. Fixes are re-walked as
  a delta - never a "smalls" round that lands unwalked.
- If a QA walk exceeds its stream's build time, say so in the train report: the
  walk is doing work the gate should own.

## 6. Integrate - the integrator's only build-adjacent job

- Merge order: dependencies first, hotspot-carriers last.
- ff-merge each green stream onto the train branch. **Never hand-resolve a
  source conflict** - mechanical resolution is how duplicate imports ship. A
  generated-file conflict is discarded and regenerated; a source conflict means
  the losing stream rebases and re-runs its own gate.
- Regenerate the generated layer once, on the merged tree.
- **Run the full gate once on the integrated result.** This is the only run that
  proves the batch; green-alone proves nothing about the train.
- Semantic-conflict hunt: for each pair of streams touching related symbols,
  name the specific way they could break together - new caller plus renamed
  callee, registry entry plus registry tripwire, template plus copy ratchet -
  and check that specific thing.
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
both expected to be 1. Stating them from memory is discipline and discipline
failed this twice; the pasted command is the enforcement, exactly as an ignore
line beats an untracked convention. Skipping this accumulates exactly the branch pile-up that makes a
later base check ambiguous, and it is how a clean repo turns into eight
long-lived branches nobody can merge.

**An urgent single fix is not a train, and does not get to skip the gate.** It
runs solo in the shared tree with the full gate and a QA walk - urgency changes
the batching, never the verification. Do not start one while a train is in
flight: a hotfix landing under active worktrees is the shared-tree collision
this method exists to remove, so land or park the train first. The fix then
appears in the next train report's integrator side-effect enumeration, so
nothing ships unaccounted just because it was fast.

## 8. Report, then retro

One line each, plus a ledger: streams dispatched; green first time; **and of
those needing a second round, how many were QA catching a real defect versus a
brief that was missing something** - only the second kind is a problem; wall
clock for longest stream, QA, integration, CI, deploy; estimate vs actual per
stream; files outside predicted sets; hotspot collisions hit; rework
attributable to the previous train.

**Enumerate every integrator side-effect that did not go through a stream** -
endpoint calls, direct DB writes, board changes, deploys, anything touching
state outside git - and name the sanctioned path each used. QA walks streams;
nothing otherwise walks the integrator, and an integrator writing raw where a
sanctioned endpoint exists silently skips whatever that endpoint also does
(notifications, audit rows, cache invalidation). This list is the only place
that class surfaces.

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
and verdict; open a report body only when flagged, QA failed, or a hotspot was
crossed. Anything written that could be a template is latency on the critical
path - and the integrator's context, not infrastructure, is the first ceiling
you will hit.

Never block a stream on an operator answer: park it, dispatch the next.
Questions batch into one message, multiple choice, max three options,
recommendation first and labelled.
