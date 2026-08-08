# The release readiness checklist

This is the checklist behind issue #115. It is what stands between a green gate
and something an operator installs, and it is written before the first release
rather than beside it, for the same reason the bar in #13 was written before any
interface was drawn. A checklist written at release time is a description of the
release.

It is not a summary of the gate. The gate refuses what it refuses and needs no
help from a document. Every item below is something no check can decide, and
each one is asked for by name so that a release cannot happen by momentum.

## How an item is discharged

Each item states the evidence that discharges it. The evidence is a run, a
transcript, a link or a number, and a statement that the item was handled is not
evidence.

An item that cannot be discharged is recorded as not done, in the completed
instance and in the release notes, with the reason. It is not ticked because the
release is otherwise ready. A checklist that has never held up a release is a
checklist nobody is using, and the first time this one is inconvenient is the
first time it is doing anything.

Nothing in a completed instance is about anything other than the work. It
records who ran it and when, and the evidence each item produced.

## The items

### 1. The round trip ran green on the release commit

What discharges it: a link to the #99 run, with the commit it ran against, and
that commit is the one being released rather than an ancestor of it.

What to do when it cannot be discharged: the release does not go out. This is
the one item on the list with no honest alternative, because #99 is the only
thing that exercises the stages against each other rather than against each
other's stubs.

### 2. The hardware harness has run against a real machine

What discharges it: a recorded #78 run against a real machine of the class the
release targets, naming the machine and what was cut.

What to do when it cannot be discharged: the release says plainly that no
produced program has been run on a machine of the target class, in the release
notes rather than only here. That sentence is not softened in a later edit.

### 3. A restore was performed from a backup

What discharges it: a transcript of an #85 restore, taken from a backup made by
the documented procedure rather than by hand, and a statement of what was
compared to establish that the restored store matches.

What to do when it cannot be discharged: the release does not go out with a
backup procedure nobody has restored from. A backup that has never been restored
is a file of unknown contents, and the store is the thing an operator cannot
rebuild.

### 4. The upgrade path was exercised from the previous release

What discharges it: a transcript of an #112 upgrade from the previous release to
this one, including whatever store migration #46 performs, and a statement of
whether a rollback was possible afterwards and how that matches what
`docs/version-policy.md` promises for this release's number.

What to do when it cannot be discharged: on the first release there is no
previous one and the item is recorded as not applicable rather than as done. On
any later release, an unexercised upgrade path is recorded as not done and the
release notes say so.

### 5. The readme's claims are supported

What discharges it: for each claim in the readme after #114 has rewritten it,
the item or the run in this list that supports it, or a note that it is
unsupported.

What to do when it cannot be discharged: the claim is changed rather than the
item ticked. A readme is the first thing somebody reads and the last thing
anybody checks.

### 6. The notices cover what actually ships

What discharges it: a comparison between the #101 notices and the contents of
the artefact built by #108, made against the artefact rather than against the
manifest that was meant to produce it.

What to do when it cannot be discharged: the release does not go out. The notices
are the operator's route to their own obligations, and a notices file that does
not match what shipped is worse than a missing one because it reads as complete.

### 7. The published artefact verifies

What discharges it: a transcript of somebody following the published #113
procedure against the artefact on the release page, using tooling the procedure
names and nothing from this project.

What to do when it cannot be discharged: the release does not go out. A
signature nobody has verified by the published route is a signature whose
procedure has not been tested.

### 8. The bar was measured and its result is stated

What discharges it: the #67 result for every element of
`docs/decisions/0013-the-bar-for-a-coherent-workflow.md`, with the number, or an
explicit statement that an element was not measured.

What to do when it cannot be discharged: the numbers that exist are stated and
the ones that do not are recorded as not measured. A missing row is not a
passing row, and a bar reported only where it was met is not a measurement.

### 9. The deployment statement carries measured numbers

What discharges it: the #117 statement, with the measured number for each unit
it names and the command and run behind each one, and the reference project it
was measured on.

What to do when it cannot be discharged: the units that were not measured say so
in the statement and in the release notes, in the same place as the ones that
were. A release does not go out with a number that was never measured unless the
release says so where the number is read.

## The rule that makes the list worth anything

An item that cannot be ticked is recorded as not done and the release says so.

Four of the nine items above say the release does not go out. Those four are
where the failure is not recoverable by disclosure: an unrun round trip, an
unrestored backup, notices that do not match the artefact, and an unverified
signature. The other five can be disclosed, and a disclosure in the release
notes is the whole cost of them.

That split is a decision made while this document was written rather than one
taken from anywhere, and it is the part of this checklist most worth arguing
with before the first release rather than during it.

## What this document is not yet

#115 closes on an executed checklist rather than on a written one, and this is a
written one. There is no release to run it against:

    gh api repos/iderex/reissbrett/releases --jq 'length'
    0
    git ls-tree -r --name-only origin/main | grep -cE '\.(go|py|cs|rs|ts|js|cpp|c|java)$'
    0

Every issue named above is open, so no item on this list can be discharged
today:

    for n in 67 78 85 99 101 108 112 113 114 117; do gh api repos/iderex/reissbrett/issues/$n --jq '"#\(.number) \(.state) \(.title)"'; done
    #67 open Measure the committed path against the bar
    #78 open Build the machine harness, and gate it honestly
    #85 open Backup and restore, proven by a restore
    #99 open The deviation upward: run the real suite round trip on a schedule
    #101 open Publish third party notices an operator can read
    #108 open Build the bundle an operator installs
    #112 open Make upgrades safe, and say what an operator has to do
    #113 open Sign and publish the release artefacts
    #114 open Rewrite the readme for a project that can be run
    #117 open State the size of deployment this release supports, and measure it

The completed instance of a run is a separate file added beside this one and
never an edit to it, so that what was asked stays readable next to what was
answered. Where that file lives is settled when the first one is written, which
is also when #15's layout has had something to say about `docs/`.
