# Concepts

The vocabulary, and why each piece is shaped the way it is. Read this before
adapting anything - most of these rules look like ceremony until you know which
failure produced them.

---

## Stream

One slice of work, in its own worktree, on its own branch, with a **predicted
file set** declared before it starts. The prediction is not bureaucracy: it is
what bounds exploration, and it is the single biggest reason briefed streams run
3-13 minutes where unbriefed ones ran 37-75.

A stream stops and reports rather than working around: operator-owned config, a
production write, a side-effectful external retry, the same test failing three
times, its file set exceeding predicted + 2, or **needing a new dependency**.
That last one surprises people. Installed packages are near-always shared state - a symlinked virtualenv, a hoisted `node_modules`, a vendor directory - so one
stream's install silently changes what every sibling's gate is testing against.
Nothing fails; the results just stop meaning what they say.

## Integrator

Decomposes, dispatches, integrates, ships. Does not build. Does not author.

Review briefs are *generated* from (dispatch brief + stream report), not
written. A fix brief is the review verdict handed back verbatim. Anything the
integrator writes that could have been a template is latency on the critical
path, and the integrator's context is the first ceiling the method hits - before
CPU, before runners, before anything you can buy.

## Gate

The repo's real verification command, run **once**, **by the stream**, in its own
worktree, against its own substrate. Not by the integrator afterwards, not again
by review.

The gate must be *executed* during onboarding, never read off a `package.json` or
`Makefile` and assumed. A gate can be written and never have worked. One adoption
found a task runner exiting non-zero on a workspace-resolution error that had
never been fixed, with 1,116 real tests sitting unreachable from the build graph - and a second defect underneath it where the runner stripped environment
variables, so the test-isolation seam that had been *proved* by hand would have
silently not worked under the gate.

## Ratchet

A stream that adds an enforcement mechanism - a lint rule, a schema constraint,
an architecture test, an ACL. It binds *every* sibling in the train the moment it
lands, so it dispatches **first**, with siblings branching off its merge, or
waits for the next train.

Its exemption is judged on **practice, not file overlap**. A ratchet policing how
tests assert or how errors render touches every sibling with zero shared files;
one adoption used the file-overlap reading and paid three same-train recurrences
of the exact class the ratchet banned.

A ratchet also has to survive its own acceptance criteria:

- **Enumerate the recognition axes** before dispatch, by grepping. Skipping this
  turned one detector into a seven-round review ladder costing ~2.5 hours against
  a 15-minute build, six rounds of which were greppable up front.
- **Evade its own detector.** A guard keyed on a name rather than a binding is
  bypassed by shadowing.
- **Report its precision** - every flag it currently raises, triaged true or
  false. One detector shipped at five flags and five false positives. A detector
  that wrong gets ignored within a week, after which the class it guards is
  silently unenforced *while still looking enforced*, which is worse than no
  detector at all.
- **Justify any hand-maintained list.** A vocabulary that evolves inside the repo
  (schema columns, route names, registry entries) must be derived from its source
  of truth - a hand copy is the mechanism's own decay. A vocabulary that is
  external and fixed (a language's builtins, another engine's function set) has
  no in-repo source and a hand list is correct. Write down which case you are in,
  or the next reader cannot tell a justified list from a rotting one.

## Mutation proof

**The builder proves its own new tests can fail, at build time, before the gate.**
It reverts each fix in turn, pastes the test going red, then restores.

This moved left from review after one train measured unfalsifiable tests as its
dominant defect class - nine-plus instances across seven streams. A test that
cannot fail is worse than no test, because everything downstream trusts it.
Review then *replays* the proof rather than discovering it, which is much
cheaper and much later otherwise.

Three ways a mutation proof lies:

- **Two variables.** A mutation must vary exactly one thing; changing two
  validates the test rather than the code.
- **Wrong reason.** A red caused by a syntax error or a broken import is not a
  proof. Report failed mutation attempts as what they were.
- **Never landed.** An edit that lands in a docstring instead of a table value,
  or a patch applied to a copy, reads as "confirmed". Verify the mutation took
  effect before its result counts - and commit before mutating, because
  reverting with a working-tree checkout against uncommitted changes destroys
  the fix rather than the mutation.

And a mutation only proves a test *can* fail. It does not prove the test fails
for the **reason claimed** - see [PATTERNS.md](PATTERNS.md) on proof of arrival
and failure specificity.

## Hotspot

A file appearing in more than roughly 5% of commits. Two items touching the same
hotspot **co-queue** - same train, dispatched sequentially, the second branching
off the train head after the first merges. Excluding the second to a later train
adds delay and buys no safety; same-scope changes need to be tested *together*,
which is where semantic conflicts surface.

Measure hotspots every train. Do not store them. One adoption found its worst
"hotspot" at 52% of all commits and it was a version-string counter bumped on
every deploy - the real structural rate was 1.5%. The two answers have opposite
implications and only measurement distinguishes them.

## Investigation

Work whose file set is unknowable at dispatch. The test is whether you can state
the **question** precisely now - not whether you can answer it. A sharp question
with an unpredictable file set is an investigation. A question you cannot phrase
sharply is not ready to dispatch at all, and saying so is the honest answer.

An investigation runs read-only and produces a *finding plus a proposed brief*.
The fix becomes a properly predicted stream afterwards. This is not overhead:
one investigation disproved its own premise and killed a build before it
existed, which is the highest-value outcome available.

A manifest cannot claim "no hotspot collisions predicted" while any stream's file
set is unknown. That is an unfounded absolute, not a prediction.

## The answer protocol

Six rules governing every output - manifests, stream reports, review verdicts,
anything said to the operator.

- **Ledger.** Every number carries the command that produced it and the output
  containing it. Cannot paste the output? Label it UNVERIFIED.
- **Scope.** State the window measured and the window claimed. Absolutes - *never, always, zero, none, every* - require a population-wide command.
- **Recall.** Any figure not re-derived this session is recall. A hedge on a
  countable quantity ("4,600+", "roughly", "over") is the tell. Countable things
  get counted.
- **Falsify.** Take the three claims the conclusion leans on hardest and try to
  prove each wrong. Say what was tried.
- **Subagent claims inherit zero trust.**
- **Assume every number will be re-derived downstream.**

Two derived rules earn their own lines. *A ticket's diagnosis is a hypothesis,
not a spec* - a brief-inherited diagnosis gets verified as the stream's first
analytical act, because a stream complies exactly and cannot easily refuse a
direction that arrives with a rationale. And *the integrator's own prescription
is a hypothesis too*: when it directs a mechanism change rather than a defect
fix, the brief must require the stream to enumerate what the old mechanism
covered and prove the new one still covers it. One such swap was sound reasoning
and a net -55 lines, and silently lost coverage in two layers.

## Priors

The accumulated failure classes for a repo, read at dispatch and appended by
every retro. This is what makes a new agent start at the current competence
rather than at zero.

Shape them as **class-level questions with concrete tells nested underneath**,
never a flat list of specifics. A flat list accumulates past twenty items and
stops being read. Two independent adoptions arrived at this shape without
knowing about each other.

## Closure rule

**A lesson is not closed until it has a guard, or a written reason why no guard
is possible.** The second is a legitimate answer - "prose cannot be statically
checked, enforcement stays in review walks" is a real closure - but it must be
written, not left blank.

Without this you get a recording culture and no enforcing culture, which is how
a repo ends up with dozens of documented lessons and one of them reading
"third-and-counting instance".
