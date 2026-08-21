# train retro - fold the train's lessons back

Run after every train. A train that does not feed its lessons back will be run
identically next time. This is the loop that makes the method compound.

Input: the §8 train report, every Stream Report, every QA verdict, and
`.train/priors.md`.

## Classify every lesson into exactly one destination

This is the whole discipline. Miscategorising is how the method either forks per
repo or gets poisoned by one repo's quirk.

| Destination | When | Where it goes |
|---|---|---|
| **PRIOR** | A finding class that will recur in this codebase | `.train/priors.md`, or the Brain if it is not repo-specific |
| **CONFIG** | A substrate fact moved - new harness, new CI, new hotspot ranking, isolation cap changed | `.train/config.md` |
| **METHOD** | The technique itself was wrong or incomplete, in a way that is not about this repo | Proposed edit to SKILL.md - **operator sign-off required** |
| **TRIPWIRE** | Third occurrence of the same finding class, **IF the class is expressible as a test** | An actual test, built in the NEXT train. If it is NOT expressible, it becomes a MECHANISM instead - see below |

Rules:

- **Default to PRIOR.** METHOD is the rare one. If a lesson mentions a specific
  file, tool, port or command, it is CONFIG or PRIOR, never METHOD.
- **METHOD changes hit every repo at once**, so they never land silently.
  Present them as a proposed diff with the evidence that motivated it.
- **Third of a class is a tripwire, not a checklist line.** A checklist line is
  an interim fix. First occurrence gets a line in the dispatch brief; third
  gets a test built in the same train that names it.
- **UNLESS the class is not expressible as a test - then the third occurrence
  becomes a MECHANISM** (operator ruling, from the retro that found it). The TRIPWIRE
  destination carried an unstated precondition: that a test CAN be written for
  the class. a completeness class - a census asserting "every" - reached its
  third occurrence and is not tripwire-able, because a test for it needs the
  ground-truth population, which is the very thing in doubt. A mechanism changes
  what the work produces so the class cannot arise; the population ledger is one
  (a stream must emit its evidence, so a claim cannot outrun it). Found by the
  blind classifier - the second consecutive train in which it located a defect
  in its own instructions.
- **Promotion: a prior appearing in TWO repos becomes a METHOD candidate.** One
  business, one ecosystem, one operator - a lesson learned in one codebase is
  almost certainly live in the next. Promote fast; the cost of a slightly
  over-general method is far below the cost of learning it twice.
- Anything cross-repo goes to the Second Brain inbox as well, loosely, for the
  Steward. Reconcile against existing notes - append a correction rather than
  contradicting an active note.

## Who classifies

**The party that benefits from a low METHOD count must not be the only party
counting.** Classification decides the readiness gate, and the integrator
running the retro is the same entity the gate constrains - that is a structural
conflict, not a character flaw, and it has produced under-classification twice
in practice.

Two tests to apply to every lesson before filing it as PRIOR:

1. **Would a different repo hit this?** If yes it is METHOD, however local the
   evidence looks. A tooling behaviour, a mechanism the method depends on, or a
   rule about a *class* of stream is never repo-local.
2. **Does the EVIDENCE require anything repo-specific to be true?** Strip the
   wording away entirely and ask whether the mechanism the lesson names exists
   in a repo with a different stack, a different language and a different CI. If
   it does, it is METHOD however the sentence was phrased.

   This test used to read "is the fix already worded universally?" - and that was
   broken, because it keyed on PROSE STYLE rather than claim content. A lesson is
   habitually written as a maxim, so the old test fired on nearly every lesson and
   could not be applied mechanically. Found by a blind classifier, which is
   the mechanism catching a defect in its own instructions (operator ruling,
   2026-08-18).

**Have the candidates classified blind by an actor that did not run the train** -
hand it the raw lesson list with your own classification withheld until it
reports. A reviewer shown your answer anchors to it and rubber-stamps; an
independent classifier produces a second opinion you can actually diff against.
Disagreement is the signal worth having.

**A classifier that returns NOTHING is re-run - never recorded as zero
findings.** Silence from the classifier is an UNKNOWN, not an absence of
disagreement, and this is the one place in the method where reading it as
absence is fatal rather than merely unhelpful: a null result recorded as zero
produces a retro that reports perfect agreement precisely when the check did not
run. Applies to a spawn that dies, one that goes idle without delivering, and one
whose output arrives empty. Observed once, where the first classifier instance
spawned, went idle and delivered nothing; the re-run then returned a METHOD share
eight points above the integrator's own and found a defect in the METHOD text
(operator ruling, 2026-08-18).

## Is the method ready to leave this repo?

Record the classification split every retro: how many lessons landed as PRIOR,
CONFIG, METHOD, TRIPWIRE.

**Count distinct FAMILIES, not lessons.** Seven lessons from one family is ONE
discovery, and a share computed over lessons reads it as seven. Collapse the list
to families first - the same collapse the classifier is asked to do - and compute
the share over those.

**Why: it measures the correct unit, and it surfaces severity.** Record both
numbers - the family count is the unit of discovery, the instance count is the
unit of urgency, and one column was conflating two different questions. A family
whose instance count is rising while its family count is flat is the shape that
most needs a test.

**What this change does NOT do, measured rather than assumed:** it does not
deflate the readiness signal. That was the original rationale and measurement falsified
it - family-level METHOD share came out 7/14 = 50%, IDENTICAL to the lesson-level
12/24 = 50%, because METHOD collapses at the same rate as every other bucket
(-42% both). The share is invariant under collapsing unless families cluster
differently by bucket, which is not the common case. Readiness-signal is struck
from the rationale; the ruling stands on unit-correctness and severity alone
(operator ruling; measured and found neutral the same day).

**METHOD share is the readiness signal.** A rising or steady METHOD share means
the technique is still being discovered and should not be rolled out to another
codebase yet - you would be propagating a draft. METHOD trending to zero across
consecutive trains means the method has stopped changing and the remaining
lessons are local, which is when a second repo is safe to onboard.

The first repo to run trains is the reference implementation, and the method is
provisional until its METHOD-change rate flattens. Say so plainly rather than
declaring the method finished.

## Read the telemetry, do not just append to it

`.train/telemetry.tsv` gained a row at close-out. The retro's job is to read the
COLUMN, not the row - a single train's numbers say almost nothing, and the
trend is the whole point of keeping them machine-readable.

Ask three things of the file every retro:

1. **Which column moved, and does the story explain it?** A number that changed
   without a reason in the lessons is either an unnoticed cause or a miscount.
2. **Which column has been flat for five trains?** Flat is not automatically
   dead - rework pinned at zero is the system working - but it must be named as
   either a held invariant or a candidate for retirement.
3. **Which column informed a decision this train?** Anything that cannot answer
   within five trains is dropped, per §8. Say which you dropped and why.

Promote from `extra` to the core only after a metric has informed a decision
twice. Widening the core casually is how a readable row becomes a spreadsheet
nobody opens.

## Measure the method, not just the work

Append to `.train/priors.md` each train, so calibration accumulates:

- **Estimate vs actual per stream.** Systematic bias in either direction is a
  decomposition problem, not an execution one.
- **Green-first-time rate.** Falling means briefs are getting thinner or slices
  wider.
- **Longest pole, named.** When it moves - build → QA → integrator → CI - the
  bottleneck has migrated and the next fix is somewhere new. Expect this;
  fixing one constraint always exposes the next. That is the shape of the work,
  not the previous fix having failed.
- **Files outside predicted sets.** Rising means predictions are degrading and
  §2 needs more evidence.
- **Rework attributable to the previous train.** The only number that says
  whether speed is costing quality.

## Priors file shape

```markdown
# .train/priors.md - <repo>
Read at dispatch. Appended by every retro. Lines here become dispatch-brief
checklist items; third occurrence of a class becomes a tripwire.

## Finding classes (occurrences)
- <class> (n=2) - <what to check at dispatch> - first seen <train>

## Calibration
- streams: <n> | est vs actual: <bias> | green-first-time: <rate>
- longest pole history: <train N: build → train N+1: QA → ...>

## Local traps
- <repo-specific gotcha that is not a finding class>
```

## Close

State plainly what changed and where it landed. If nothing was learned, say so
- a retro that manufactures lessons to look productive is worse than a short
one.


## The clock, and why there is no correction exemption

The readiness clock - two consecutive trains with a falling METHOD share and no
new §5 or §6 text - is reset by ANY new §5 or §6 text, including text that
corrects a defect in those sections.

A carve-out was considered once ("a correction is not a discovery, so it should
not reset the clock") and REFUSED. Every author believes their edit is a
correction; the distinction cannot be checked by anyone other than the author,
which makes the clause unfalsifiable. **A mechanical clause that occasionally
reads harshly beats a judgement-based one that always reads kindly.**

One train reset its own clock this way: the §6 coupling fix was a correction of a
defect §6 had carried for several trains, and it counts. The clock starts from that train.

## Three instances of the headline class inside the method's own artifacts

One cycle produced the train's dominant class - "something that is not the
thing satisfies the check" - three times inside the retro's OWN artifacts, not in
the code under review:

- the blind classifier's report existed only in conversation while a file
  claiming to hold it sat on disk;
- a figure went stale within four hours, inside a write-up about staleness;
- a header read "persisted verbatim" over a body explicitly marked "condensed".

This is recorded as evidence, not as a footnote of embarrassment. A class that
recurs inside the machinery built to catch it is a class that is real and
general - it is not a property of the code being reviewed, and an author's
familiarity with it does not confer immunity (operator ruling, 2026-08-18).