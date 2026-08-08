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
| **TRIPWIRE** | Third occurrence of the same finding class | An actual test, built in the NEXT train |

Rules:

- **Default to PRIOR.** METHOD is the rare one. If a lesson mentions a specific
  file, tool, port or command, it is CONFIG or PRIOR, never METHOD.
- **METHOD changes hit every repo at once**, so they never land silently.
  Present them as a proposed diff with the evidence that motivated it.
- **Third of a class is a tripwire, not a checklist line.** A checklist line is
  an interim fix. First occurrence gets a line in the dispatch brief; third
  gets a test built in the same train that names it.
- **Promotion: a prior appearing in TWO repos becomes a METHOD candidate.** One
  business, one ecosystem, one operator - a lesson learned in one codebase is
  almost certainly live in the next. Promote fast; the cost of a slightly
  over-general method is far below the cost of learning it twice.
- Anything cross-repo goes to the shared knowledge store as well, loosely, for the
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
2. **Does the fix, as you wrote it, mention this repo?** If the wording is
   already universal ("every stream brief carries…"), you have written a METHOD
   change and filed it as a prior.

**Have the candidates classified blind by an actor that did not run the train** -
hand it the raw lesson list with your own classification withheld until it
reports. A reviewer shown your answer anchors to it and rubber-stamps; an
independent classifier produces a second opinion you can actually diff against.
Disagreement is the signal worth having.

## Is the method ready to leave this repo?

Record the classification split every retro: how many lessons landed as PRIOR,
CONFIG, METHOD, TRIPWIRE.

**METHOD share is the readiness signal.** A rising or steady METHOD share means
the technique is still being discovered and should not be rolled out to another
codebase yet - you would be propagating a draft. METHOD trending to zero across
consecutive trains means the method has stopped changing and the remaining
lessons are local, which is when a second repo is safe to onboard.

The first repo to run trains is the reference implementation, and the method is
provisional until its METHOD-change rate flattens. Say so plainly rather than
declaring the method finished.

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
