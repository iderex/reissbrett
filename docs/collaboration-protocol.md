# The collaboration protocol contract

Two programs written at different times have to agree on something. This
document is that agreement: every message a client and a server exchange, when
it is sent, what the receiver has to do with it, what happens to one nobody
understands, and what may and may not be relied on about the order things
arrive in.

It is written before either program exists, which is the point. A protocol that
is whatever the current client happens to send is a protocol nobody can
implement a second client against.

## Nothing here has been measured

There is no client, no server and no code in any language on the mainline:

    gh api "repos/iderex/reissbrett/git/trees/main?recursive=1" --jq '[.tree[] | select(.type=="blob") | .path | split(".") | last] | unique | join(" ")'
    DCO LICENSE gitattributes md txt yml

So every number below is a bound this contract sets rather than a measurement
anybody took, and every sentence about behaviour is a requirement rather than an
observation. Where a number has to be chosen against a measurement, this
document says which issue owes the measurement instead of inventing one.

## The one constraint placed on this document from outside

`docs/decisions/0007-the-collaboration-model.md` chooses locking for the first
release and places exactly one design constraint here, in its own words:

    git grep -n 'The seam is #48' origin/main -- docs/decisions/0007-the-collaboration-model.md
    origin/main:docs/decisions/0007-the-collaboration-model.md:214:The seam is #48. If the protocol contract is written so that the question the

The question the server answers on the wire is whether this person may write
this document now. It is not whether this person holds this lock.

That is why the word lock does not appear in any message name or field name
below. What the client asks for is a write claim on a document. Under the model
in force today a write claim is granted by taking a lock and refused when
somebody else holds one, and the refusal carries who and since when because that
is what a person needs. Under a different model the same request would be
answered a different way and the same messages would carry it. The model lives
in the server. The wire carries the question and the answer.

This costs one thing and it is worth naming rather than presenting the seam as
free. A client reading `write-claim.refused` with a holder and a time cannot tell
a lock from anything else that produces a holder, so a later model that refuses
for a reason with no holder has to answer with a refusal code this contract
already declares rather than inventing a shape. The closed code set below is
where that is held.

## The connection this rides on

One connection per client, protected by `docs/decisions/0116-protecting-the-client-server-connection.md`,
which decides TLS at 1.3 or above with the server identified by a public key the
client records on first connection.

This contract describes no unprotected variant of itself. There is no message
below that a client may send before the connection is protected, no
downgrade exchange, and no field by which either side offers to continue without
it. A reader looking for the plaintext form of this protocol should find nothing,
because there is nothing, and that is deliberate rather than an omission.

Everything below assumes the connection is established and the server's identity
has been accepted. What happens when it has not is `0116`'s and is not restated.

## Frames

Every byte on the connection after the protection is established belongs to a
frame. A frame is:

- one byte of kind
- four bytes of length, unsigned, most significant byte first
- that many bytes of body

Two kinds exist and no more.

Kind `0` is a control frame. Its body is one JSON object encoded as UTF-8.

Kind `1` is a payload frame. Its first sixteen bytes are the identifier of the
transfer the bytes belong to, and the rest is content. A payload frame carries no
structure of its own, and nothing about it is parsed.

A receiver reads the kind and the length, compares the length against its
configured maximum for that kind, and refuses before it allocates anything. That
ordering is the whole reason the length is in the frame header rather than in the
body: #59 requires that the unauthenticated path allocate nothing proportional to
a size the client declares, and a length inside a JSON body cannot be checked
until after the body has been read. The maximum for each kind is configuration
under #59 with a documented default, and this contract sets no number.

A frame whose kind is neither `0` nor `1` ends the connection, after one control
frame carrying `refused` with the code `frame-kind`. Framing errors are not
recoverable: a receiver that does not know where the next frame begins cannot
resynchronise, and pretending otherwise turns one bad byte into a stream of
plausible messages.

## Why the control channel is JSON, and why the payload channel is not

The means, argued here rather than carried over from habit.

The control channel is small structured messages: an identifier, a hash, a
version number, a code, a sentence. It is never model data, because model data
travels as payload frames. JSON encodes that, it is in the standard library of
both languages #14 records for this project, and it adds no schema compiler, no
generated code and no third kind of source file to a tree that counts two
languages deliberately. A transcript of a session is also readable, which matters
on a board where a claim is expected to carry the command that produced it and
where the output of that command has to be quotable.

What it costs, stated rather than left to be met. JSON carries no schema, so
nothing refuses a malformed message except the code that reads it, and that code
has to be written as though every field were hostile because it is: the decoder
is a surface that takes bytes from strangers, which is why it is under #94's
fuzzing and #20's coverage bar. JSON also carries bytes badly, which is why no
model data is ever in a control frame.

The payload channel is raw bytes with a sixteen byte identifier in front. Bytes
that are already content-addressed have no structure worth encoding, and
inflating them by a third to put them inside a text format would be paying for
readability on the one part of the traffic nobody reads.

The alternatives, and why each was rejected.

A schema language with a code generator, whether protocol buffers or anything
shaped like it. Rejected because it adds a compiler, a generated-code step and a
third file type to a tree that priced itself at two languages, and it buys
compactness on a channel that is small by construction. If the control channel
ever turns out to be a measurable cost, this is the answer to make again, and the
revisit condition below says on what evidence.

A binary object encoding with no schema, such as CBOR. Not rejected on the
merits, and it would answer every requirement here. It is not chosen because it
adds a dependency in both languages and gives up the readable transcript, and
neither of those is paid for by anything this contract needs.

One connection carrying only JSON, with model data base64 inside it. Rejected
because #48 already says the transfer is separate from the messages, and because
a third of the traffic would be encoding overhead on the largest thing this
project moves.

Two connections, one for control and one for content. Rejected because it doubles
what `0116` has to establish and because ordering between the two would then be a
guarantee this contract could not make.

## Version negotiation

`docs/version-policy.md` already fixes the rules: one integer starting at 1, a
server accepts its own version and the one below it, a client refuses a server
that offers only a lower one, and a client newer than the server is refused at
connect with a message naming both sides and which one is upgraded. Those rules
are not restated here. What this contract adds is the exchange that carries them.

The client sends `hello` as the first control frame on the connection. It carries
the protocol version the client speaks, and nothing else that the server needs
before it decides whether to keep talking.

The server answers `hello.accepted`, carrying the version that will be used for
the rest of the connection, which is the client's version if the server accepts
it. Every message after that point is read at that version by both sides.

Or the server answers `refused` with the code `protocol-version`, carrying the
versions it accepts and the version the client offered, and ends the connection.
The client turns that into the sentence naming which side is upgraded, in #63's
vocabulary. The server does not compose that sentence, because the client is
where a person is.

Nothing else happens before `hello.accepted`. In particular no session, no
credential and no document identifier crosses the connection first, so a stranger
who reaches the socket can cause the server to do one comparison of two integers
and nothing more.

## The one field this contract does not fix

Whether the negotiation carries an identity for the server is not settled, and
this document does not settle it.

Entry 6 of #1 is answered and federation is declined. Two records of that answer
put different obligations on this contract, and #1 says so in its own words
rather than picking one:

    gh issue view 1 --repo iderex/reissbrett --json comments --jq '.comments[-1].body' | grep -n 'Nothing here picks one'
    35:Nothing here picks one. What is recorded is that entry 6's answer is settled and

One reading requires a server identity field in the negotiation and asks #60 to
test it. The other reads the same answer as the absence of that field, on the
ground that a second set of identity questions is the cost the first release
avoids.

So `hello.accepted` above is specified without one, and that is a gap rather than
a decision. Adding the field later moves the protocol version under the policy
already written, which is what makes leaving it out survivable and is not an
argument for leaving it out. This is the one thing on this contract that is mine
to decide, and it is the only field below whose presence is undecided.

## The message set

Every message is a control frame. Every message carries `type`. Every request
carries `id`, chosen by the sender, and the response to it carries the same `id`.
A request is a message a response is expected for. A notification is a message no
response is expected for.

The fields listed per message are the ones a receiver may rely on. Fields are
described by what they hold rather than by a type name, because naming a type
here would fix an encoding detail this contract has no reason to fix.

### Connecting

`hello`, from the client, once, first. Carries the protocol version the client
speaks. The server answers `hello.accepted` or refuses.

`hello.accepted`, from the server. Carries the version in force for this
connection.

`session.begin`, from the client. Carries whatever #49 decides proves who the
person is. This contract carries the field and specifies nothing about its
contents, because credential handling is #49's and a protocol document that
invents one would be deciding it in passing. The server answers `session.begun`,
carrying an identifier for the session and the time it expires, or refuses.

`session.end`, from the client. The server answers `session.ended`. A connection
that closes without it ends the session too, so this exists to make the ordinary
case explicit rather than to make the abrupt case work.

`session.expired`, from the server, a notification. Sent when a session ends for
a reason the client did not ask for, including the account being disabled, which
#49 requires. The client stops sending anything but `session.begin`.

### Reading

Four requests here, and a fifth under `### Moving parts` below for the bytes.
How many requests the read side needs, whether listing a project's documents and
listing a document's versions are one message or two, and whether a client asks
for parts by name or asks for a version and is told the names, are answered
below rather than left to whoever writes the first client.

`projects.query`, from the client. Carries nothing. The server answers
`projects`, carrying every project the person is a member of, each with an
identifier, a name, and the role the person holds there. A project the person is
not a member of is absent from the answer. There is no request that names a
project the person has not been told about.

`documents.query`, from the client. Carries a project. The server answers
`documents`, carrying every document in that project the person may read, each
with an identifier, a name, and the identifier of the version that is current for
it. A document the person may not read is absent rather than refused, for the
same reason as above: a refusal naming a document is a statement that the
document exists.

`versions.query`, from the client. Carries a document. The server answers
`versions`, carrying that document's recorded versions in the order they were
published, each with its identifier, who published it, when, the message the
publisher gave, and whether it is the current one. This is the history a viewer
may read under `docs/decisions/0050-the-permission-model.md`, and it is the list
#42 compares two entries of.

`version.query`, from the client. Carries a document and a version identifier.
The server answers `version`, carrying exactly what `publish` carried for that
version: the ordered list of part names, the container framing
`docs/decisions/0006-what-a-version-of-a-model-is.md` requires to reassemble the
file, and the readable description of the feature tree. It refuses with
`no-such-version` where the document is readable and the identifier is not one of
that document's versions.

That refusal is worth naming rather than passing over. `no-such-version` has been
in the closed code set below since this contract was first written, and until
`version.query` existed no message in the set could produce one. A declared
refusal that nothing can cause is a hole in the message set with a label on it,
and this is the message the label was waiting for.

Where a person may not reach a project or a document at all, the answer is
`not-permitted`, and it is the same answer whether the subject exists or not.
That is #49's rule about a failed attempt being indistinguishable from a failure
of a different kind, applied one level up: a caller who cannot reach a thing
learns nothing about whether it is there. `no-such-document` is therefore only
ever seen by a caller who could have read the document had it existed, which is a
caller already holding the project's listing.

#### Why these are four messages and not one

Each answers a different question about a different subject, and the sizes differ
by orders of magnitude. One message with a field selecting between them would be
a message whose required fields depend on the value of another field, and this
contract cannot make that safe: the rule below reads past an unknown field, which
is only sound while every field's meaning is fixed by the message type alone.
`write-state.query` is already separate from `notification.query` for the same
reason, so this is the shape already in use rather than a new one.

#### Why a client is told the part names and then asks for them by name

The alternative is one request that names a version and receives its bytes. It is
rejected because a part is named by a hash of its content, so a workstation that
already holds a part - from another version of the same document, from another
document that shares it, or from the version it published itself - would receive
it again. `parts.offer` and `parts.wanted` exist so that a client sends only what
the server lacks, and a read side without the same two steps gives that property
up in the direction where the file is usually larger.

So reading a version is `version.query` for the names, then `parts.request` under
`### Moving parts` below for the bytes of the ones the client does not hold. The
client decides what it holds. The server is never asked.

#### What none of these asks the server to do

None of them asks the server to open a document. Every field above is either
metadata the server recorded when a version was published or bytes it stored
under a name, and `docs/decisions/0037-the-process-boundary.md` is untouched by
all four: the readable description is returned because it was uploaded, not
because the server produced it, and the container framing is returned for the
same reason. The section on what this protocol does not do says that a message
asking the server to open a document would undo that decision in a field, and
that sentence still holds against every message here.

### Writing

`write-claim.request`, from the client. Carries the document and, if the client
wants one, a requested duration. The server answers `write-claim.granted` or
refuses.

`write-claim.granted`, from the server. Carries a claim identifier, the time the
claim expires, and the interval at which the client is expected to renew it. The
duration and the interval are the server's to set: #51 requires them configurable
and documented, and a client that asked for longer gets what the server gives
rather than what it asked for.

`write-claim.renew`, from the client. Carries the claim identifier. The server
answers `write-claim.renewed` with a new expiry, or refuses with the code
`claim-lost` if the claim is no longer the client's, which is the case #51 calls
the one that decides whether the model is tolerated.

`write-claim.release`, from the client. Carries the claim identifier. The server
answers `write-claim.released`.

`write-claim.revoked`, from the server, a notification. Sent when a claim the
client holds stops being the client's while the connection is up, whether it
expired or an administrator took it. Carries which of those it was, and who did
it where a person did. #51 requires the person who held it to be told, and this is
the wire half of that.

`write-state.query`, from the client. Carries a project or a list of documents.
The server answers `write-state`, carrying per document whether it is free, held
by the client, or held by somebody else with who and since when, and for a held
document how close it is to expiry. That last part is what #52 shows differently
from an actively held one, and the threshold is #52's to state.

`write-state.changed`, from the server, a notification. Carries the same shape
for one document. It is an optimisation and not a guarantee, which the section on
ordering below is precise about.

### Moving parts

Parts are what `docs/decisions/0006-what-a-version-of-a-model-is.md` records a
version as a list of, named by a hash #40 chooses. The transfer is separate from
the messages, and this is what separate means.

`parts.offer`, from the client. Carries the names of the parts a version is made
of. The server answers `parts.wanted`, carrying the subset it does not already
hold. #53 requires that only those are sent, and this exchange is how the client
finds out which.

`transfer.begin`, from the client. Carries a transfer identifier and the name of
one part. The server answers `transfer.ready`.

Payload frames follow, carrying that transfer identifier. The bytes of one part
may be split across as many frames as the sender likes, and frames belonging to
different transfers may be interleaved, which is what stops one large part from
blocking every small one.

`transfer.end`, from the client, carrying the transfer identifier and the number
of bytes sent. The server answers `transfer.stored` when what it received hashes
to the name that was declared, and refuses with the code `content-mismatch` when
it does not. The server never stores a part under a name its bytes do not
produce, and that check is the reason content addressing is worth having on this
route rather than only on disk.

`transfer.cancel`, from either side, carrying the transfer identifier. Everything
received for it is discarded.

`parts.request`, from the client. Carries the names of the parts it wants. The
server answers `parts.sending`, carrying the subset of those names it holds and
will send. A requested name missing from that subset is a part the server does
not hold, and that is an answer rather than a refusal, for the same reason
`parts.wanted` is one: the client asked about a set, and the useful reply is a
set.

The same three transfer messages then carry a part in the other direction, sent
by the server, with the client doing the hashing and the refusing. Restoring a
version a workstation does not hold is that direction, and `parts.request` is
what starts it. Before that message existed this paragraph described a direction
nothing could ask for.

### Publishing and what follows

`publish`, from the client. Carries the document, the ordered list of part names,
the container framing `0006` requires to reassemble the file, the readable
description of the feature tree, and what to do with the write claim afterwards.
That last one is a field rather than an assumption, because #53 requires the lock
behaviour on publish to be explicit in the request.

The readable description is produced on the workstation and uploaded, because
`docs/decisions/0037-the-process-boundary.md` decides that the process listening
on the socket never loads the suite. The server cannot check that the description
matches the parts, and `0037` says what that costs and why it is bounded.

The server answers `published`, carrying the version identifier, or refuses. A
publish is one operation from the outside: it either becomes the version others
see or it leaves the document on the previous one, which is #47's guarantee
applied at this level and #53's fourth condition.

`publish.for-review`, from the client, with the same fields. Used where #57's
review is on for the project. The server answers `published.for-review`.

`review.decide`, from the client. Carries the change under review and either an
approval or a refusal with its reason. #57 requires the reason to live with the
change rather than underneath it, so it is a field on this message and not a
comment somewhere else. The server answers `review.decided`.

`notification.query`, from the client. Carries a project. The server answers
`notifications`, carrying every outstanding notification for that person: which
document of theirs references which document, which version it is pinned to,
which version was published, how many published versions the pin is behind, how
old the pinned version is, and the result of #55's check where one has been run.
`docs/decisions/0008-how-a-published-change-reaches-what-references-it.md`
requires all of those and this message is where they arrive.

`notification.new`, from the server, a notification. The same shape for one item.
Also an optimisation, for the same reason as `write-state.changed`.

`reference.decide`, from the client. Carries the notification and either an
acceptance naming the new version or a refusal naming the class of finding it was
refused on. The server answers `reference.decided`. A refusal leaves the pin where
it is and leaves the notification outstanding, which `0008` requires, so the
server does not clear it.

### Refusing

`refused`, from either side, in answer to a request, carrying the same `id`.

Three fields. A code from the closed set below. The subject the code is about,
where there is one, such as the document or the limit. And a sentence for a
person, in #63's vocabulary, which the client may show and may replace with its
own wording for the same code.

The codes, and this set is closed. A version that needs another one is a version
move, because a client that meets a code it does not know cannot act on it:

`protocol-version`, `frame-kind`, `frame-too-large`, `malformed`,
`unknown-type`, `not-authenticated`, `session-invalid`, `not-permitted`,
`write-claim-held`, `claim-lost`, `no-such-document`, `no-such-version`,
`content-mismatch`, `limit`, `unavailable`.

`write-claim-held` is the one that carries a holder and a time. `not-permitted`
is #50's answer and carries no holder, because who may write a document at all is
a different question from who is writing it now. `limit` names which limit under
#59, because a client that cannot tell a limit from a network failure retries and
makes it worse.

Nothing in this set says what went wrong inside the server. A refusal that leaks
an internal reason to a caller who failed to authenticate is a refusal that
answers a question the caller had no right to ask, which is #49's rule about a
failed attempt being indistinguishable from a failure of a different kind.

## A message nobody understands

Both directions, because a client meeting an unknown message from a server is the
case that gets forgotten.

A control frame whose body is not one JSON object, or which carries no `type`, is
`malformed`. The receiver refuses it. Where it carried no `id` there is nothing to
correlate the refusal with, so the connection ends after the refusal, because a
sender that cannot be told which message was bad will send it again.

A control frame with a `type` the receiver does not know: a request is refused
with `unknown-type` carrying the type, and the connection stays up. A
notification is ignored. That difference is deliberate. A request that is not
answered leaves the sender waiting forever, and a notification is by definition
something the receiver was not required to act on.

A known type carrying a field the receiver does not know is read without the
field, and the field is not an error. Negotiation guarantees the two sides are at
most one version apart, so an unknown field is a peer one version ahead.

That rule is only safe because of a second one, which this contract imposes on
every version after this one. No version may add a field to an existing message
type whose absence changes whether it is safe to act on that message. A change of
that kind is a new type, and the version moves. Without that, ignoring an unknown
field is a receiver acting on a message it only partly understood, and this
contract would be specifying that as correct behaviour.

A payload frame naming a transfer the receiver does not know is discarded and
answered with `refused` carrying `unknown-type` and the transfer identifier. It is
not a framing error, because the frame was well formed and the receiver knows
where the next one starts.

## What may be relied on about ordering and delivery

This is the section #60 tests against, and it is written as what may be relied on
rather than as what will usually happen, because the second is not a contract.

On one connection, control frames arrive in the order they were sent, exactly
once, or the connection fails. That is the transport's property rather than this
protocol's, and this protocol adds nothing to it and takes nothing away.

Responses do not arrive in the order their requests were sent. The server may
answer concurrently, and correlation is by `id` and never by position. A client
that assumes otherwise works until the day two requests take different amounts of
time, which is every day.

Across connections nothing is guaranteed at all. A request sent on a connection
that then failed may have been acted on or may not, and the client cannot tell
from the wire. So every request that changes state carries an `id` the client
chose, the server records the outcome against it, and a client that reconnects and
sends the same `id` again receives the same answer rather than causing the change
a second time. That is what makes a publish interrupted by a dropped connection
safe to retry, and it is a requirement on the server rather than a convention.

For one document, the server applies write claims and publishes in a total order,
and every client sees a state consistent with that order. Across documents there
is no order at all. Two publishes to two documents may be observed in either
order by anybody, and nothing may be built on which came first.

Notifications from the server are not delivered. They are an optimisation, and
the authoritative route to both notifications and write state is the client
asking. A client that has just connected asks; a client that has been connected
may rely on what it was told last and knows it may be stale. Two things follow
and both are the point. A person who was offline when something was published
finds it on their next connection, which is what #53 requires and what #56 makes
possible. And #52's display can say when what it shows was last confirmed,
because there is a moment it was confirmed at, rather than an assumption that
nothing has happened since.

A read answer describes the moment the server answered it. `documents`,
`versions` and `version` are true when they are sent and may be stale when they
are read, and this contract promises nothing else about them. Two of the three go
stale in one direction only, because nothing here removes or rewrites a recorded
version: an answer about a version stays true and a version list can only grow.
`documents` carries a current version identifier, and that one moves.

What is deliberately not promised: that a notification arrives at all while a
connection is up, that two clients see a change at the same moment, and that a
`write-state.changed` a client did not receive means nothing changed.

## What this protocol deliberately does not do

Each with the issue or the open question it belongs to.

It carries no model data in a message. Parts are named by hash and their bytes
travel as payload frames, which is #40's addressing and #53's transfer.

It describes no hosted service. Entry 4 of #1 is answered as self-operation only,
so there is no message about tenancy, billing or an account that spans operators.

It describes no server-to-server exchange. Entry 6 of #1 declines federation
without foreclosing it. Everything above is between one client and one server,
and the one place a future answer would land is the negotiation, which is the
open field named earlier.

It carries no merge. `0007` rejects optimistic editing with a merge and says what
would have to change for that to be looked at again. There is no message here
that combines two versions.

It carries no event feed. #58's audit trail is the operator's, it is read and
exported on the server, and #58 says in its own words that it must not become a
general event log. Putting it on the wire would be the first step toward one.

It carries no search. `documents.query` and `versions.query` answer with what is
there, in full, and nothing above takes a query expression. What a search would
match is a decision nobody has taken, and a field for it here is where it would
get taken by accident.

It carries no partial read of a part. A part is named by a hash of its bytes, so
a receiver holding some of them can check nothing, and that check is the whole
reason the name is a hash. A transfer may still be split across as many payload
frames as the sender likes, because the receiver checks once at the end.

It carries no telemetry of any kind. #86 decides that, and the absence here is not
a place a later decision fills in quietly.

It asks the server to compute nothing about a model. `0037` keeps the suite out of
the listening process, and the readable description is produced where the kernel
already is. A message that asked the server to open a document would be that
decision undone in a field.

It decides no credential and no permission. #49 owns what proves who a person is
and #50 owns who may do what; this contract carries the field for the first and
the refusal for the second, and specifies neither.

It sets no number. Frame maxima, rates, connection counts and content upload sizes
are #59's, against #117's measurements. Claim durations and renewal intervals are
#51's. Nothing above quotes a number, so nothing above can be quoted later as
though a number had been chosen here.

## What this imposes on other issues

#49 fills the contents of `session.begin` and the meaning of `session.expired`,
and nothing in the credential design may require a message this contract does not
have.

#50 answers `not-permitted` from one decision point, and the protocol offers no
second route to an operation that would bypass it.

#51 sets the claim duration and the renewal interval, and produces
`write-claim.revoked` for both expiry and an administrator, distinguishing them
in the field this contract declares.

#52 reads `write-state` and `write-state.changed` and says when what it shows was
last confirmed, which the ordering section above makes possible rather than
optional.

#53 uses `parts.offer` and `parts.wanted` so that only the parts the server lacks
are sent, and carries the claim behaviour in the `publish` request rather than
implying it.

#54 sends `reference.decide` and relies on a refusal leaving the notification
outstanding.

#55's result travels in `notifications` and in `notification.new`, and the
sentence stating what the check does not establish travels with it rather than
being added by the client.

#57 uses `publish.for-review` and `review.decide`, with the refusal reason as a
field.

#42 produces its summary on a workstation from what `version.query` returns for
two versions. The description is stored with the version and the server computes
nothing, which is `0037` held on this route as well.

#43 restores a version with `version.query` for the names and the framing,
`parts.request` for the bytes it does not already hold, and the byte comparison
`0006` states, made where the file is reassembled.

#50's table names a message beside every operation it governs, and its read row
now has four to name instead of the absence it recorded.

#56 reconciles on reconnect by asking rather than by being told, and the four
requests above are the asking. The ordering section is what makes that the
authoritative route.

#59 applies every bound at the frame header, before a body is read, and answers
`limit` naming which one.

#60 tests exactly the guarantees in the ordering section and no others, including
the retry of a state-changing request under its original `id`.

#94 fuzzes the frame reader and the control decoder as one surface, because the
frame header is where the first hostile number arrives.

#102 counts this connection among the ones it enumerates, with the protection
`0116` decides.

## What refuses any of this today

Nothing. PROSE, NOT ENFORCEMENT, and the mark is not the terminal kind: every
statement above is a property of code that does not exist, and #48's own fifth
condition is where it becomes refusable. That condition asks for a conformance
test derived from this document, running headless, failing when an implementation
departs from it. Until that lands, what stands behind this contract is this
document.

Two of the properties here are worth naming as the ones a conformance test has to
reach rather than sample, because they are the ones an implementation passes by
accident. That an unknown request type is refused and the connection survives.
And that a state-changing request replayed under its original `id` produces the
same answer rather than the change a second time.

## Revisiting this

Three conditions, each checkable.

If I answer the open field above in favour of a server identity in the
negotiation, it is added, the protocol version does not move because nothing
has been built yet, and the answer is written down here and not assumed.

If #117's measurements show the control channel is a measurable cost at the size
of deployment this release supports, the encoding is argued again against those
numbers, and the schema-language alternative rejected above is where that argument
starts.

If #7's model is replaced, the messages above are the ones that survive it and the
server's answers are what change. If that turns out to be false, the seam this
contract was written around did not work and this document is where it is
recorded, rather than in the issue that discovers it.

No second person has read this. The commands above read the mainline and the
tracker rather than a working tree, and they stand in place of one.
