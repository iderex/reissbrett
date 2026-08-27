# The metrics, what each one is for, and how they leave the server

Issue #84. Taken on 2026-08-27.

The set is chosen backwards, from the decisions an operator has to make, so that a
metric which supports no decision is absent rather than merely unread.

## Nothing here has been built

There is no server and no metric. The tree carries no code in any language:

    git ls-tree -r --name-only origin/main | sed -n 's/.*\.//p' | sort -u | tr '\n' ' '
    gitattributes md txt yml

Every sentence below is a requirement. Nothing here reports a value, a unit that
was observed, or a cost that was measured.

## The export route, and why it is a pull

The server exposes the current value of every metric on a route an operator's own
collector reads when it wants them. The server sends nothing anywhere.

Three reasons, and the first is this board's rather than a preference.

A push makes an outbound connection. `docs/threat-model.md` records the position
this project takes on what leaves the host:

    git grep -n 'what it carries is an absence' origin/main -- docs/threat-model.md
    origin/main:docs/threat-model.md:188:different file. #86 is telemetry, and what it carries is an absence rather than

and #102 owes an enumeration of every outbound connection the delivery form can
make. A metrics exporter that dials out is a connection on that list, permanently,
for a feature the operator could have had by being dialled. Choosing the pull
keeps that list shorter by one, and it keeps it shorter without asking anybody to
configure anything.

A pull needs no state on the server. A push needs a destination, a credential for
it, a retry policy and a queue for what could not be sent, and every one of those
is a thing #80 has to validate and #81 has to keep out of a log. A route that
answers a request needs none of them.

A pull fails visibly. When the collector cannot reach the server the collector
says so, which is the same signal as the server being down, and that is the answer
an operator wants. A push that stops arriving looks like a service with nothing to
report.

## The format

A text format an operator's existing collector already reads, emitted by the
server writing bytes rather than by a library that owns the process.

The means check, since this is a format decision and the rule asks for one every
time. The format is text, so a claim about what the server exports can carry the
command that produced it, which a binary encoding would not allow without a tool
between the reader and the answer. It adds no dependency this tree has to carry:
the server writes the bytes, and what reads them is the operator's, on their side
of the boundary. It needs no parallel apparatus to test, because a test asserts on
a string. And nothing outside this repository forces it, which is stated here
rather than invented, so the choice rests on the three properties above and on
nothing stronger.

What was refused. A structured push protocol, for the reasons in the section
above. A format of this project's own, because the whole value of the choice is
that the operator's existing tooling reads it without being taught. A second
format beside the first, because two exports are two things that can disagree
about the same number and the disagreement is found by somebody debugging
something else.

This document names no library. Whether the bytes are produced by hand or by
something the toolchain already offers is a decision at the moment there is code,
and naming one here would decide it in a document with no business doing so.

## The metrics, and the decision each one supports

Every row states the decision an operator makes with it. A metric that reaches
this list without one does not land, which is this issue's own fourth condition
read as a rule rather than as a checklist item.

### Will the disk fill, and when

**Store size.** The bytes the version store occupies. Supports: whether the
operator needs more disk, now.

**Store growth rate.** The change in store size over a stated window. Supports:
when the operator will need more disk. This is the number #44 measures, and the
two are required to agree rather than to resemble each other: the metric is the
same quantity #44's retention argument is made against, not a second estimate of
it.

Both are reported against the deployment size #117 states, so an operator sees how
far along it they are rather than a number with nothing to compare to. That is
this issue's sixth condition and it depends on #117 having stated one.

### Is somebody blocked

**Write claims currently held.** A count. Supports: whether the shop is working
through the lock model or around it.

**Age of the oldest held claim.** A duration. Supports: the same decision, and it
is the half that moves first. A steady count with a growing oldest age is one
person who went home, and a count that rises with a flat age is a busy shop.

`docs/decisions/0007-the-collaboration-model.md` is what these two are evidence
about. If they say the model is working against this shop, that is the revisit
condition that document already carries, arriving as a number.

### Is it working

**Publishes.** A count. Supports: nothing on its own, and it is here because the
next two are meaningless without a denominator.

**Publish failures by kind.** A count per kind, where the kinds are the refusal
codes `docs/collaboration-protocol.md` declares. Supports: whether the failures an
operator is seeing are their problem or this software's.

**Worker process failures.** A count. Supports: the same decision, one layer down,
and it is separate because `docs/decisions/0037-the-process-boundary.md` makes the
worker a process that can die without the server dying. A server that looks
healthy while every job fails is the case this metric exists for.

**Rebuild duration.** A distribution rather than an average. Supports: whether the
machine the operator chose is large enough. An average hides the case that matters,
which is the slow tail somebody is waiting through.

### Is somebody attacking it

**Authentication failures, as a rate.** Supports: whether somebody is guessing.
As a rate rather than a total, because a total is a number that only ever goes up
and no operator can act on it.

**Refusals by limit, as a rate.** One series per limit #59 declares. Supports:
whether a limit is set wrong or somebody is pushing at it. Which of the two it is
cannot be read from the number, and the document says so rather than implying the
metric answers it.

## What is never exported, and what enforces it

No metric carries a label derived from model content or from document identity. A
metric labelled by document name carries a customer's part number to wherever
metrics go, and metrics go somewhere by design.

This is the same rule as `docs/decisions/0082-what-may-appear-in-a-log.md` on a
surface that feels impersonal and is not, and that document points here for it:

    git grep -n 'Metric labels' origin/main -- docs/decisions/0082-what-may-appear-in-a-log.md
    origin/main:docs/decisions/0082-what-may-appear-in-a-log.md:141:Metric labels. #84 has its own constraint on labels derived from model content,

The label set is closed. Every series above is labelled by what it is, by the
refusal code or the limit where a row says so, and by nothing else. A closed set is
what makes this issue's second condition a test that asserts on the label set
rather than a review that has to imagine what somebody might add.

## What this does not decide

The route the export is served on, whether it is authenticated, and whether it
sits on the same listener as the protocol. Those are #80's configuration and #116's
transport, and a metrics route that a stranger can read is a decision with a cost
neither of them has taken yet.

Any number. No window, no bucket boundary, no scrape interval and no retention.
Windows and intervals are the operator's, and bucket boundaries wait on #117's
measurements.

The units, beyond bytes and durations being what they say. A unit is stated per
metric where the metric exists, which is this issue's first condition and is code.

Retention of the exported series. That is the operator's collector and their
decision, and #103 tells them what the series contain.

## What refuses any of this today

Nothing. PROSE, NOT ENFORCEMENT, and the issue that owes the mechanism is this
document's own, #84, whose second condition asks for a test asserting the label
set.

No record in `.github/invariants.txt` can carry it instead. A label derived from a
document name is a value at run time, and every rule in that file is a pattern over
the text of a tracked file.

## Revisiting this

The pull decision is revisited if an operator's collector cannot reach the server
at all, which is a deployment where the server is behind something the collector
is not. That is a stated case rather than a preference, and the answer to it is
not necessarily a push.

The set is revisited by removal rather than by addition. A metric nobody has used
to make the decision its row names is a row that was wrong, and the argument for
taking it out is the one this document is organised around.
