# The single beginner workflow the first release commits to

This is the decision behind issue #9. The readme claims a coherent workflow, and
coherent means something narrow here: this project has one recommended way to
make a part, it says so, and the interface in M5 is built around that one way
rather than around a menu of equivalents.

The path is written in a form the rest of M5 can be built against. Six stages,
in order, each with an entry condition and an exit condition, and one place a
person is told where they are.

## The committed path

One part, from an empty document to a program a machine could run. One part at a
time, and no assembly.

The names of the things in each stage are provisional. #63 chooses the words
this project uses and records where the workshop word and the suite's word
differ, so the stages below describe what happens rather than fixing the
terminology.

### Stage 1. The part exists

Entry: the workspace is open in the state defined by #62, and no document is
open.

Exit: a saved document holding one body and nothing else, recorded as the first
version in the store from M3.

What finishes this stage is a file on disk with a version behind it, not a
decision the person makes. That is deliberate: the first thing a beginner should
have is something they cannot lose, and putting the version store at the front
of the path means every later stage has somewhere to go back to.

### Stage 2. The first profile

Entry: a saved document with one body and no solid in it.

Exit: one closed sketch on one of the body's datum planes, fully constrained.

Fully constrained is the exit condition rather than an encouragement. The suite
reports the degrees of freedom that are left, so the condition is read rather
than judged, and a profile with degrees of freedom left is what produces a part
whose dimensions move later for reasons nobody can find.

A datum plane rather than a face, at this stage, because there is no solid yet
to have a face.

### Stage 3. The first solid

Entry: stage 2's exit.

Exit: the body holds exactly one solid, the recompute is clean, and the profile
from stage 2 has been consumed by the feature that made the solid.

This stage offers two features and no others: one that adds material along a
straight path away from the profile, and one that adds it by turning the profile
around an axis. Nearly every part a beginner wants starts as one of those two,
and the point of the path is that the person does not spend their attention
choosing between six things that would all work.

Anything else that could have made the first solid is off the path. It is
reachable, and the section on exclusions below says where.

### Stage 4. The refinements

Entry: a body with exactly one solid.

Exit: every sketch in the body is fully constrained, the recompute is clean, the
body still holds exactly one solid, and the person has said they are finished.

A refinement is a sketch on a face of the solid or on a datum plane, turned into
material added or removed. This is the only stage in the path that repeats, and
the repetition is explicit in the interface rather than left as a person
wondering whether they are still allowed to add things.

The exit needs the person's word because no property of the document says a
shape is finished. That is stated here so that nobody later reads the exit
condition as something the software decided. The other three parts of it are
machine readable and are what the guided path enforces.

Two of those three are worth naming for what they refuse. Exactly one solid
refuses the part that has quietly split in two, which is a state the suite will
carry along without complaint and which breaks stage 6. A clean recompute
refuses the document that still opens but no longer rebuilds, which is the state
#65 exists to make survivable.

### Stage 5. The drawing

Entry: stage 4's exit.

Exit: a drawing page holding the views and the dimensions somebody else would
need to make the part, with no view carrying a reference into the model that no
longer resolves.

The unresolved reference half is machine readable, and #72 is what keeps it true
when the model changes after the drawing exists. Whether the dimensions on the
page are the right ones is a judgement about the part, and this project does not
check it and does not pretend to.

The drawing is not an input to stage 6. It is in the path because a drawing is
what somebody else reads, and this project is about more than one person working
alone. A reader who expects stage 5 to be optional should read that sentence as
the answer rather than looking for a technical reason.

### Stage 6. The program

Entry: stage 4's exit, a machine profile from #73, and a stock definition.

Exit: a post-processed program for the machine class chosen in entry 5 of #1,
refused by #75 if the toolpath would collide with the stock, and recorded
against the model version it was produced from by #77.

The machine class is not chosen here. Entry 5 of #1 is open, it belongs to the
maintainer, and stage 6 is written so that its exit condition is complete for
whichever class is chosen. #10 stages the machining work and #74 pins the post
processor once there is a class to pin one for.

The refusal in #75 is part of the exit condition rather than a later safety
check bolted on. A program that would collide is not a finished stage 6 with a
warning attached; it is an unfinished stage 6.

## Where the person is told where they are

One place, naming the stage, what it is for, what finishes it, and what comes
next. #61 builds it.

There is exactly one such place, and that is the rule rather than a design
preference. Two indicators that can disagree is the incoherence this project
exists to treat, and the cheapest way to produce that state is to add a second
one later because the first was in the wrong corner.

## What the path excludes, and where the excluded thing lives

The suite ships thirty three module directories at the version pinned in #5:

    gh api repos/FreeCAD/FreeCAD/contents/src/Mod?ref=1.1.3 --jq '[.[] | select(.type=="dir") | .name] | join(" ")'
    Assembly BIM CAM Cloud Draft Fem Help Idf Import Inspection JtReader Material Measure Mesh MeshPart OpenSCAD Part PartDesign Plot Points ReverseEngineering Robot Sandbox Show Sketcher Spreadsheet Start Surface TechDraw TemplatePyMod Test Tux Web

Four of those carry the committed path. Sketcher is stage 2, PartDesign is
stages 3 and 4, TechDraw is stage 5 and CAM is stage 6. Start is behind what
appears before a document is open, which is #62's business rather than the
path's.

Every other entry in that listing is excluded from the guided path and reachable
through the escape hatch in #68, unchanged and with the suite's own interface
rather than a version of it this project has rewritten.

Nothing is removed. The bucket for capability that is not present at all is
empty, and it is empty for a reason worth stating rather than assuming: the
bundle in #11 is an assembly of the artefact upstream publishes rather than a
build of the suite from source, so what it carries is what upstream carries.
That is a property of an artefact that does not exist yet. #108 is where the
bundle is built, and if it ever drops a module, this section stops being an
accounting of a directory listing and becomes an accounting of a bundle. Whoever
lands #108 owes that check.

The listing is also a coarse instrument, and the same sentence appears in #5's
document for the same reason. A module directory is not a capability. Several of
the names above read as test scaffolding or sample code rather than as something
a person chooses, and a capability that lives inside a module on the path can
still be off the path, which is exactly what stage 3 does when it offers two
features out of a longer list. The finer accounting is the guided path's own, in
#61, which knows what it offers at each stage. The listing is the outer bound
and it is the part that can be checked today.

What leaving the path costs is stated at the moment of leaving rather than in a
document. Work done off the path is not covered by the guidance, by the messages
in #64 or by the measurements in #67. That is #68's fourth condition and this
document does not repeat it as a promise of its own.

## The alternatives this rejects

### A path that starts from an assembly

Rejected. Three reasons, and the first is measurable.

The naming instability that makes references into a model come loose is open
upstream, and assembly is where it bites hardest because an assembly is nothing
but references into other models:

    gh api search/issues -X GET -f q='repo:FreeCAD/FreeCAD is:issue is:open label:"Topic: Toponaming"' --jq '.total_count'
    51

    gh api search/issues -X GET -f q='repo:FreeCAD/FreeCAD is:issue is:open label:"Mod: Assembly"' --jq '.total_count'
    110

Both counts move on their own and were read on 2026-08-08. They are quoted for
their size rather than their exact value.

The second reason is that an assembly doubles the collaboration problem. M4 is
already the hardest milestone on this board, and a first release that has to
answer what happens when two people hold two parts of one assembly is a first
release that answers it badly.

The third is that a beginner who cannot yet make a part reliably has no business
in an assembly, and a guided path that starts there teaches the harder thing
first.

What would have to change for this to be revisited: the two counts above falling
rather than rising, and document level locking from M4 having survived real use
rather than only its own tests. Neither is a date and neither is close.

### Teaching the suite as it is

Rejected, and this is the alternative the project exists against, so it is
stated rather than skipped.

The suite presents several ways to do everything and commits to none of them. A
guided path that presents all of them and explains the differences is a longer
version of the current situation. The person's attention goes into choosing, and
a person who does not yet know what any of the options are cannot choose between
them on information they do not have.

What would have to change for this to be revisited: nothing this document can
name honestly. If this alternative is right, the premise in the readme is wrong,
and the argument to have is about the premise rather than about the path. That
is recorded here so a later reader does not mistake the absence of a revisit
condition for an oversight.

### A path that ends at a model rather than at a program

Rejected, and this is the closest of the three.

Ending at a model would remove stages 5 and 6, which are the two stages that
pull the most into the first release: the machine class in entry 5 of #1, the
collision refusal in #75 and the simulator proof in #76. Dropping them would
make M5 and M6 considerably smaller.

It is rejected because the audience this project is aimed at makes things. A
workflow that ends at a file the person then has to take somewhere else has not
closed the loop, and closing the loop is the claim in the readme.

What would have to change for this to be revisited, stated concretely because
this one might actually happen: if the collision and simulation work in #75 and
#76 cannot be made safe enough to put in front of a beginner in the first
release, the path ends at stage 5 and the claim in the readme is narrowed in the
same landing. Narrowed in the same landing is the condition. A path that quietly
stops at the drawing while the readme still says otherwise is the failure this
sentence exists to prevent.

## What the decision costs

Anyone whose work starts with an assembly, with imported geometry or with a mesh
is outside the committed path on the first release, and the interface will feel
like it was not built for them, because it was not. #68 is what keeps that from
being a wall rather than a slope, and it is why #68 is in M5 rather than in a
later milestone.

A person who wants only a drawing still passes through stages 1 to 4. The path
does not have an entry point in the middle, because an entry point in the middle
is a second way of doing things, which is what committing to one path means
giving up.

Committing also means that a second recommended way added later is a change to
M5 rather than a setting. That cost is paid once, at the moment somebody asks
for the second way, and it is worth knowing before that conversation rather than
during it.

## What this document does not decide

The machine class in entry 5 of #1, which stage 6 names rather than chooses.

The collaboration model, which is #7. The committed path is one part and one
person. What happens when a second person opens the same document is #7's
decision and this document assumes no answer to it.

The interface language, which is #118. The path is a sequence of stages and it
does not depend on the words the stages are named in, but the vocabulary in #63
is checked against #118's string set rather than against this document.

## What builds, teaches and measures this

#61 builds the guided path and enforces the entry and exit conditions above.
#62 delivers the state stage 1 starts from. #63 supplies the words. #64 supplies
what is said when a stage fails. #65 handles the failure that most often ends an
attempt. #66 makes all of it testable without a display. #68 is the way off the
path and back. #69 is a worked example of exactly these six stages, and #67
walks the path end to end against the bar in #13.
