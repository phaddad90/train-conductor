# Patterns

The recurring failure shapes this method exists to catch. Each one was paid for.
If you adapt the method, these are the rules to keep - they are the load-bearing
ones, and each looks like ceremony until you have hit it.

---

## Green but skipped

The single largest family. Something reports success without having done the
work, and everything downstream trusts it.

Seen as: a task runner exiting zero having executed nothing; a build tool exiting
zero without the manifest a route needs; a nightly backup succeeding against the
wrong datastore; a suite reporting "requires DATABASE_URL" identically whether
the variable is unset or merely unreachable; a cached green from a skipped run
replaying as a configured green.

**The question that catches it:** when a tool reports success here, what did it
actually *observe*? Name the artifact you checked, not the exit code.

Corollary: a check that was attempted but did not take effect is not a check.
That one rule generates three specifics - declare hunt classes before you look,
verify a mutation landed before its result counts, and execute a data migration
rather than SELECT-ing what it would touch.

## Tests that cannot fail

Measured in one train as the dominant defect class: nine-plus instances across
seven streams. A test written alongside its fix, never proven red, is decoration
that reads as coverage.

Fixed by moving mutation proof **left** - the builder reverts each fix and pastes
its test going red, before the gate, in its own worktree. Cheap there, expensive
anywhere else.

Then three refinements, each from a real escape:

- **Proof of arrival.** A test must demonstrate it reached the code it claims to
  exercise. One test failed under mutation because a validation layer *in front
  of* the target refused the input - it was exercising the guard in front of the
  guard.
- **Failure specificity.** Assert the reason, never a substring a generic or
  default message also contains. One permanent guard kept passing because an
  unrelated error message happened to contain the asserted substring, so it
  survived the very thing it documented being removed.
- **Discrimination.** For any test justified as "this proves we chose X over Y",
  ask what it would report if Y were implemented. If the answer is "the same", it
  never entered the discriminating region. Two of three such tests passed against
  the rejected implementation.

## Shared mutable substrate

Worktrees solve file collisions and nothing else. The real ceiling on parallel
agents is whatever fixed, mutable thing they all reach.

Seen as: a template database dropped and rebuilt under a process-local guard,
destroying a sibling mid-clone - surfacing as ~37 unexplained failures rather
than a collision; a browser navigated away mid-measurement by a concurrent agent;
a shared virtualenv where one install silently changes what every sibling's gate
tests against; a `__dirname`-resolved output directory, correctly unique per
worktree and still shared by every test file in one run.

**Enumerate every shared resource once, and state the scope at which each is
isolated.** The right answer at the wrong scope reads as a clean bill of health.

## Hand-maintained lists inside enforcement

An enforcement mechanism that flags by a hand-kept list decays as the thing it
guards evolves, and the decay is invisible because the mechanism still runs.

Seen three times in one repo inside three weeks: a column list built from the
first migration that had no idea a later table used a different type by design,
producing five flags and five false positives; an effect-word list that made the
same measurement wrong twice, at 12 and then 6, before it was actually 1; and a
third that was *correctly* hand-kept.

The distinction matters. A vocabulary that evolves inside the repo must be
derived from its source of truth. One that is external and fixed has no in-repo
source and a hand list is right. **Write down which case you are in**, or the
next reader cannot tell a justified list from a rotting one.

## Recording without enforcing

A repo accumulates written lessons and almost no guards. The tell is a lesson
whose own text says "third-and-counting instance".

**A lesson is not closed until it has a guard, or a written reason why no guard is
possible.** Third occurrence of a class is a guard, not another checklist line - a checklist line is an interim fix.

## Nobody reviews the orchestrator

Review walks streams. Nothing walks the integrator, and its errors propagate with
full authority because streams comply exactly.

Two distinct halves, and both need instruments:

- **Side-effects.** Every integrator action that did not go through a stream - endpoint calls, direct database writes, deploys - enumerated with the
  sanctioned path each used. One integrator was writing directly to a table where
  a sanctioned endpoint existed, silently skipping the notification that endpoint
  also fires. Asked to audit itself once, it found a second bypass in under a
  minute.
- **Judgement.** Three mechanical checks on the manifest before dispatch: every
  predicted file exists or is marked to-be-created; every acceptance criterion
  names the mutation *and* the failure it must produce; every figure is a median
  of ≥5 runs or is labelled UNVERIFIED. Each of those caught a real dispatch
  error - a stream sent against a directory not in the repo, a criterion
  impossible to satisfy honestly, and a single timing sample taken on a busy
  machine that was 7× wrong and nearly bought an optimisation programme.

## Compliance without refusal

A stream complies exactly. It cannot easily refuse a direction that arrives with
a rationale and a bound - which makes an inherited diagnosis dangerous.

**A ticket's diagnosis is a hypothesis, not a spec**, and so is the orchestrator's
own prescription. When a mechanism swap is directed - replace this parser, use
that library's output - the brief must require the stream to enumerate what the
old mechanism covered and prove the new one still does. One swap was sound
reasoning and a net -55 lines, and silently stopped covering two layers.

## Unverified claims

An agent states a cause it never measured. Two mechanically detectable tells,
both from real reports:

- **A hedge on a countable quantity** - "4,600+ lines", "roughly", "over" - is the
  signature of recall rather than measurement. The real figure was 5,453.
- **Scope inflation** - measuring a 20-commit window and claiming "zero branch
  structure" over 90 days. There were 17 merges.

A third: a grep for a token inside a file that *documents* that token matches its
own comment. Absence-of-token claims about source must exclude comments or parse
structure.

## Process artifacts riding a merge

Anything the process itself creates inside a worktree needs an ignore rule
shipped with the process, not a convention. Two streams writing the same report
path is an add/add conflict at integration, and "left untracked" is defeated by
`git add -A`.

## Correlation read as a seam

Before instrumenting the top of a co-change ranking, explain *why* it is there.
One repo's worst apparent hotspot sat in 52% of all commits and was a version
counter bumped on every deploy; its real structural rate was 1.5%. A registration
seam at 52% would make parallel work near-impossible. A counter costs nothing - the integrator owns it and bumps it once per train. Opposite conclusions from the
same number, and only measurement separates them.
