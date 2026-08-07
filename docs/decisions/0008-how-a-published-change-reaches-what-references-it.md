# How a published change reaches the documents that reference it

A bracket is used in an assembly. Somebody publishes a new version of the
bracket. This document says what happens to the assembly, when, and who is
told.

The study in `docs/decisions/prior-collaboration-attempt.md` records what the
earlier effort did here, and this document argues against that record.

## The decision

A reference is pinned to a named version. It never moves on its own. When a
new version of the referenced document is published, the holder of the
referencing document is notified, and the notification carries the result of a
check that says what accepting the new version would do to their document.
Accepting is an action a person takes.

The other two behaviours from #8 are not available. There is no setting that
makes a reference follow the newest published version, and there is no pinning
without the check. Both are named here so that leaving them out is visibly a
decision rather than an omission.

## Why the automatic behaviour is not offered at all

Making it a setting would be a way of not deciding. The argument against it is
that it breaks the one thing this project is for: somebody sits down at a
project and it is the project they left. An assembly that changed under its
author between two sittings, because a part it uses was republished, is the
failure this whole plan is written against, and offering it as a preference
means it happens to whoever leaves the default alone in an organisation where
somebody else set it.

The cost of refusing it is real and is stated rather than hidden. Nothing
pulls an update through automatically, so a project can quietly end up made of
parts that are months behind. That cost is what the check and the staleness
reporting below exist to make visible, and it is a cost this project accepts.

## Why pinning without the check is not offered either

Pinning alone is what the earlier effort had. The study records two positions
on a share link, following the current version or pinned to one, with pinned
as the default, and records that nothing computed whether an update would
break the thing referencing it. The choice was made once by whoever made the
link and the person downstream lived with it.

Pinning alone turns every notification into a decision made without
information. The person is asked whether to accept a change to somebody else's
part, and the only way to find out is to accept it and look. Most people will
not, which is how drift happens. The check is what makes the notification
worth reading, and it is what #55 delivers.

## What counts as breaking a referencing document

This is the part #55 has to implement and #56 has to test, so it is written as
three classes with a rule for each rather than as a sentiment. All three are
evaluated by rebuilding the referencing document against the new version of
the referenced one, through the runner from #30, without a person and without
an interface.

Class one, the reference does not resolve. The new version no longer carries
an object the referencing document names. This is breaking, without
qualification, and it is the cheapest of the three to detect.

Class two, the document does not recompute. The reference resolves and the
referencing document fails to rebuild, or rebuilds with an error reported by
the suite on any object. This is breaking, without qualification.

Class three, the document recomputes and the result is wrong in a way that
recomputing cannot see. This is the class #8 names as the hard one, and it is
the reason the check is worth building at all. The rule: after a successful
recompute against the new version, this project computes a stated set of
measures over the referencing document and compares each against the same
measure taken against the pinned version. A measure that moves outside its
declared tolerance is reported as a change, and one measure is reported as
breaking rather than as a change.

The measure that is breaking is interference. Solid parts of an assembly that
did not intersect before and intersect after are a broken assembly, and unlike
everything else in class three this does not depend on anybody's intent. The
others are reported and not judged: the overall bounding box of the assembly,
its total volume, and the count of constraints the suite reports as unsolved
or redundant.

The distinction matters and is easy to lose. A part getting larger is usually
the point of the change, so a bounding box that moved is information rather
than a verdict. Two parts now occupying the same space is not something
anybody intended.

What this check does not do, stated so that #55 is not asked for it and #56
does not test for it. It does not decide whether the new geometry is correct.
It does not track a face, an edge or a vertex across the change, because #39
sets out why identity inside a document cannot be relied on and #42 is
required to say it cannot tell rather than to guess. It does not certify that
accepting is safe. It reports three classes of finding and the person decides.

## What the reference graph has to hold, and where it is built

The graph is built when a version is recorded, by #41, and it is held on the
server rather than only inside the document body. A server that has to open
every document to answer what uses this part cannot notify anybody in
reasonable time, and one that answers from an index built at record time can.

Per recorded version, the graph holds the outgoing references of that version:
for each one, the referenced document, the exact version of it that is pinned,
and the path or identifier inside the referencing document that names it. The
last of those is what makes a class one finding say which reference failed
rather than that one did.

The incoming direction is derived from the outgoing edges rather than stored
twice, and what publishing a version queries is that derivation: which
versions of which documents pin any version of the document just published.

Two requirements this places on #41, both of which it has to carry.

The references are extracted at record time and stored beside the version, not
recomputed by opening the document when somebody asks.

A reference is recorded against a version and not against a document, because
the whole point of pinning is that the edge names which version. An edge that
loses the version cannot answer whether anything needs to change.

The study records an inference worth carrying into #41 rather than a fact. In
the earlier effort the graph that existed ran between a file version and the
share links pinned to it, rather than between documents, and what uses this
part where a part is used inside another document was not answered by those
schemas. The study says plainly that this was read out of the schemas and
stated nowhere, and that #41 should confirm it rather than build on it. This
document inherits that limit: nothing here is designed against the earlier
graph, only against the observation that a link-shaped graph does not answer a
document-shaped question.

## The document whose author never responds

Nothing happens, and that is the decision rather than an oversight.

The pin holds. The referencing document keeps working against the version it
names, for as long as that version exists, and a published version is never
removed automatically, which is what #44 owes and what makes this answer
possible at all. There is no expiry, no grace period after which the update is
applied, and no state in which a reference silently becomes something else.

What accumulates instead is visible. An outstanding notification stays
outstanding and is shown against the referencing document. Beside it, two
numbers a person can act on: how many published versions of the referenced
document the pin is behind, and how old the pinned version is. Those are
reported wherever the referencing document is listed, so that a project made
of old parts looks like a project made of old parts rather than looking fine.

An operator can see the same thing across a project, so that a person who has
left an organisation does not leave a set of documents nobody knows are stale.
Seeing is all it is. Nothing acts on the numbers, and no permission this
project grants lets one person accept an update into somebody else's document,
because that would be the automatic behaviour again with a different actor.

## What this imposes on other issues

#41 builds the graph described above, at record time, keyed by version.

#55 implements the three classes and returns them as a result rather than as a
verdict, and reports interference as breaking and the rest as change.

#56 tests the classes, including the case that recomputes cleanly and
interferes.

#54 is the action a person takes on the notification, and it either accepts a
named new version or refuses it and says which class of finding it refused on.

#84 exports the staleness numbers so an operator sees them without opening a
project.

## Revisiting this

Two conditions, both checkable.

If #55 finds that class three cannot be computed at acceptable cost on a
realistic assembly, the notification still ships and it carries classes one
and two with the absence of class three stated in the notification itself.
What is not available is a notification that reports no finding while class
three was never evaluated.

If #7 decides a collaboration model in which a referencing document can be
edited by somebody who does not hold it, the paragraph above about nobody
accepting an update into somebody else's document is re-argued against that
decision rather than carried over.
