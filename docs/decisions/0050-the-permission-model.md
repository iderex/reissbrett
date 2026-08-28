# The permission model, per project and per document

Issue #50. Taken on 2026-08-27.

Who may see a project, who may edit it, who may publish a version, and who may
administer it, answered once so that every operation can be decided in one place.

## Nothing here has been built

There is no server, no client and no decision point, so every sentence below is a
requirement rather than an observation. The tree carries no code in any language:

    git ls-tree -r --name-only origin/main | sed -n 's/.*\.//p' | sort -u | tr '\n' ' '
    gitattributes md txt yml

Nothing in this document measures anything. It states a model and the reasons for
it, and the three conditions of its issue that are properties of code are open
when it lands.

## What decides this from outside

`docs/decisions/0007-the-collaboration-model.md` puts this question in its own
place and says what it is not:

    git grep -n 'answers who may write a' origin/main -- docs/decisions/0007-the-collaboration-model.md
    origin/main:docs/decisions/0007-the-collaboration-model.md:205:identity work in #39. The permission model in #50, which answers who may write a

Who may write a document at all is a different question from who is writing it
now. The second is the write claim, which is #51. This document answers the first
and never the second, and the two refusals are separate codes on the wire for
exactly that reason.

`docs/collaboration-protocol.md` carries the refusal this model answers with and
says where the answer has to come from:

    git grep -n 'from one decision point' origin/main -- docs/collaboration-protocol.md
    origin/main:docs/collaboration-protocol.md:598:#50 answers `not-permitted` from one decision point, and the protocol offers no

`docs/threat-model.md` names what this model does not defend against, and it is
the operator's own administrator. Nothing below narrows that.

## The decision

Three roles per project: viewer, contributor, administrator. A person holds one of
them in a project. A document may narrow a person's role and may never widen it.

### What each role carries

A viewer may read the project's documents, read the version history of a document
they may read, and read the readable change summaries #42 produces for those
versions. A viewer may take no write claim, publish nothing, decide nothing, and
change no membership.

A contributor may do everything a viewer may do, and additionally take a write
claim on a document they may write, publish a version of it, publish it for review
where #57's review is on, decide a review on a change somebody else published for
review, and answer a notification about a reference held by a document they may
write.

An administrator may do everything a contributor may do, and additionally add and
remove members, set and change roles, narrow a role on a document, and revoke a
write claim held by somebody else.

### Which role each operation asks for

The operations are named by what they are. Where the contract has a message for
one, the message is named beside it, and where it has none the absence is stated
rather than a message invented.

| Operation | Least role |
| --- | --- |
| `documents.query`, `versions.query`, `version.query` | viewer |
| `parts.request`, and the transfer toward the client that follows it | viewer |
| `write-state.query` | viewer |
| `write-claim.request`, `write-claim.renew`, `write-claim.release` | contributor |
| `publish`, `publish.for-review` | contributor |
| `review.decide` | contributor |
| `notification.query`, `reference.decide` | contributor |
| `parts.offer`, `transfer.begin`, `transfer.end` toward the server | contributor |
| Add or remove a member, set a role, narrow a role on a document | administrator |
| Revoke somebody else's write claim | administrator |

`hello` and `session.begin` ask for no role. They are #49's and they establish who
is asking, which is what everything above is decided against.

`projects.query` has no row, and the absence is deliberate rather than an
oversight. It answers with the projects the person is a member of and names
nothing else, so what it asks for is membership itself and not a role inside a
project. Every row above is a role held in one project, and giving this operation
one would mean inventing a role above the three this document decides. The
contract states the same limit from its own side: there is no request that names
a project the person has not been told about.

### The read operation had no message to name, and this is what changed

WHAT STOOD HERE RECORDED AN ABSENCE, AND THE ABSENCE IS GONE RATHER THAN
NARROWED. The first row of the table above read "Read a document and its
history" in prose, because the contract declared no request that opened a
document, listed a project's documents, listed a document's versions, or asked
the server to send parts. The transfer messages were specified in both
directions and only the client's direction had a request that started one, and
the refusal code `no-such-version` was declared with nothing in the message set
able to produce it. That was written down here, recorded on #48, and left
unrepaired on purpose, because inventing a message in this document would have
decided the contract in passing.

#48 has since specified them. Read out of the tree this document is in rather
than recalled, from a checkout of the commit that carries both files:

    grep -oE '^`[a-z][a-z.-]+`' docs/collaboration-protocol.md | grep -E '\.query`$' | sort -u
    `documents.query`
    `notification.query`
    `projects.query`
    `version.query`
    `versions.query`
    `write-state.query`

    grep -c 'no-such-version' docs/collaboration-protocol.md
    3

    grep -c '^`parts.request`' docs/collaboration-protocol.md
    2

The first prints the request names the contract introduces at the start of a
line and keeps the ones that ask a question. Four of the six are new and two,
`write-state.query` and `notification.query`, were always there. The second
counts three occurrences of `no-such-version` where there was one: the message
that now refuses with it, the paragraph saying why that matters, and the closed
code set where it previously stood alone. The third finds `parts.request`, which
starts the transfer toward the client that this table's second row governs, at
its specification and at the line naming it for #43.

So the read permission is stated against messages like every other row, and it
is the same permission it was: viewer, decided here, unchanged by the contract
having caught up. What moved is the contract, not this model.

One thing did not move and is worth keeping visible. Nothing refuses any of
this. The rows above are a requirement on a decision point that does not exist,
which the section below on what refuses this says at greater length, and a table
that now names messages is not a table anything checks.

## The inheritance rule

A person's role in a project applies to every document in it. A document may carry
a narrower role for a named person, and the narrower one wins. A document may not
carry a wider one.

Narrowing only, in one direction, is the whole of the rule, and the reason is what
a reader of the project can conclude from it. If a document could widen, a
person's project role would bound nothing, and answering what this person can
reach would mean walking every document in the project rather than reading one
membership list. Narrowing keeps the project role an upper bound, so the expensive
question has a cheap answer that is always right.

Narrowing does not apply to an administrator, and this is stated rather than left
to be discovered. An administrator may set and change what narrows a role, so a
narrowing placed on an administrator is removed by the person it was placed on. A
control somebody can lift is not a control, and writing it into the model as
though it held would be worse than not having it.

## Why this granularity, and not more

Two roles are not enough, and that is not a judgement. `0007` already requires that
somebody can take a lock away from somebody else, be told about it, and have it
recorded:

    git grep -n 'takes the lock away from you' origin/main -- docs/decisions/0007-the-collaboration-model.md
    origin/main:docs/decisions/0007-the-collaboration-model.md:237:An administrator takes the lock away from you. You are told, the act is recorded

A model of readers and writers has nobody who may do that, so the third role is
required by a decision that is already landed rather than chosen for symmetry.

Four further shapes were considered and refused, one at a time.

A separate publisher, who may write but not make a version others receive, is what
#57's review already is. Whether a change has to be reviewed before it becomes the
version others receive is a property of the project or the document rather than of
the person, and putting it in a role would give a project two ways to express one
thing that could then disagree. The model is written so that #57 can arrive
without a role being added.

A separate owner above administrator buys the answer to one edge case, the last
administrator, and that case is answered below without it. An owner who is also
the only person able to transfer ownership recreates the same edge case one level
up.

A per-operation permission set, with roles as a convenience over it, is the shape
that scales, and it is refused for this release. Every level of granularity is a
level somebody setting up a project has to understand, and there is no evidence
here that anybody needs one: nothing has been built and no operator has asked. It
is also the change that is cheapest to make later, because a role can be redefined
as a set of permissions without any of the three names above moving.

Guest access, somebody who can see one document without being a member, is refused
because it needs an identity for a person who has no account, and that is #49's
question rather than a role.

## The four edge cases, answered

The issue names four. Each has an answer here, and each still owes a test, which
is that issue's fourth condition and is code.

**What a person who has left can still see.** Nothing. Removal from the project
removes the role, and with no role there is no operation. Their name stays in the
version history and in the audit trail #58 keeps, because those record what
happened, and removing a member is not a claim that they never wrote anything.
What removal cannot do is retract what is already on their machine. A version they
restored before they left is a file they hold, and this project cannot un-copy it.
That is a limit and it is stated as one rather than narrowed until it sounds like
a guarantee.

**Whether a write claim can be broken, by whom, and what the holder is told.** It
can, by an administrator, and by nobody else. The holder is told, and the wire
carries which of the two reasons it was:

    git grep -n 'or an administrator took it' origin/main -- docs/collaboration-protocol.md
    origin/main:docs/collaboration-protocol.md:335:expired or an administrator took it. Carries which of those it was, and who did

The act is recorded in the audit trail, which is #58, and the holder's local work
is untouched, because the claim governs who may publish rather than who may keep
what they have written. A contributor may not break another contributor's claim.
The claim exists so that two people do not publish over each other, and a claim
anybody may break is not one.

**Whether history is visible to somebody who cannot edit.** It is. A viewer sees
the version history of every document they may read, and the readable change
summaries with it. History follows the document: where a document is not readable,
neither is its history, and there is no state in which a person can see that a
document changed without being able to see the document. Hiding history from a
reader would leave them the current state with no way to tell what moved, which is
the failure #42 exists against, and it would make a viewer useless as a reviewer.

**What happens to a project whose only administrator is gone.** Two halves, and
they are different problems.

The ordinary route refuses to create the case. Removing the last administrator of
a project, or changing their role to anything else, is refused. So the case is
never reached by somebody working through the interface, and it is reached only by
the account itself going away, which is #49's account lifecycle rather than this
model's.

What is left is reached by the operator, from outside the project. Somebody who
runs the server can appoint an administrator for a project that has none. This is
not a role in the model and it is not an in-project recovery: a project cannot be
made permanently unreachable because one account was disabled, and pretending that
an operator with database access could not do it anyway would be the narrowing
`docs/threat-model.md` already refuses under N1. The act is recorded in the audit
trail as an operator action and is distinguishable there from a promotion made
inside the project, because those two are not the same event and a trail that
shows them as one hides the interesting one.

Which route the operator uses is not decided here. It is an operator surface and
it belongs with the rest of them, in #80 and #110.

## One decision point

Every operation passes through one function that is given who is asking, what they
are asking to do, and which project or document it is about, and answers permitted
or not. There is no second route.

The reason is the failure the issue opens with. A permission check written into
each handler is a permission check that is missing from one of them, and the
missing one is found by somebody who should not have been able to do the thing
rather than by a test. One decision point turns whether every operation is checked
from a review question into something #98 can refuse, and it turns the matrix in
the issue's third condition into a test against one function rather than a crawl
over handlers.

What that costs, stated because it is not free. Every operation has to carry enough
context to be decided in one place, so a handler that could have decided something
cheaply from what it already held passes it along instead. That is the price of
the property and it is paid knowingly.

## What this imposes on other issues

#48 owed the read-side requests named above and has specified them. The
permission for them is the row it always was, viewer, and the contract added no
question this model had to answer a second time.

#49 owns what proves who a person is. This model is stated against a person and
takes no position on how they are identified. Disabling an account is #49's, and
this model is what has nothing left to decide once it happens.

#51 refuses a write claim to a viewer, and the refusal is `not-permitted` rather
than `write-claim-held`, because there is no holder to name.

#57 decides whether a change needs review. This model gives that no role, so #57
may put it on the project or on the document without either choice adding a name
here.

#58 records membership changes, role changes, revocations and operator
appointments, with actor, subject and time, and distinguishes an operator
appointment from an in-project promotion.

#80 and #110 own the operator surface that appoints an administrator for a project
that has none.

#98 refuses an operation that does not pass through the decision point, and #20
puts this surface on the gated list. Both are conditions on the issue this
document is written under, and both are code.

## What this does not decide

It sets no number. Session lifetimes, claim durations and request limits are
#49's, #51's and #59's.

It decides no credential, no storage form and no login route.

It says nothing about what an operator may see on their own server. That is N1 in
the threat model and is outside every role here.

It does not decide how a role is displayed or where it is set in an interface.

## What refuses any of this today

Nothing. PROSE, NOT ENFORCEMENT. #98 owes the mechanism for the single decision
point and #20 owes the coverage bar, and there is no code for either to run
against yet. No record in `.github/invariants.txt` can carry this instead: every
rule there is a pattern handed to `git grep`, and a handler that decided a
permission on its own matches no pattern that a handler calling the decision point
does not.

## Revisiting this

Two conditions, each with something to check rather than a feeling.

If an operator asks for a permission that is not one of these three roles, the
per-operation model refused above is what this becomes, and the three names stay
as sets over it.

If #57's review turns out to need somebody who may publish only for review, that
is the publisher role refused above arriving with evidence, and it is decided
again against that evidence rather than against this paragraph.
