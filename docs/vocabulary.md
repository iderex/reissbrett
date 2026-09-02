# The vocabulary of the committed path

This is the document behind issue #63. It fixes one term per concept on the
committed path decided in #9, so that the guided path, the messages, the
documentation, the sample project and the change summaries call the same thing
the same thing.

The disease being treated is two words for one thing. A beginner who meets one
word in the guided path, a second in a message and a third in a menu is paying
attention to the words instead of to the part, and that cost is what element 2
of the bar in #13 counts.

## What this document does not yet satisfy

Three of the six conditions on #63 are met here and three are not, and saying
which is the point of putting this section first rather than last.

Met. The concepts on the committed path are listed with one term each and the
rejected alternatives carry their reason. The terms the suite fixes are marked
as fixed and used unchanged. The vocabulary is English throughout.

Not met, and none of the three is met by anything written here. #95 does not
check a document against this list, because that check does not exist. Every
message from #64 is not checked against it, because there are no messages. And
the last condition asks that the vocabulary be checked against the externalised
string set from #118 rather than against code, and that string set does not
exist:

    git ls-tree -r --name-only origin/main | grep -cE '\.(go|py|cs|rs|ts|js|cpp|c|java)$'
    0

So this list is a seed rather than a finished vocabulary, and what makes the
seed worth taking now is stated in #63 itself: the words of the committed path
are already on the mainline, and a vocabulary chosen after the fact would have
to repair them.

## How a term is chosen

Three rules, in this order.

**The suite's word wins where the suite owns the word.** The suite's menus are
in front of the person while the guided path is talking to them, and this
project sits on top of those menus rather than replacing them. Where a concept
has a label in the suite at the pinned version, that label is the term, it is
marked FIXED below, and this project does not introduce a better synonym for
it. A better word that disagrees with what is on the screen is the disease.

**Otherwise the workshop word wins.** Where the concept is this project's own,
the term is what somebody in a workshop already calls the thing rather than
what an internal component would be called.

**Where the workshop word and the suite's word differ, both are recorded and
one is used.** A person will meet the other one, and a vocabulary that pretends
otherwise sends them to look for a word that is not there.

## The version everything below is read at

The suite is pinned to FreeCAD 1.1.3 by
`docs/decisions/0005-the-pinned-suite-version.md`, and every suite label quoted
below is read at that tag rather than from a running installation. Nothing in
this tree runs the suite, so every label here is a reading of source and not an
observation of a menu.

## Stage 1. The part exists

| Concept | Term | Source |
| --- | --- | --- |
| The file the suite saves, holding one part | document | this project, and the suite's own word |
| The container inside the document that a part is built in | body | FIXED |
| A recorded state of a document in the store from M3 | version | this project |
| The place versions are recorded | store | this project |

The term for the container is fixed by the suite:

    gh api repos/FreeCAD/FreeCAD/contents/src/Mod/PartDesign/Gui/CommandBody.cpp?ref=1.1.3 --jq '.content' | tr -d '\n' | base64 -d | grep -nE 'QT_TR_NOOP\("New Body"\)'
    91:    sMenuText = QT_TR_NOOP("New Body");

Rejected for `document`: `file`, `model`, `part file`. `file` names the bytes on
disk rather than the thing the person is working on, and stage 1's exit
condition is deliberately about both, so a term naming only one half of it makes
the exit condition ambiguous. `model` is rejected because it is what the person
is making rather than what holds it, and stage 6 records a program against a
model version, where the word has to keep that meaning. `part file` is a
compound where a single word already works.

Rejected for `version`: `revision`, `commit`, `snapshot`. `commit` carries
another tool's mental model into a room where nobody asked for it. `snapshot`
suggests the whole document is stored again each time, which is the opposite of
what `docs/decisions/0006-what-a-version-of-a-model-is.md` decides. `revision`
is the closest and is rejected only because the issues and documents already on
this board say `version`, and changing them costs more than it buys.

## Stage 2. The first profile

| Concept | Term | Source |
| --- | --- | --- |
| The closed outline the person draws | sketch | FIXED |
| A reference plane to draw the sketch on | datum plane | FIXED |
| A relation that removes freedom from the sketch | constraint | FIXED |
| The state in which no freedom is left | fully constrained | FIXED |
| The freedom that is left | degrees of freedom | FIXED |

    gh api repos/FreeCAD/FreeCAD/contents/src/Mod/Sketcher/Gui/Command.cpp?ref=1.1.3 --jq '.content' | tr -d '\n' | base64 -d | grep -nE 'QT_TR_NOOP\("(New Sketch|Edit Sketch|Leave Sketch)"\)'
    155:    sMenuText = QT_TR_NOOP("New Sketch");
    334:    sMenuText = QT_TR_NOOP("Edit Sketch");
    365:    sMenuText = QT_TR_NOOP("Leave Sketch");

    gh api repos/FreeCAD/FreeCAD/contents/src/Mod/Sketcher/Gui/ViewProviderSketch.cpp?ref=1.1.3 --jq '.content' | tr -d '\n' | base64 -d | grep -n 'Fully constrained'
    3730:            QStringLiteral("fully_constrained"), tr("Fully constrained"), QString(), QString());

    gh api repos/FreeCAD/FreeCAD/contents/src/Mod/Sketcher/Gui/CommandConstraints.cpp?ref=1.1.3 --jq '.content' | tr -d '\n' | base64 -d | grep -nE 'sMenuText = QT_TR_NOOP\("Constrain"\)'
    1654:        sMenuText = QT_TR_NOOP("Constrain");

    gh api repos/FreeCAD/FreeCAD/contents/src/Mod/Sketcher/Gui/TaskSketcherMessages.cpp?ref=1.1.3 --jq '.content' | tr -d '\n' | base64 -d | sed -n '68,69p'
            setLinkTooltip(tr("The sketch has unconstrained elements giving rise to those "
                "Degrees Of Freedom. Click to select these unconstrained elements."));

`constraint` is marked fixed for the word rather than for a label. What the
suite labels is the verb, `Constrain`, and the noun is the form this project
writes; `restriction`, `rule` and `relation` are rejected because each of them
would be a second word for the thing the button makes.

`fully constrained` is the exit condition of stage 2 and it is the suite's own
words in the suite's own order. That is worth stating because it is the term
most likely to be improved by somebody writing a message. `fully dimensioned`,
`locked down` and `solved` all read better in a sentence, and every one of them
sends the person looking for a state the suite does not report under that name.

The plane the sketch sits on is fixed as well, and the reading below also
carries the three stage 3 and stage 4 labels so the file is read once:

    gh api repos/FreeCAD/FreeCAD/contents/src/Mod/PartDesign/Gui/Command.cpp?ref=1.1.3 --jq '.content' | tr -d '\n' | base64 -d | grep -nE 'QT_TR_NOOP\("(Pad|Pocket|Revolve|Datum Plane)"\)'
    190:    sMenuText = QT_TR_NOOP("Datum Plane");
    1227:    sMenuText = QT_TR_NOOP("Pad");
    1256:    sMenuText = QT_TR_NOOP("Pocket");
    1330:    sMenuText = QT_TR_NOOP("Revolve");

## Stage 3. The first solid

| Concept | Term | Source |
| --- | --- | --- |
| The single closed shape the body holds | solid | this project, and the suite's word |
| A step in the body that adds or removes material | feature | FIXED |
| Adding material along a straight path away from the sketch | pad | FIXED |
| Adding material by turning the sketch around an axis | revolve | FIXED |
| Rebuilding the body from its features | recompute | FIXED |

`pad` and `revolve` are read from the listing above. Stage 3 of
`docs/decisions/0009-the-committed-beginner-workflow.md` describes both features
without naming either, deliberately, because it left the naming to this
document. These are the names, and they are the suite's.

`revolve` is the entry most likely to be got wrong from memory. The operation is
commonly written `revolution` in prose about the suite, and the label at the
pinned version is `Revolve`. Anything this project writes uses `revolve`.

Rejected for `recompute`: `rebuild`, `refresh`, `update`. All three are better
English and none of them is what the suite reports. Stage 3's exit condition
turns on the recompute being clean, so a message using another word describes a
state the person cannot look up.

## Stage 4. The refinements

| Concept | Term | Source |
| --- | --- | --- |
| A sketched feature added after the first solid | refinement | this project |
| A flat surface of the solid a sketch can sit on | face | FIXED |
| Removing material along a straight path | pocket | FIXED |

`refinement` is this project's word and has no counterpart in the suite. It
names the repeating part of the path, which the suite does not name at all
because the suite has no path. Rejected: `edit`, which names changing something
that exists rather than adding something; `operation`, which is taken at stage 6
and is one of the collisions recorded below; and `modification`, which is longer
and says no more.

## Stage 5. The drawing

| Concept | Term | Source |
| --- | --- | --- |
| The whole of what stage 5 produces | drawing | this project |
| The sheet it is laid out on | page | FIXED |
| One projection of the model on the page | view | FIXED |
| A measurement written on the page | dimension | FIXED |

    gh api repos/FreeCAD/FreeCAD/contents/src/Mod/TechDraw/Gui/Command.cpp?ref=1.1.3 --jq '.content' | tr -d '\n' | base64 -d | grep -nE 'QT_TR_NOOP\("New (Page|View)"\)'
    108:    sMenuText = QT_TR_NOOP("New Page");
    298:    sMenuText = QT_TR_NOOP("New View");

    gh api repos/FreeCAD/FreeCAD/contents/src/Mod/TechDraw/Gui/CommandCreateDims.cpp?ref=1.1.3 --jq '.content' | tr -d '\n' | base64 -d | sed -n '1370p'
        sMenuText = QT_TR_NOOP("Dimension");

`drawing` and `page` are not synonyms and the distinction is kept on purpose.
The suite has no single label for the whole artefact, so `drawing` is this
project's word for it, and `page` stays the sheet the views and dimensions sit
on. Stage 5's exit condition is about what the page holds, so a document that
used the two words interchangeably would make that condition unreadable.

## Stage 6. The program

| Concept | Term | Source |
| --- | --- | --- |
| The container holding the machine, the stock and the operations | job | FIXED |
| The material the part is cut from | stock | FIXED |
| One cutting step inside the job | operation | FIXED |
| The path the tool follows | toolpath | this project, and the suite's word |
| Turning the job into machine instructions | post process | FIXED |
| The machine instructions themselves | program | this project |

    gh api repos/FreeCAD/FreeCAD/contents/src/Mod/CAM/Path/Main/Gui/JobCmd.py?ref=1.1.3 --jq '.content' | tr -d '\n' | base64 -d | grep -n 'New Job'
    58:            "MenuText": QT_TRANSLATE_NOOP("CAM_Job", "New Job"),

    gh api repos/FreeCAD/FreeCAD/contents/src/Mod/CAM/Path/Post/Command.py?ref=1.1.3 --jq '.content' | tr -d '\n' | base64 -d | grep -n 'CAM_Post", "Post Process"'
    110:            "MenuText": QT_TRANSLATE_NOOP("CAM_Post", "Post Process"),

`job` is the one term here that the path document does not use at all. Stage 6
names a machine profile and a stock definition and never names the thing that
holds them. So `job` is added rather than chosen over another word, and the path
document is not changed for it.

`program` is this project's word for the output, chosen against `G-code`,
`NC file` and `post-processed output`. `G-code` names a dialect and this project
promises none. `NC file` is trade shorthand a beginner does not have.
`post-processed output` describes the pipeline rather than the thing. `program`
is also the word `docs/what-this-project-is-not-for.md` uses for what an
operator verifies before running, and one word across the safety statement and
the guided path is worth more than a more precise word in either.

## The words on the path that carry two meanings

One term per concept is the rule #63 states and the direction #13's bar counts.
The other direction, one concept per term, is not counted by that bar and it is
where the committed path is actually exposed today. Three words are recorded
here rather than repaired, because two of the three are the suite's labels and
this project does not get to rename them.

**profile.** Three meanings on one path, and two of them are already in the path
document:

    git show origin/main:docs/decisions/0009-the-committed-beginner-workflow.md | grep -nE '\bprofiles?\b'
    35:### Stage 2. The first profile
    43:than judged, and a profile with degrees of freedom left is what produces a part
    53:Exit: the body holds exactly one solid, the recompute is clean, and the profile
    57:straight path away from the profile, and one that adds it by turning the profile
    108:Entry: stage 4's exit, a machine profile from #73, and a stock definition.

Line 108 is a machine profile, which is the description of a machine from #73.
Lines 35 to 57 are the sketch. The third is the suite's, and it is a cutting
operation:

    gh api repos/FreeCAD/FreeCAD/contents/src/Mod/CAM/Path/Op/Gui/Profile.py?ref=1.1.3 --jq '.content' | tr -d '\n' | base64 -d | sed -n '185p'
        QT_TRANSLATE_NOOP("CAM_Profile", "Profile"),

What this vocabulary does about it. The bare word `profile` names none of the
three. Stage 2's outline is a `sketch`, which is the suite's own word for it and
costs nothing to adopt. The machine description from #73 is a `machine profile`
and is never shortened. The cutting step is a `profile operation`, with the
second word always present.

That leaves a disagreement with the path document rather than a repair of it,
and the disagreement is stated rather than closed here. Stage 2 is titled "The
first profile" and its body uses `profile` for the sketch four times. Changing
those words is a change to a landed decision document, and a vocabulary
document is not where a decision is edited. #61 builds the guided path and is
the first artefact that has to say one of the two words out loud, so that is
where the two are made to agree.

**pocket.** A PartDesign feature at stage 4 and a family of CAM operations at
stage 6, both labelled by the suite. There is no repair available and none is
wanted: both are the suite's labels, and this project introducing a synonym for
either would be the failure this document exists against. What this vocabulary
asks is that the word never appear on its own in anything this project writes.
It is a `pocket feature` at stage 4 and a `pocket operation` at stage 6.

**operation.** Used above for a cutting step inside a job, and used by
`docs/decisions/0050-the-permission-model.md` for a request a role is allowed to
make. Those are different rooms and the ambiguity has not bitten anything yet,
so nothing is renamed. It is named here so that a message about permissions and
a message about machining are not written with the same bare noun.

## Lock and write claim are two concepts, not two words for one

They look like the defect this document exists to catch and they are not, so the
reading is recorded rather than left for somebody to find and report twice.

    git show origin/main:docs/collaboration-protocol.md | sed -n '36,38p'
    That is why the word lock does not appear in any message name or field name
    below. What the client asks for is a write claim on a document. Under the model
    in force today a write claim is granted by taking a lock and refused when

`write claim` is what a client asks for on the wire, and it is what the
permission model in `docs/decisions/0050-the-permission-model.md` grants a
contributor. `lock` is the mechanism the server uses to answer, decided in
`docs/decisions/0007-the-collaboration-model.md`. The distinction is deliberate:
a different mechanism would answer the same request and the wire would not
change.

What that means for anything written to a person. The person asks to write a
document and is told whether they may, so the term in the guided path and in
every message is `write claim`. `lock` belongs in documents about how the server
works, and a message telling a person their lock was refused is naming a
mechanism at somebody who asked a different question.

## What is deliberately not in this vocabulary

Everything off the committed path. The suite's other modules keep their own
words unchanged, which is what the escape hatch in #68 promises, and a
vocabulary reaching into them would be a rename of somebody else's product.

The words for what the collaboration model does beyond what stage 1 reaches.
Publishing, review, notification and reference each have a term in
`docs/collaboration-protocol.md` and in `docs/decisions/0050-the-permission-model.md`,
and those documents are the authority for them. They are named here only where
the committed path meets them, which is `version` and `store` at stage 1 and
`write claim` above.

Anything about how good a word is. Whether `refinement` is the right word for
stage 4 is a judgement, and the measurement in #67 with people who have not used
the suite is what would settle it. Until then it is a choice, not a finding.

## The words this check refuses

This section is data. `.github/workflows/documentation-lint.yml` reads the lines
below out of this file rather than carrying a copy of them, which is #95's third
condition, so a word is added to the refusal by editing this document and
nothing else.

Each line names a word this vocabulary rejected and the term that replaced it. A
tracked document using the left-hand side reds the check, and the message names
the right-hand side.

    refuse: revolution -> revolve
    refuse: NC file -> program
    refuse: post-processed output -> program
    refuse: part file -> document
    refuse: fully dimensioned -> fully constrained
    refuse: locked down -> fully constrained

This file is the one document the check does not scan, because it is the file
that has to name the refused words in order to declare them. That exemption is
in the workflow with the same reason beside it.

## Which rejected words are refusable, and which are not

The list above is shorter than the rejected alternatives named in the tables,
and the difference is the whole of what this refusal can do rather than an
oversight.

A rejected word is refusable only where it carries no other meaning in this
tree. `revolution`, `NC file`, `post-processed output`, `part file`,
`fully dimensioned` and `locked down` name the concept they were rejected for
and nothing else, so a pattern that finds one has found the drift.

`file`, `model`, `edit`, `update`, `refresh`, `operation`, `rebuild` and
`modification` are rejected words and ordinary English in the same tree. A
pattern refusing them would refuse honest sentences on every page, and a check
whoever meets it turns off is worse than no check.

`G-code` is the entry to read before adding a seventh line, because it looks
refusable and is not:

    git grep -niE '\bg-?code\b' -- docs/decisions/0012-where-the-nearest-projects-fall-short.md
    docs/decisions/0012-where-the-nearest-projects-fall-short.md:156:machine interprets called Gcode."
    docs/decisions/0012-where-the-nearest-projects-fall-short.md:294:  ships sketch to G-code in one product. What is missing is a guided path, not

Both are correct text. One is inside a quotation from another product's own
documentation and the other describes what that product ships. The rejection in
the stage 6 table is about what this project calls its own output, and no
pattern separates the two, so the word stays rejected and unrefused.

`snapshot` is the same case one step further out. It is rejected as a term for a
version, and #85's backup and restore work will legitimately use it for a
filesystem or volume snapshot, which is a different thing under the same word.

## What refuses any of this today

The six words above, and nothing else. `PROSE, NOT ENFORCEMENT` for the rest,
issue #95 for the documentation half and #64 for the messages half.

A term used in a document and absent from this vocabulary is refused by nothing,
because the check refuses a declared word rather than admitting a list of
allowed ones. A message using another word is refused by nothing, because there
are no messages.

`.github/invariants.txt` names the vocabulary rule among the rules it does not
yet carry, and it is untouched by the check above. That is deliberate and it is
worth the sentence, because a record there is the cheaper-looking route. A
record in that file carries its own pattern, so the refused words would be
written twice, once in this document and once beside the machinery. #95's third
condition refuses exactly that: the check reads this document rather than a
copied list. So the refusal lives where it can read this file, and the record
that file is waiting for is a pattern rule of a shape this list is not.
