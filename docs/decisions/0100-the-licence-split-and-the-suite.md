# The licence split, and what it means against the suite

This is the analysis behind issue #100, and it is where the answer to entry 2 of
#1 lands. That entry asked whether the extensions carry the same licence as the
server. The answer taken on 2026-08-08 is a split: the server under AGPL-3.0, the
extensions under LGPL-2.1, which is the licence of the process they load into.

The analysis is not what produced that answer, and it is written afterwards
rather than before. What it is for is the moment somebody asks whether the
combination the delivery form ships is one this project is allowed to ship, and
the answer has to be something other than an assurance.

This is an engineering reading of licence texts against the shape of the
artefacts, made from the sources it quotes. It is not legal advice and nobody
qualified to give any has read it. Where it depends on something it did not
measure, the dependency is in the sentence rather than in a closing paragraph.

## What is being combined

The delivery form is decided in `docs/decisions/0011-the-delivery-form.md` and is
not restated here. Three routes come out of it and they are three different acts.

The bundle is one download carrying the CAD suite, this project's extensions, the
settings, the theme and the first run experience. Shipping it is redistribution
of somebody else's application together with this project's own code.

The extensions route publishes the extensions into an installation the operator
already has. Nothing of the suite is redistributed on that route.

The collaboration server is a container image, delivered separately, and it is
not in the bundle.

The process boundary in `docs/decisions/0037-the-process-boundary.md` decides
that the process listening on a socket never loads the suite, and that the first
release may need no worker at all. So on the first release there is no artefact
of this project that both terminates network connections and loads the suite into
its own process.

## What the suite's licence actually says

The platform reports one string for the whole repository, and that string is the
detection of a licence file rather than a statement of the terms each file is
offered under:

    gh api repos/FreeCAD/FreeCAD --jq '{default_branch, license: .license.spdx_id}'
    {"default_branch":"main","license":"LGPL-2.1"}

The root licence file at the pinned version is the LGPL 2.1 text itself, and that
text carries no "or later" disposition of its own, because the disposition is
something each file states:

    gh api "repos/FreeCAD/FreeCAD/contents/LICENSE?ref=1.1.3" --jq '.content' | base64 -d | sed -n '1,2p'
                      GNU LESSER GENERAL PUBLIC LICENSE
                           Version 2.1, February 1999

The file headers are where it is stated, and there are two shapes of them. The
older one names the Library General Public License, which is LGPL 2.0, and offers
any later version:

    gh api "repos/FreeCAD/FreeCAD/contents/src/App/Document.cpp?ref=1.1.3" --jq '.content' | base64 -d | sed -n '6,9p'
     *   This library is free software; you can redistribute it and/or         *
     *   modify it under the terms of the GNU Library General Public           *
     *   License as published by the Free Software Foundation; either          *
     *   version 2 of the License, or (at your option) any later version.      *

The newer one carries a machine-readable identifier that says the same thing
about version 2.1:

    gh api "repos/FreeCAD/FreeCAD/contents/src/Mod/Sketcher/App/SketchObject.cpp?ref=1.1.3" --jq '.content' | base64 -d | sed -n '1p'
    // SPDX-License-Identifier: LGPL-2.1-or-later

Five files were read at `ref=1.1.3` and every one of them carried an "or later"
permission. Two are in the older prose, `src/App/Document.cpp` and
`src/Gui/Application.cpp`. Three carry the identifier,
`src/Mod/Part/App/TopoShape.cpp`, `src/Base/Persistence.cpp` and
`src/Mod/Sketcher/App/SketchObject.cpp`. Five is a sample and not a census, and it
is written as one.

The counts are larger and they are weaker evidence rather than stronger, because
the code search endpoint reads the default branch and not the tag this project
pins:

    gh api search/code -X GET -f q='repo:FreeCAD/FreeCAD "SPDX-License-Identifier: LGPL-2.1-or-later"' --jq '.total_count'
    4688
    gh api search/code -X GET -f q='repo:FreeCAD/FreeCAD "at your option) any later version"' --jq '.total_count'
    5040

So the two shapes together reach thousands of files on `main`, and what has been
established at the pinned version is five files. Whether every file at 1.1.3
carries an "or later" permission is not established here, and the paragraph below
says what turns on it.

## Why the "or later" permission is the fact this turns on

LGPL-2.1 without "or later" and AGPL-3.0 do not combine into one work. If the
suite were offered under version 2.1 only, then any single work made of this
project's AGPL-3.0 code and the suite's code could not be conveyed by anybody,
and the split would not be a preference but the only shape available.

With "or later" present, the suite's code may be taken under LGPL-3.0, and
LGPL-3.0 is the GNU GPL version 3 with additional permissions, which is what
makes a combined work conveyable under GPL-3.0 and under AGPL-3.0. The route
exists, it is one-directional, and it is not needed on any of the three routes
below.

This is why the measurement above matters more than the platform's one-word
answer, and why the fifth file mattering is not pedantry. A reader who takes
`LGPL-2.1` from the API and stops has the version right and the disposition
missing, and the disposition is the half that decides whether a combination is
possible at all.

## The three routes, one at a time

The bundle. It carries the suite, unmodified, alongside this project's
extensions. The extensions are under LGPL-2.1, which is the licence of the
process they load into, so there is no combination of AGPL-3.0 code with the
suite anywhere in the bundle: the AGPL-3.0 artefact is the server, and the server
is not in it. What the bundle owes is the obligations of redistribution, which
`docs/decisions/0011-the-delivery-form.md` already names and #101 carries: the
licence text, the copyright notices, and the corresponding source on the terms
the suite's licence sets. Those are owed whether or not anything is combined.

The extensions route. Nothing of the suite is redistributed, so the
redistribution obligations do not arise on this route at all. What loads into the
CAD process is the same LGPL-2.1 code as in the bundle, so the question the split
removes stays removed.

The server container. It carries this project's AGPL-3.0 server and no part of
the suite, which is the process boundary decision rather than a licensing
convenience. AGPL-3.0's network clause reaches it, and that is the intended
effect: an operator running the server for others is offering it over a network.
Nothing here narrows that.

## The boundary, and what actually keeps it correct

The boundary is not a directory. It is the rule that no source file is used on
both sides of it unless it is licensed for both.

A file under AGPL-3.0 that an extension imports puts AGPL-3.0 code inside the CAD
process, and that is a combination with LGPL-2.1 code which the "or later"
permission would have to be relied on to conduct, in the one place this project
had decided not to need it. Shared code is therefore under LGPL-2.1 or it is not
shared, and copying rather than importing is not a way around it.

The worker in `docs/decisions/0037-the-process-boundary.md` is the case this
boundary has to be watched at, and it is named here rather than left to be found.
That document decides the first release may need no worker at all, so today there
is nothing to place. A worker that loads the suite is on the extension side of
this boundary whatever else it belongs to, and the issue that introduces one
answers which licence it carries rather than inheriting the server's by proximity.

## What this analysis depends on

That the suite's files at 1.1.3 carry an "or later" permission. Five were read
and thousands were counted on a different branch, and the gap between those two
statements is not closed here.

That the extensions load into the CAD process and are distributed as source,
which is what a Python extension surface makes them. Nothing here is written for
a compiled extension, and one would need this reread.

That the server holds no part of the suite, which is the process boundary
decision and not an observation, because there is no code yet to observe.

That no file is shared across the boundary, which nothing checks. There are no
source files, so there is nothing to check yet, and the check that would catch it
is not written.

## What this does not settle

Where the two sides live in the tree. That is the layout in #15, and the boundary
above is stated over artefacts rather than over paths precisely so it does not
decide it. A path-level expression of the same rule is what a check could refuse,
and it is available once the layout is.

The header text each side of the boundary expects, and the refusal of a source
file that lacks it. That is the first condition of #100 and it is not discharged
here. What has been established while writing this is that the refusal cannot be
built out of the machinery `.github/invariants.txt` carries today, which refuses a
file for matching a pattern. A missing header is an absence, and no record in
that file can express one.

Which file carries the second licence text, and in what form it is declared. That
is a layout question and an adoption question, and both wait on the paragraph
above.

The third party notices the bundle owes, which are #101, and the licence column
of the bill of materials in #22, which reads them.
