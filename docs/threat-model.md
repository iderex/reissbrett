# The threat model

Every security decision on this board was made against a model somebody was
holding in their head. Writing it down is what lets a decision be checked
against it rather than trusted, and what lets the next decision be argued rather
than guessed.

This document is written before the software exists. It is a model of what is
being built and not a report on what was measured, and where it says a threat is
addressed it means an issue on this board owes the work, not that the work is
done. Where that distinction is easy to lose the sentence says which it is.

## What is being protected, in order

The order matters more than the list, because it decides what gives way when two
of them conflict.

**The integrity of the models.** First, and not by a small margin. A design that
is quietly wrong is the worst thing this project can produce. Everything else on
this list fails loudly: a model nobody can reach is obvious, a model somebody
read is discovered. A model that is subtly different from what its author left
behind is discovered on the shop floor or not at all, and by then it has been
machined.

**Their confidentiality.** Second. A shop's product designs are the shop's
commercial position, and the premise this project is built on is that they stay
on hardware the operator controls.

**Their availability.** Third, and third deliberately. Losing access to a model
for a day costs a day. This is not an ordering anybody would choose for a system
where availability is the product, and it is the right one here, because a store
that stays up by accepting a write it could not verify has traded the first item
on this list for the third.

**The correctness of what reaches a machine.** Placed last in this list and not
last in importance, which is why it has its own section below. It is a safety
concern rather than a security one, and it is here because the attack path is
the same one.

## Why a safety concern is in a security document

A program that moves a machine is produced from a model, through this project,
and handed to hardware that will do what it is told. Every step of that path is
the path an attacker would use, and every control on that path is a control
against both. A model altered in the store and a model altered by a defect
produce the same program.

So the threats below are written to cover both, and the distinction is kept in
the language rather than in separate documents. Where a control exists because
somebody could act, it says so. Where it exists because something could go
wrong, it says that instead. What is never claimed is that a produced program is
safe, which is #79's wording to write and #75's refusal to enforce.

## Who this defends against

Six, each with what addresses it. An issue named here owes the work; none of it
exists yet.

**T1. Somebody on the operator's own network who is not meant to have access to
a project.** The shop's network is not a trust boundary, and treating it as one
is how most of this goes wrong. Addressed by #49 for accounts and sessions, #50
for the permission model per project and per document, and #116 for protecting
the connection between the client and the server so that being on the wire is
not enough.

**T2. Somebody who has left.** The account is the boundary rather than the door
badge. Addressed by #49, because a session that outlives the account is the same
as no account, and by #58, so that what they did while they had access stays
attributable afterwards.

**T3. Somebody who gets a malicious document into the store.** A document is
somebody else's file format, parsed by somebody else's C++ kernel, and this
project's whole job is to accept documents. Addressed by #37, which keeps the
modelling kernel out of the process that listens on a socket, and by #94, which
fuzzes the surfaces that take bytes from strangers. #40 and #47 are the other
half of it: content addressed storage makes an altered part detectable rather
than silent, and a write that is interrupted is absent rather than half there.

**T4. Somebody who can reach the server from outside.** How exposed the server
is depends entirely on what the operator did, and this project does not get to
assume the good case. Addressed by #59 for rate limits and request sizes at the
edge, #49 and #50 as above, and #116 for the connection. #110 matters here too,
because a first run that does not tell an operator what is wrong is how a
default becomes an exposure.

**T5. Somebody who tampers with what this repository is built from.** The
supply chain before an artefact exists. Addressed by #21 for a locked dependency
graph with a reproducible restore, #96 for pinned actions and a lock file that
cannot drift, #18, #89 and #90 for analysis with more than one lens, and #22 for
a bill of materials that says what is actually in a build.

**T6. Somebody who tampers with what this repository publishes.** The artefact
after it exists. Addressed by #113 for signing and publishing release artefacts,
#96 for the rest of the chain around them, and #97, which asks for verified
signatures on the protected branch so that what goes into a release is what
somebody wrote.

## Who this does not defend against

Stated as plainly as the list above, because a threat model that defends against
everything defends against nothing.

**N1. The operator's own administrator.** Somebody who runs the server can read
what is on it. This project can make what they did visible through #58 and it
cannot make it impossible, and pretending otherwise would be the kind of
narrowing this project refuses elsewhere.

**N2. Somebody with physical access to the server.** Out of scope for the same
reason, and it is the operator's control to apply rather than this project's.

**N3. A compromised workstation belonging to somebody who legitimately has
access.** Their session is their session. What this project owes here is limited
and real: #81 keeps secrets out of a log and a bug report, so a compromised
machine does not hand over more than it already has.

**N4. The CAD suite's own defects.** This project implements no geometry kernel,
no constraint solver and no toolpath generator, and a defect in one of those is
reported upstream, which `.github/SECURITY.md` already says and does not
restate here. What this project does own is how it configures, packages and
ships that suite, which is inside T3 and T5 rather than outside this document.

**N5. Somebody who is trusted to merge here.** I am the only person with write
access. Nothing in this repository refuses a change I make, and the second
reader that would catch one is prose rather than a mechanism, which
`CONTRIBUTING.md` says in its own words. This is accepted rather than addressed,
and what would change it is a second person rather than a check.

## Threats accepted with a reason

Two, both of which an addressed-everything version of this document would have
quietly dropped.

**A malicious document opened in the desktop process.** #37 keeps the kernel out
of the process that listens on a socket, which is the server side. On the
desktop the kernel and the document are in one process by construction, because
that is what the suite is. Accepted, with the reason that the operator is
already running that suite on that machine, and this project does not make the
exposure larger. What it must not do is make it larger, which is #33's rule
about what this layer may call and #98's test of that rule.

**A legitimate user who fills the store.** #59 holds request sizes and rates at
the edge, which is the outside case. An authenticated person recording versions
until the disk is full is not addressed by anything on this board today. #44
keeps large binary parts off the growth path, which changes the slope and not
the outcome. Accepted, and recorded here rather than in a design, because the
answer is a quota and a quota is a product decision nobody has made.

## Every security decision on this board, traced

The list is derived rather than typed, so it cannot drift against the tracker:

    gh api 'repos/iderex/reissbrett/issues?state=all&labels=security&per_page=100' --jq '.[] | select(.pull_request | not) | "\(.number) \(.title)"' | sort -n
    18 Add the static analysis gate for the chosen language
    21 Lock the dependency graph and make the restore reproducible
    22 Produce a software bill of materials on every build
    27 Add the security policy and a private reporting route
    37 Keep the modelling kernel out of the process that listens on a socket
    49 Accounts, sessions and authentication
    50 The permission model, per project and per document
    59 Rate limits and request sizes at the edge
    81 Handle secrets so they cannot reach a log or a bug report
    89 Add a code scanning gate for this language
    90 Add a second analyser with a different lens
    94 Fuzz the surfaces that take bytes from strangers
    96 Hold the supply chain: pinned actions, a lock file that cannot drift, signed releases
    97 Ask for verified signatures on the protected branch
    104 Write the threat model
    113 Sign and publish the release artefacts
    116 Protect the connection between the client and the server

Fifteen of the seventeen are named above, against T1 to T6 or against N3. Two
are not, and both are justified here rather than reconsidered.

#104 is this document, which defends against nothing and exists so the rest can
be checked.

#27 is the private reporting route, and it traces to no threat on the list on
purpose. It is not a control against an attacker; it is how this project finds
out that one of the controls above did not hold. Writing it against a threat
would be claiming that a reporting route stops somebody, and it does not stop
anybody.

Three decisions carry security weight without carrying the label, and they are
named here so that the derived list above is not read as the whole set. #45
decides what is never stored, which is the strongest control there is against
the confidentiality item and works by not holding the data. #82 logs what
happened without logging what was modelled, which is the same idea applied to a
different file. #86 is telemetry, and what it carries is an absence rather than
a default. All three are labelled for where they land rather than for what they
defend, and each traces to the confidentiality item rather than to a threat
actor, which is why they read as legal or operational issues on the tracker.

The sentence about #86 said something weaker when this document landed: that
telemetry is kept off unless an operator switches it on. That is not the
position. The decision on #1 is that there is none at all, with nothing
collected, nothing sent and no opt-in, stated there without exception and
without a later review:

    gh api repos/iderex/reissbrett/issues/1/comments --jq '.[] | select(.created_at=="2026-08-08T21:10:03Z") | .html_url'
    https://github.com/iderex/reissbrett/issues/1#issuecomment-5228180864

The difference belongs in this document rather than only on #86, because the two
are different controls. A default is a setting, and a setting is something an
operator has to find, something a later release can move, and something this
document would have to trace to whoever can change it. A feature that is not
built has none of those, and the confidentiality item is defended here by the
second one.

What this correction does not do is turn the weaker sentence into an assurance.
Nothing here was measured. There is no software to observe, and #86's own
condition asks for the outbound requests of the delivery form to be enumerated
by watching it run, including the ones made by components this project did not
write. Until that happens the paragraph above records a decision and not a
behaviour, and #102 is where the enumeration is owed rather than here.

## What this document does not settle

It does not decide the permission model, the authentication scheme or the
transport. It names the issues that do and it is what those issues argue
against.

It measures nothing. Every claim above is about what is planned. When #37, #94
and #116 land, what they actually refuse is measured there, and a threat whose
control turns out not to bite comes back here rather than staying addressed on
paper.

It does not cover the operator's own deployment. What an operator exposes and to
whom is #103's document and their decision, and this model assumes the bad case
rather than the documented one.
