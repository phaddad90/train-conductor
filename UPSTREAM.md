# Upstream provenance

This repository is a **downstream port**. It is not where the method is authored.

The canonical home is a private repository where the method is used daily and
every folded change is committed with the train number, the ruling that
authorised it, and one line of evidence. This port is a periodic, sanitised
export: train numbers, ticket ids and repo-specific nouns are stripped, and the
mechanisms and their reasoning are kept.

**Canonical is private deliberately.** Making the public copy canonical would
turn every method change into a disclosure decision taken mid-train, at exactly
the moment nobody is inclined to sanitise, and one lapse would be permanent.

## Exported from

    upstream commit: f5adf532063b920f2f32d365eca2b13cfa3a188b
    exported:        2026-08-21

## Drift

A port whose staleness nobody can measure is just a stale artifact wearing a
different hat - which is the failure class the method itself names:

> Any artifact that is the ONLY evidence a step ran is itself a DETECTOR.
> Retiring or replacing it silently removes a check.

So the sha above is the detector. Upstream runs a check that reads it and reports
how many upstream commits are unexported. Before this export the port was five
trains stale and carried none of the population ledger, the QA law, the
durability clauses, the detector rule or the tripwire amendment - anyone adopting
it was getting a draft that predated most of what makes the method hold.
