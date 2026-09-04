# The collaboration model for the first release

This is the decision behind issue #7. Two people work on one project. This
document says what happens.

It is the decision the premise of this project rests on, so it is argued here
rather than inherited from what #7 proposed, and the two models it does not take
are rejected by name with what would have to change for each rejection to be
looked at again.

## The decision

Pessimistic locking at the document level, for the first release.

A person takes a lock on a document before editing it, and nobody else may write
that document until the lock is released. A request for a lock that is already
held is refused, and the refusal names who holds it and since when. Underneath
the lock sits the version store from M3. Beside it sits the review step in #57,
which is optional per project and off by default.

The failure mode of this model is a person waiting. That is the property the
decision is made on, and every argument below is a way of restating it.

## The granularity, and why it is the document

The lock is on a whole document. Not on a body, a part or a feature inside one.

Three things point at the same answer and they are worth separating, because
only the first is a matter of taste.

The identity work in #39 already fixes it. Its own body states the level this
model locks at:

    gh issue view 39 --repo iderex/reissbrett --json body --jq '.body' | grep -n 'locks at for the first release'
    18:construction. This is what #7 locks at for the first release.

#39 places document identity at the path or an assigned identifier and calls it
stable by construction. The level below that, the object in the feature tree, is
something #39 has to establish by experiment on the pinned version, and the
experiment has not been run. There is nothing yet to lock at below the document
that anybody has shown to survive a rename, a reorder, the deletion of a
neighbour and a reopen.

The unit the version store records is the same unit. #6's decision records a
version as the whole container:

    git grep -n 'A version is a list of names' origin/main -- docs/decisions/0006-what-a-version-of-a-model-is.md
    origin/main:docs/decisions/0006-what-a-version-of-a-model-is.md:13:A version is a list of names of stored parts, plus a derived readable

A lock finer than the thing the store records would be a claim on something that
cannot be published on its own, so the person holding it would still have to
coordinate with whoever holds the rest of the document before anything reaches
anybody else. That is the coarse lock with extra steps.

The committed path in #9 works on one part at a time and has no assembly in it.
For the first release the lock unit and the unit of work are therefore the same
thing, which is the case where document level locking costs least. That stops
being true the moment assemblies arrive, and it is one of the reasons the cost
section below is not written as though this were free.

A lock per body or per part is what people will ask for. What has to exist
before it can be given to them is #39's object level experiment coming back with
an identity that survives all four of its operations. Until then, offering it
would mean locking against a name that moves, and a lock that silently transfers
to a different object is worse than no lock.

## Why locking rather than the other two

Locking is the only one of the three whose failure mode is a person waiting. The
other two can produce a document that loads and whose shape is wrong, and a
wrong part that looks finished is the outcome this project can least afford,
because the committed path in #9 ends at a machine program.

It is buildable on the extension surface. The decision about where this layer
attaches is #4's, and it proposes staying on the suite's documented extension
surfaces. Locking needs to know when a document is opened for editing and when
it is saved, which is a much smaller demand on #33's list than either
alternative makes.

It matches how the audience already works. Two people cannot machine the same
billet either, and a shop that has used any mechanical data management system
has met a checkout. This is the one place where the incumbent way of working is
an advantage rather than a habit to be undone.

The review step in #57 recovers most of what optimistic editing was wanted for.
The reason people ask for concurrent editing is usually that they want a change
to be seen and accepted rather than that they want two cursors in one document.
#57 delivers the first without the merge.

## What the decision costs

It scales badly as a project grows. The number of people who can work at once is
bounded by the number of documents, and a project with one large document has a
team of one. The escape from that is finer granularity, which is the thing #39
has to make possible first.

It puts the whole weight of the model on #51. Stale lock handling has to be
genuinely good rather than adequate, because a lock held by somebody who has
gone home is the failure everybody will meet and the one that decides whether
the model is tolerated. #51's own body says the same thing and it is not a
coincidence.

It is the model people will complain about first, and the complaint will be
correct in the case it is made about. A person blocked at four in the afternoon
by a colleague who left at two has lost their afternoon, and no amount of
correct design makes that pleasant.

The cost grows with assemblies. One part at a time is what makes this cheap
today, and the day the committed path admits an assembly is the day this
document is looked at again rather than the day somebody works around it.

## Optimistic editing with a merge, and why not

Rejected.

Merging two parametric feature trees is not a text merge. The result of getting
it wrong is not a conflict marker; it is a document that opens, rebuilds and has
the wrong shape, and nothing downstream would notice. #9's path ends at a
machine program, so the wrong shape becomes metal.

The instability that makes the merge hard is open upstream and is not this
project's to fix:

    gh api search/issues -X GET -f q='repo:FreeCAD/FreeCAD is:issue is:open label:"Topic: Toponaming"' --jq '.total_count'
    51

Read on 2026-08-08. The count moves and it is quoted for its size.

There is a second obstacle that belongs to this project rather than to upstream.
#6 records a version as the container's entries stored under names derived from
their bytes. There is no merge operation on that representation, and building
one means knowing what is inside the container and how two of them combine.
Characterising the container is #38's work and it has not been done, so a merge
today would be designed against an assumed layout.

What would have to change for this to be looked at again. All three of these,
not one of them: #38 characterising the container well enough that a merge is
written against a known structure, #39's object level experiment returning an
identity that survives its four operations, and the upstream count above falling
rather than rising. Even then the argument would have to be about what a merge
does when it cannot tell, and the answer would have to be a refusal rather than
a guess.

## Server-owned documents with an operation log, and why not

Rejected, and this is the rejection that depends on a decision this document
does not make.

The model requires intercepting every mutation the suite performs, so that the
log is complete. An incomplete log is not a slightly worse version of this
model; it is a server whose idea of the document diverges from the file on disk
without anybody being told.

That is a demand on the extension surface in #33 which this project cannot make
from where #4 proposes to sit. #4 proposes the documented extension surfaces and
no patched build. An extension is told about the things the suite chooses to
announce, and completeness across every mutation is not something an announcing
interface can promise.

The dependency is stated rather than hidden: entry 3 of #1, whether this project
ever ships a patched build of the suite, is open and is mine to answer. This
rejection is written against #4's proposal, which is what the board has today.
It is not written against a decision that has been made, and if entry 3 is
answered in favour of a patched build then this rejection is one of the things
that answer reopens.

What would have to change for this to be looked at again. Either #33's extension
surface note listing an interface that reports every mutation of a document, in
which case the rejection falls without entry 3 being touched at all, or entry 3
being answered in favour of a patched build, in which case the interception
becomes possible and the question turns into whether it is worth the fork. The
first is the one to watch, because it can happen upstream without anybody here
deciding anything.

## What the earlier attempt did here

The study in #2 is on the mainline and it reached this question directly. There
was no lock at all:

    git grep -n 'last write wins over whole files' origin/main -- docs/decisions/prior-collaboration-attempt.md
    origin/main:docs/decisions/prior-collaboration-attempt.md:277:recommendation: the implemented model was last write wins over whole files, with

The study's section on it is headed "There was no lock" and it establishes, from
the service's own source rather than from anybody's memory, that the vocabulary
of exclusive access does not appear in the backend except as version pinning and
as a restore command. Two people editing one part produced two versions, and the
one that survived as current was whichever was uploaded second.

That is the thing this decision departs from, and the departure is the whole
point. Conflict recorded by the version list rather than prevented means the
person who lost their work finds out afterwards. The study says explicitly that
whether that is acceptable is this issue's decision, and the answer here is that
it is not.

Nothing in the study argues against locking. It records an absence, and an
absence is not evidence that the absent thing was tried and failed.

## The path to something better

The point of writing this down is that the first release should not have to be
thrown away to get past it.

What survives a change of model, in full: the version store and everything in
M3, because it records what happened rather than who was allowed to do it. The
identity work in #39. The permission model in #50, which answers who may write a
document at all and is a different question from who is writing it now. The
audit trail in #58. Publication and notification in #53, acceptance and refusal
in #54, and the check in #55. Accounts and sessions in #49. The review step in
#57, which is the part most likely to survive every model this project ever has.

What would be replaced: the exclusivity rule itself, which is #51, and the
display in #52, which would have something different to say.

The seam is #48. If the protocol contract is written so that the question the
server answers is whether this person may write this document now, rather than
whether this person holds this lock, then the answer can change from a lock to
something else without the client being rewritten. If it is written the other
way, the model is in the wire format and every client carries it. That is the
one design constraint this document places on #48, and it is placed here because
#48 is where it becomes expensive to change.

## The failures this model produces, and what the operator sees

Each one names the M4 issue that delivers what the operator sees. None of them
is delivered by this document.

The document you want is held by somebody else. Before you open it you see that
it is held, by whom, and since when, in the interface rather than in a message
after you tried, which is #52. If you ask for the lock anyway the refusal names
the holder and the time it was taken, which is #51.

The holder has gone home. A lock near the end of its lease looks different from
one that is actively held, so you can tell waiting from blocked, which is #52.
The lease expires on its own and the expiry takes time rather than happening the
instant a connection drops, which is #51.

An administrator takes the lock away from you. You are told, the act is recorded
in the audit trail, and it is shown to you as the person who held it. All three
are #51's fourth condition, with the recording in #58.

Your lease expired while you were working. When you reconnect you are told
plainly what happened and your local work is still there, which is #51 and #56.
The lock governs who may publish, not who may keep their work.

You are offline and want a new lock, or want to publish. Both fail clearly and
the message says what is needed, which is #56.

You were offline, your lock expired, and somebody else published. Your local
work is intact, you are shown the situation, and you decide. That is #56's
fourth condition and it is deliberately the uncomfortable answer rather than the
convenient one.

Two people ask for the same lock at the same instant. One is granted and one is
refused naming the holder, proven under real concurrency rather than by
argument, which is #60.

Your client cannot reach the server, so what the lock display shows is out of
date. The display says that what it shows is not current, which is #52's fourth
condition, because stale information presented as current is worse than nothing.

You publish a version something else references. The referencing side is
notified, which is #53, and either accepts it or refuses it with what broke,
which is #54, with the breakage caught before it lands by #55.

You may not take the lock at all. The permission model per project and per
document refuses it, which is #50.

Your session is no longer valid. That is #49.

Your change should not become the version others receive yet. It is published
for review instead, and a refusal comes back with the reason attached to the
change itself rather than underneath it, which is #57.

The connection between your client and the server is the surface all of the
above travels over, and #116 is what protects it. #59 holds the edge against a
client that asks for too much.

## What this document does not decide

Entry 6 of #1, whether federation between operators is ever offered, is open and
is mine to answer. This model assumes a project lives on one operator's server.
Nothing above forecloses federation, and #48 is where it would either be made
possible later or made expensive.

Entry 3 of #1, whether this project ever ships a patched build, is open and is
named above as the thing that would reopen one of the two rejections. This
document does not answer it and does not assume an answer to it.

What holds the lock in each state of a review is #57's, and its fourth condition
already requires that state machine to be documented. This document says only
that the question has to be answered rather than left implicit.

The lease duration is #51's, and #51 requires it to be configurable and
documented. No number is set here.

## What builds this

#48 writes the protocol contract and carries the constraint named above. #49,
#50 and #59 are the edge. #51 is the lock and #52 is what a person sees before
they start. #53, #54 and #55 carry a published change to what references it. #56
is offline and reconnection. #57 is review. #58 is the audit trail. #60 proves
the server under concurrent editors and #116 protects the connection.
