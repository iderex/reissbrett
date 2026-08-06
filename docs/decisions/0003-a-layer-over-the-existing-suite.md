# The shape: a layer over the existing suite

This is the decision behind issue #3. It fixes the shape of the project, and
every other issue on the board rests on it, so it is written down before
anything is built on top of it.

## The decision

reissbrett is a distribution and orchestration layer over the existing open
modular CAD suite, which is FreeCAD. It implements no geometry kernel, no
constraint solver, no mesh pipeline, no drawing generator and no toolpath
generator. Where the workflow needs any of those, it calls the suite.

## What the alternative would cost

The alternative is writing a modeller. The size of that is measurable rather
than a matter of opinion. Every number below was produced by the command
printed above it, and all of them were run on 2026-08-06 while this document
was being written.

    gh api repos/FreeCAD/FreeCAD --jq '{language: .language, stars: .stargazers_count, license: .license.spdx_id}'
    {"language":"C++","license":"LGPL-2.1","stars":32664}

    gh api repos/FreeCAD/FreeCAD/contents/src/Mod?ref=1.1.3 --jq '[.[] | select(.type=="dir")] | length'
    33

Thirty three module directories at the pinned tag, and the ones this project
leans on hardest are the ones that took longest to get right: the sketch
solver, the parametric feature tree, the drawing generator, and the machining
module with its post processors.

    gh api repos/FreeCAD/FreeCAD/contents/src/Mod/CAM/Path/Post/scripts?ref=1.1.3 --jq 'length'
    40

Forty post processors is years of contact with real machines and real
controllers, and no from-scratch effort arrives holding that. It is also the
part of the stack that cannot be reasoned into existence, because what a
particular controller accepts is a fact about that controller.

## The argument

The gap this project is aimed at is a coherent workflow, a stable interface and
team collaboration. None of those three requires owning the kernel.

A coherent workflow is a decision about which of several possible paths the
product commits to, and about what the person is shown at each step. That is
made outside the kernel, by choosing what to expose and in what order.

A stable interface is a decision about what the workspace looks like when it
opens and what it does when the person is finished. That is made outside the
kernel as well, though how much of it is reachable from outside is the separate
question that issue #4 settles and that entry 3 of #1 keeps open at the
maintainer's level.

Collaboration is about who may write which document when, and about how a
published change reaches the people who reference it. It touches the document
format and the process boundary. It does not touch the solver.

A from-scratch modeller would spend its first several years reproducing what
already exists, and it would arrive with the same three gaps still open,
because nothing about writing a kernel closes any of them.

## What the decision costs

This project inherits the suite's release cadence, its defect load and its
interface decisions. The open issue counts on the modules this workflow depends
on, at the time of writing:

    gh api search/issues -X GET -f q='repo:FreeCAD/FreeCAD is:issue is:open label:"Mod: Sketcher"' --jq '.total_count'
    451

    gh api search/issues -X GET -f q='repo:FreeCAD/FreeCAD is:issue is:open label:"Mod: Assembly"' --jq '.total_count'
    110

    gh api search/issues -X GET -f q='repo:FreeCAD/FreeCAD is:issue is:open label:"Topic: Toponaming"' --jq '.total_count'
    51

A defect in any of those is a defect this project's users meet, and it is one
this project cannot fix from where it sits. The route open to it is an upstream
report or an upstream patch, on upstream timelines. That is the price of the
decision, it is not small, and it is the reason the pin in #5 and the
compatibility harness in #34 exist at all.

The second cost is that a bad release upstream is a bad release here. The
project can refuse to run against a version it was not built for, which is what
#35 delivers, and refusing is the best it can do.

## What this project implements, and what it delegates

The purpose of this list is that a later issue can be placed on one side or the
other without an argument about it.

This project implements the committed workflow and the order of its stages, the
interface the person sees while following it, the vocabulary used across the
interface and the documentation and the messages, the document model this
project records, the content addressed version store, the collaboration server
with its locks and its permission model and its audit trail, the machine
profiles and the rules for refusing a job a machine cannot run, the packaging
an operator installs, and the first run.

This project delegates the geometry kernel, the constraint solver, the
parametric feature recomputation, the mesh pipeline, the import and export
filters, the drawing generation, the toolpath generation and the post
processors, and the native document format itself.

Two things sit on the boundary and are named so they are not fought over later.
Toolpath verification against stock, in #75, consumes what the suite generates
and decides whether to refuse it, so the decision is ours and the generation is
not. Drawing correctness under a changing model, in #72, is the same shape: the
drawing is generated by the suite and the check that it still matches is ours.

## What would have to become true to revisit this

The decision is revisited if the suite stops being maintained at a rate this
project can stand on. The checkable form of that is the date of the most recent
non-prerelease release:

    gh api repos/FreeCAD/FreeCAD/releases --jq '[.[] | select(.prerelease == false)][0] | {tag: .tag_name, published: .published_at}'
    {"published":"2026-07-25T04:53:36Z","tag":"1.1.3"}

Eighteen months with no such release, with the issue counts above still rising,
is the condition. It is a signal about the project rather than a proof, and
somebody has to look at what is behind it before acting.

The decision is also revisited if the licence of the suite changes to one this
project cannot build on, which is checkable directly:

    gh api repos/FreeCAD/FreeCAD --jq '.license.spdx_id'
    LGPL-2.1

Neither condition is met today. Nothing else revisits it. In particular, the
number of upstream defects being high is not a reason to write a kernel, since
a new kernel starts with defects nobody has found yet rather than with none.

## A note on the numbers

Three of the numbers above move on their own: the star count, and the three
issue counts. They are quoted with the date they were read and they are
expected to differ on a later run. The module count and the post processor
count are read at a fixed tag and should not move.

One of them did. Issue #3 quotes thirty five module directories from the same
command at the same tag, and the re-run above returns thirty three. The full
listing at that tag holds thirty eight entries, of which thirty three are
directories and five are files.

Thirty five is what the same command returns one minor version earlier, and the
two directories that account for the difference can be named:

    gh api repos/FreeCAD/FreeCAD/contents/src/Mod?ref=1.0.2 --jq '[.[] | select(.type=="dir") | .name] | length'
    35

    diff <(gh api repos/FreeCAD/FreeCAD/contents/src/Mod?ref=1.0.2 --jq '.[] | select(.type=="dir") | .name' | sort) <(gh api repos/FreeCAD/FreeCAD/contents/src/Mod?ref=1.1.3 --jq '.[] | select(.type=="dir") | .name' | sort)
    1d0
    < AddonManager
    7d5
    < Drawing

Thirty five is therefore a real count of a real tree, and it is the count at
1.0.2 rather than at the tag the command in the issue names. How it came to be
written beside a command naming 1.1.3 is not something this document can
establish, so it is not guessed at here. The number in this document is the one
that reproduces at the tag written beside it, and the difference is recorded
rather than quietly corrected, because the same slip made about a version this
project pins itself to would not be harmless.
