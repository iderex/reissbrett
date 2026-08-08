# The version policy

This is the decision behind issue #111. A version number is a promise, and this
project has four of them to keep apart: its own, the version of the underlying
suite a release stands on, the store format on disk, and the protocol between a
client and a server.

Collapsing them into one number would be simpler to say and wrong in the place
it matters. An operator upgrading a server asks whether their clients still
connect, whether the store they have can still be read, and whether the suite
they have installed is still supported, and those are three different questions
with three different answers.

## Nothing here has been observed

Every rule below is a decision made while this document was written. None of it
is a measurement, because there is nothing to measure:

    gh api repos/iderex/reissbrett/releases --jq 'length'
    0
    git ls-remote --tags https://github.com/iderex/reissbrett | wc -l
    0
    git ls-tree -r --name-only origin/main | grep -cE '\.(go|py|cs|rs|ts|js|cpp|c|java)$'
    0

The policy is written before the artefacts it governs on purpose. A numbering
rule written after the first few releases is a description of what happened to
be shipped.

## This project's own number

Three parts, `MAJOR.MINOR.PATCH`, and what a change in each one promises.

A patch release carries defect fixes and nothing else. Same suite pin, same
store format, same protocol version. It goes on over the previous patch release
without a plan, and it can be rolled back to the release it replaced.

A minor release carries new capability, and it is also where a move of the
suite pin within its series lands. The store format and the protocol version
are unchanged, so a client one minor release behind still connects and a store
written by the new release is still readable by the old one.

A major release is one an operator has to plan for. Exactly three things
require one: a change to the store format number, a change to the protocol
version that drops the previous one, and a move of the suite pin to a different
series. A major release is the only kind that can make a rollback impossible,
and its notes say which of the three it is.

There is no `0.x` series where these rules are suspended. An operator running
an early release against their shop's designs carries the same risk as one
running a later release, and a leading zero does not reduce it, so the first
release is `1.0.0` and every rule above applies to it. What that costs is
real and is worth seeing before it bites: a change that would have been quiet
under a `0.x` label becomes a major release with notes, and there will be more
major releases early than a project that gave itself a grace period would have.

## The suite version a release stands on

Not this project's number to choose. #5 pins it and
`docs/decisions/0005-the-pinned-suite-version.md` holds the decision. What this
policy adds is that the pin is stated with the release rather than discovered
after an upgrade.

The pin means two different things on the two delivery routes, and the release
states both:

    git show origin/main:docs/decisions/0005-the-pinned-suite-version.md | sed -n '9,21p'
    The first release stands on FreeCAD 1.1.3.

    The bundle in #11 carries exactly that version and no other, so for anyone using
    the product as delivered the pin is a property of the artefact rather than a
    request.

    The extensions route in #11 supports the 1.1 series: 1.1.0 and later, up to and
    including the newest patch release of that series that the compatibility harness
    in #34 has actually been run against. A version outside that range is refused at
    load rather than tolerated, and #35 is what does the refusing.

    No weekly build is supported, in either route.

So a release carries an exact version for the bundle and a range for the
extensions route, and the range has an upper end that is a fact about what #34
has been run against rather than a number somebody chose. A release states the
range it was actually tested to, not the range it hopes to cover.

## The store format version

One integer, starting at 1. It moves when a store written by the new release
cannot be read by the previous one, or when a store written by the previous one
needs a migration before the new one can read it. #46 owns the migration.

A release reads store formats at or below its own and migrates upward once, on
first use.

A release refuses a store whose format number is above its own, rather than
reading the parts it recognises. Reading the recognisable parts of a newer store
is how a downgrade silently drops what it did not understand, and a version
store that quietly loses history is worse than one that will not open.

A downgrade is possible only between releases that share a store format number.
Once the number has moved and a store has been migrated, going back is not
available, and the notes of the release that moves it say that in those words
rather than leaving it to be discovered.

## The protocol version

One integer, starting at 1, negotiated when a client connects. It moves when a
message either side has to understand changes shape or meaning. #48 owns the
negotiation and is where this rule is enforced.

A server accepts its own protocol version and the one below it. A client speaks
its own version and refuses a server that offers only a lower one.

That fixes the upgrade order rather than leaving it to be worked out: the
server is upgraded first, and the clients follow. One version of overlap is
enough for a shop to upgrade its clients after its server rather than at the
same moment, and it is deliberately not enough to leave a client several
releases behind indefinitely. Every extra version a server accepts is a second
code path through the part of the system that holds other people's work.

A client newer than the server is refused at connect, with a message naming the
version the client speaks, the version the server accepts, and which side is
upgraded. It is refused rather than allowed through, because a new client
against an old server creates records the server cannot store.

## The combinations that are supported

Between a client and a server: protocol version N on the server accepts clients
at N and at N-1. Anything else is refused.

Between a server and a store: the store's format number equals the server's
own, or is below it and is migrated on first use. Above it is refused.

Between a release and the suite: the exact pin on the bundle route, the range
above on the extensions route.

Every combination outside those three is refused rather than warned about. The
refusals are owned elsewhere and none of them exists yet: #48 for the protocol,
#46 for the store format, #35 for the suite version.

## What a change to the suite pin does to this project's number

A move of the pin is never a patch release. The reason is in #5's own policy
for moving it:

    git show origin/main:docs/decisions/0005-the-pinned-suite-version.md | sed -n '118,121p'
    When all four hold, the pin moves and the previous version leaves the supported
    range on the extensions route at the same time, so the range never grows by
    accident. When one of them does not hold, the pin stays and the reason is
    written into the issue that proposed the move.

Support is taken away from somebody at the moment the pin moves, and taking
support away inside a patch release is how an operator is surprised by an
upgrade they were told was a fix.

Within the pinned series, a pin move is at least a minor release. To a
different series it is a major release, because the module surface #5 measured
between series is the kind of change this layer has to carry two answers for.

The number makes the event visible and the notes say what it was: both the
version left and the version arrived at, named.

## What this fills in for the security policy

`.github/SECURITY.md` states a rule and points here for the definition behind
it:

    git show origin/main:.github/SECURITY.md | sed -n '56,62p'
    There is no released version of this project yet, so nothing is supported and
    nothing is being patched.

    The rule that applies once there is one: the current release series receives
    security fixes, and a series stops receiving them when the next one is published
    unless the version policy says otherwise. #111 is where that policy is written
    and what this section is filled in from. Until #111 lands, this paragraph states

A series is a `MAJOR.MINOR` pair, and security fixes on a series are published
as patch releases on it. This policy says nothing otherwise, so the rule in the
security policy stands as written and needs no exception.

That section is not edited by this change, and the reason is not tidiness. It
would have to name a supported version, there is no released version to name,
and #111's condition asking for it is open until there is one. Filling it in
now would put a supported version into a security policy for software nobody
can install.

## What this policy does not decide

The post processor and its own version are recorded in a produced program by
#77, and whether that version participates in this project's number is a
question for #74 and #77 rather than one this document folds in. A post
processor is selected by the machine profile in #73, so a shop can be running
two of them against one release, and a number that covers the release cannot
also cover which one they chose.

Federation between two operators' servers is entry 6 of #1, it is open, and
this policy covers a client and a server under one operator. If that entry is
ever answered in the direction of federation, this document gains a section
about a version boundary between servers rather than having one of its rules
rewritten.

## What refuses any of this today

Nothing does. There is no code in this tree, no release, and each of the three
refusals named above sits in an issue that is open:

    for n in 35 46 48; do gh issue view $n --repo iderex/reissbrett --json number,state,title --jq '"#\(.number) \(.state) \(.title)"'; done
    #35 OPEN Detect a suite version this layer was not built for, and refuse
    #46 OPEN Migrate a store written by an older version of this software
    #48 OPEN Write the collaboration protocol contract

So this document is prose, not enforcement, in the whole. It is the statement
those three issues are built against, and until they land nothing in this
repository refuses a combination this policy forbids.
