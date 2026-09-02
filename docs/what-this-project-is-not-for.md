# What this project is not for

`NOTICE.md` already says that this software is developed for lawful use and
that operators are responsible for their own compliance. That is the general
statement, and every repository can make it. This document is the specific one,
and it exists because of what this project sets out to produce: files that make
a machine move.

Each section below says what this project does not do. None of them carries a
reassurance that takes it back. That is a rule about how this file is edited
rather than a note about its tone: if a later change makes one of these limits
narrower, the statement here gets sharper rather than gentler. A limit softened
into something comfortable is worse than the limit it replaced, because
somebody reads the comfortable version and stops there.

## Nothing here is a report on a running program

The limits below are written before the software they describe exists, for the
same reason the bar in #13 was written before any interface was drawn. A limit
written afterwards is a description of what was built.

This repository holds documents and workflow files and no code:

    git ls-tree -r --name-only origin/main | grep -cE '\.(go|py|cs|rs|ts|js|cpp|c|java)$'
    0

There is no released version and no tag:

    gh api repos/iderex/reissbrett/releases --jq 'length'
    0
    git ls-remote --tags https://github.com/iderex/reissbrett | wc -l
    0

So every sentence below is a statement of what the first release commits to,
and none of them is an observation of behaviour anybody has seen.

## The operator verifies every produced program before running it

A program this project produces is a proposal about how a machine should move.
It is checked by the person who is standing at the machine, on the machine they
are standing at, before it runs. That is not a formality that can be delegated
to the software, and this project does not offer to take it over.

The operational half of this, what an operator does the first time a new
program runs on a machine and what the checks in the software cover, is #79,
and it is not written yet. This section is the statement rather than the
procedure.

## This project certifies nothing

A drawing produced here is not an inspection record. A model held here is not a
qualified design. A stored version is a record of what a file contained, not
evidence that the thing it describes was ever correct.

If an operator's field requires a process, a sign-off, an approval or a
qualification, nothing this project does is a substitute for it, and nothing
here is evidence towards it unless whoever runs that process says so.

## This project is not a safety system

The plan carries two checks between a model and a machine. #75 verifies a
toolpath against the stock, the fixture and the tool geometry and refuses
output on a collision, and #76 proves a produced program on a simulator. Both
are open and neither exists:

    for n in 75 76; do gh issue view $n --repo iderex/reissbrett --json number,state,title --jq '"#\(.number) \(.state) \(.title)"'; done
    #75 OPEN Verify the toolpath against the stock and refuse a collision
    #76 OPEN Prove the produced program on a simulator before any metal is cut

Until they do, no automated check in this project examines a toolpath at all,
and the operator's own verification is the only thing between a program and a
machine.

When they exist they will reduce the chance of one class of error, which is a
tool, a holder or an axis arriving somewhere the model says it should not. They
will not make a program safe. A simulation that matches says that the software
and the simulator agree, and it says nothing about the machine's actual state,
the fixturing, the workholding, the material, or the tool that is really in the
spindle, because none of those is visible to any of this.

Which class of error each check reduces, stated against what those checks
actually do rather than against what this section expects of them, is #79's
wording and belongs there. This document does not write it in advance.

## Where this project's claims stop

The readme claims a workflow, collaboration and versioning, and that the work
stays on the operator's own hardware:

    git show origin/main:README.md | sed -n '3p'
    A distribution and orchestration layer over the open modular CAD suite that adds a guided path through one workflow (sketch to feature to drawing to CAM), with team collaboration and versioning. It runs on your own hardware; the models and the project history never leave it.

Those are the claims. Outside them:

The readme holds two things out of that sentence deliberately. Whether the guided
path is usable by somebody new to CAD, and whether the interface stays consistent
along it, are named there as intentions with no measurement behind them, so
neither is a claim this document has to place a limit on.

This project implements no geometry kernel, no constraint solver and no
toolpath generator, which `.github/SECURITY.md` already says in its own words
for a different purpose. Accuracy, tolerance behaviour and the results of any
geometric operation are the underlying suite's, and this project claims nothing
about them beyond what that suite claims for itself.

A part that passes every check this project makes is not thereby
manufacturable. Whether it can be made, on the machines and with the material
and the time available, is a judgement this project has no way to make and does
not offer.

Nothing here claims that a version history is a backup. #85 owes an operator a
backup and a restore, and until an operator has run one, a store on one machine
is one machine's worth of durability.

## Consistency with the licence

The licence disclaims warranty and liability in its own terms:

    git show origin/main:LICENSE | sed -n '587,596p'
      15. Disclaimer of Warranty.

      THERE IS NO WARRANTY FOR THE PROGRAM, TO THE EXTENT PERMITTED BY
    APPLICABLE LAW.  EXCEPT WHEN OTHERWISE STATED IN WRITING THE COPYRIGHT
    HOLDERS AND/OR OTHER PARTIES PROVIDE THE PROGRAM "AS IS" WITHOUT WARRANTY
    OF ANY KIND, EITHER EXPRESSED OR IMPLIED, INCLUDING, BUT NOT LIMITED TO,
    THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR
    PURPOSE.  THE ENTIRE RISK AS TO THE QUALITY AND PERFORMANCE OF THE PROGRAM
    IS WITH YOU.  SHOULD THE PROGRAM PROVE DEFECTIVE, YOU ASSUME THE COST OF
    ALL NECESSARY SERVICING, REPAIR OR CORRECTION.

Nothing above contradicts that, and nothing above relies on it either. A
warranty disclaimer is a statement about legal liability written for a court.
This document is a statement about capability written for somebody deciding
whether to trust a file with a machine, and the second one is the one that is
useful before anything goes wrong.

The two are not interchangeable in the other direction either. A limit stated
here stays stated if the licence ever changes, because it is a fact about what
the software does rather than a term of the offer.

## What this document does not yet satisfy

#105 asks for four things this landing does not do, and none of them should be
read as done.

It asks that this document be consistent with #79's warning, checked against
it. #79 has produced nothing, so half of that check has no counterpart to run
against, and the half that could run is the licence, above.

It asks that this be reachable from the readme and from the software rather
than only from the repository. There is no software. The readme is #114's, and
a link added here would be a change outside the scope this work declared.

It asks that the review checklist ask whether a change narrowing a limit made
the statement sharper. The question is now asked, in
`.github/pull_request_template.md`, which is where a question put to a change
belongs; the nine items in `docs/release-readiness-checklist.md` are evidence a
release carries, and a change is reviewed long before one of those is run.

Asked is the whole of it, and it is worth being blunt about how little that is.
A template is a prompt. Nothing reads what somebody writes under it, nothing
compares this document against a change that narrowed a limit, and a pull
request that deletes the section entirely is refused by nothing. So the rule at
the top of this document is still the only thing holding the sharper statement,
and it is a rule about how a file is edited, which is the kind this tree has
nothing to read.

#95's check exists now, and naming what it reaches makes that disclosure
smaller rather than larger. `.github/workflows/documentation-lint.yml` refuses a
path into this tree that does not resolve, an issue reference that names nothing
on this tracker, a required check run name no workflow job emits, and a word the
vocabulary rejected. None of those is a limit that got gentler. A sentence here
softened tomorrow passes all four legs.

It asks for consistency with the warranty disclaimer once #100 lands. THIS
PARAGRAPH SAID #100 WAS OPEN AND THAT THE ANALYSIS IT OWES HAD NOT LANDED. The
issue is open and the analysis landed the morning after this was written:

    git log --format='%h %ad %s' --date=iso -1 -- docs/decisions/0100-the-licence-split-and-the-suite.md
    850c577 2026-08-09 07:05:03 +0200 Write the licence analysis against the suite (#100)

What that changes for this document is less than it looks, and the smaller answer
is the honest one. The analysis makes no warranty statement at all, so there is
no second thing for this document to be consistent with:

    git show origin/main:docs/decisions/0100-the-licence-split-and-the-suite.md | grep -ciE 'warrant|liabilit|disclaim'
    0

It is a compatibility reading of licence texts against the shape of the
artefacts, which is a different question from what this project promises about
what its software does. So the licence half of that condition is discharged by
the licence text in the section above and by nothing else, and the condition's
own wording, which expects the disclaimer to come from #100, does not describe
where the disclaimer actually is.

What the analysis does carry is a limit of its own, and it travels with anything
that leans on it:

    git show origin/main:docs/decisions/0100-the-licence-split-and-the-suite.md | sed -n '13,16p'
    This is an engineering reading of licence texts against the shape of the
    artefacts, made from the sources it quotes. It is not legal advice and nobody
    qualified to give any has read it. Where it depends on something it did not
    measure, the dependency is in the sentence rather than in a closing paragraph.

Nothing in this document leans on it. That is stated so a later reader does not
take the analysis for a legal opinion standing behind the limits above, which is
the direction this document is written against.

The other half of the condition has not moved. #79's warning does not exist to
be checked against:

    gh api repos/iderex/reissbrett/issues/79 --jq '"#\(.number) \(.state) \(.title)"'
    #79 open Write the operator's warning about running a produced program
    git ls-tree -r --name-only origin/main -- docs/ | grep -c 'warning'
    0
