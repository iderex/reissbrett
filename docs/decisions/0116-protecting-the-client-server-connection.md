# Protecting the connection between the client and the server

The claim this project rests on is that the models and the project history stay
on the operator's host. From M4 onward that host is several machines on a shop
network, and the designs travel between them. This document decides what
protects that traffic, how the client knows which server it is talking to, and
what happens when the answer changes.

It decides the client side. The server's own configuration is #80, the operator
meets it in the compose example in #109, and who a person is once the connection
stands is #49 rather than this.

Nothing in the tree covered this before. The commit is named rather than left
as `origin/main`, because the mainline now carries this document and the command
finds it:

    git grep -niE 'TLS|transport protection|fingerprint' e38c0cfe9f9186cd10aaa362dda3a054e991d46d -- docs/ README.md .github/ ; echo "exit=$?"
    exit=1

## The decision

Three parts, and the third is the one that decides whether the first two are
worth anything.

**Every connection is protected, and there is no other kind.** The client speaks
TLS and nothing else, at 1.3 or above, to a server on the far side of a building
and to a server on the same bench. There is no plaintext mode, no opportunistic
upgrade from one, and no configuration value, command line flag or environment
variable that produces one. A build that cannot negotiate 1.3 with the server
fails to connect and says so.

**The server is identified by a public key the client records the first time it
connects.** How the client comes to accept that key on the first connection has
two routes, in the order below. After the first connection, the recorded key is
what the client checks against, and nothing widens it back to a route.

**A recorded identity that changes is a refusal.** Not a warning, not a prompt
with a continue button, and not a preference somebody can pre-answer. The client
stops, names what changed, and says what the person should do.

## Why the unprotected connection is not offered even as an option

The argument against offering it is not that plaintext is worse. It is that the
option is the attack.

A client with a plaintext mode has a setting whose value somebody can be talked
into changing, and the person who would be talked into it is a machinist on the
phone with somebody who sounds like support. A shop network is not a trusted
network because it is small: it carries a machine controller, a laptop that goes
home at night, and whatever a visiting contractor plugged into a port.

The cost of refusing the option is real and lands on setup rather than on
running. An operator who cannot produce a certificate cannot get the client to
connect at all, where a plaintext fallback would have let them work today and
fix it later. That is the trade this document takes, and it is taken because
"work today and fix it later" is how a permanent state begins.

## How the client accepts a key on the first connection

Two routes, tried in this order, and the first one that answers is the answer.

**The workstation already trusts the issuer.** If the server's certificate
validates against the trust the workstation already carries, the client accepts
it without asking anybody. This is the route that costs a person nothing, and it
covers both a shop that runs its own authority and one that has a certificate
from a public one. The client still records the key underneath, so the identity
is pinned from that moment on and a later change is caught even though the
issuer would still vouch for it.

**Nobody vouches for it, so a person compares it once.** The client shows a
fingerprint of the key and asks the person to compare it against the one the
server printed when it first started, which is what this document asks of #110.
Accepting is an act, the fingerprint is shown in a form somebody can read aloud
over a phone, and the client records the key. Declining leaves nothing recorded.

The order matters and it is not an implementation detail. Putting the trust
store first means the shop that has an administrator never sees a fingerprint,
and the shop that does not is the only one paying the comparison cost. Putting
the comparison first would charge every operator for the weakest case.

What the client never does on a first connection is accept silently with nothing
to compare against. That is the failure mode that makes trust on first use worth
arguing about, and this document does not take it.

## Why the identity is the key and not the certificate

A certificate expires and gets reissued, and an operator who reissues one looks
exactly like an attacker if the client pinned the certificate. That would make
the refusal fire on the most ordinary maintenance there is, and a refusal that
fires on routine work is a refusal people learn to click through.

Pinning the public key removes the common case: a certificate reissued with the
same key is the same identity, the client says nothing, and the person does
nothing. What is left is the case that genuinely deserves a stop, which is the
key changing.

The obligation this puts on the operator's documentation is that renewing with a
new key costs a visit to every workstation, and it should not be discovered on
the day it happens. Where that sentence is written down is not settled here.
No issue on this board owes the operator's certificate lifecycle: #109 is the
compose example, #112 is upgrading this software rather than rotating a key, and
#103 is what the operator's data protection documentation says rather than what
they have to do. That gap is named here rather than assigned from here.

## What the client says when the identity changes

The refusal names three things and no more.

What it expected, as the recorded fingerprint. What it got, as the presented
one. And the sentence that this project cannot tell an operator who replaced the
key from somebody standing in the middle of the connection, because it genuinely
cannot, and saying otherwise in either direction would be the assurance this
board refuses.

What the person should do is one instruction: ask whoever runs the server
whether the key changed, and compare the fingerprint against what that server
now prints. Not "if you trust this, continue", because that asks the person to
supply the judgement the client just said it could not make.

The wording is #63's to fix and the message is in the words of the task rather
than the words of whatever library raised, which is #64's rule applied to this
message.

## What it costs the operator

Stated here rather than left to be met.

Somebody has to produce a certificate for the server before anybody can use it.
For a shop with an administrator that is routine. For a shop without one it is
the hardest single step in the whole installation, and #109's compose example
and #110's first run are where it is either made bearable or not.

A workstation without the issuer in its trust store costs one fingerprint
comparison per workstation, done by a person, once.

A key change costs a comparison again on every workstation, and until it is
done, that workstation cannot work. There is no route that skips it.

None of these is measured. There is no client, no server and no operator, and
this paragraph is what the decision is expected to cost rather than what it was
observed to cost.

## The alternatives, and why each was rejected

**Plaintext on a local network.** Rejected because the network is not the
boundary. The traffic carries the models, and a shop network is a place where a
laptop that was somewhere else last night is plugged in.

**Plaintext behind an operator-set override.** Rejected in the section above.
The override is the only setting anybody would ever be talked into changing.

**Opportunistic protection, using it when the server offers it.** Rejected
because a connection that was stripped and one that was never offered protection
look identical to the client, so the property it provides cannot be relied on by
anything, including #102's statement.

**Requiring a certificate from a public authority.** Rejected because it forces
a publicly resolvable name and reachability from outside the shop onto a server
whose whole point is that it does not leave the shop. It would push operators
toward exposing the server or running split horizon naming, and both are worse
than the problem.

**A private authority as the only mechanism.** Rejected as the sole route
because it puts a certificate authority on a machinist. It is not rejected as a
route: it is the first of the two above, and a shop that runs one gets the
quieter path.

**A fingerprint compared once, on its own, with no trust store route.** Rejected
because it charges every operator, including the ones who already solved this,
and because the comparison is the step most likely to be performed carelessly
when it is performed constantly.

**Client certificates as the answer.** Not rejected, and not this document's
decision. That is authentication of the person or the workstation, which is #49,
and it sits on top of the connection this document protects rather than
replacing it.

## Where the client keeps what it trusts

One file on the workstation, holding the recorded key per server the client has
connected to.

It is not a secret. It is a public key and a name, and losing it costs a
comparison rather than a compromise. It is still covered by #81's rule, for a
different reason than a secret is: the file says which servers this workstation
talks to, and that belongs to the operator rather than in a support bundle
somebody attaches to a bug report. #81 is where that is checked, and this
document is what it is checked against.

It does not belong in #58's audit trail, and that is worth saying because it
looks like it might. That trail answers who changed a part and when, on the
operator's server, and it says in its own words that it must not become a
general event log. A workstation recording which key it accepted is a fact about
one machine rather than about the project, and putting it there would be the
first step toward the log #58 refuses to be.

## What this imposes on other issues

#48's contract describes messages carried over this connection and names no
unprotected variant of itself.

#102 counts this connection among the ones it enumerates, with the protection
above, rather than treating it as internal and skipping it.

#110 prints the server's key fingerprint in its first run output, in the same
form the client shows it, so that the comparison has something to compare
against that did not travel over the connection being checked.

#109 points at this document for the client side and does not restate it.

#80 validates the server's own certificate configuration at startup and fails
closed, which is that issue's rule applied to this input.

#31 covers the tests: every test this decision generates runs with no display
and without elevation, and a test that needs a real certificate authority is not
a reason to reach for either.

## What this does not settle

It does not decide authentication. Who a person is, and what they may do, are
#49 and #50.

It does not decide the protocol carried over the connection, which is #48.

It does not choose a library, a certificate format on disk, or a fingerprint
encoding. Those are decisions for the code, made under the means recorded on
#14, and naming them here would fix them before anything has been built against
them.

It measures nothing. There is no client and no server, so every sentence above
is a decision and none of it is an observation. What the client actually does
when it is offered an unprotected connection is #116's second condition and is
proven by a test that offers it one, not by this document.

PROSE, NOT ENFORCEMENT, and the mark is not the terminal kind. Every part of the
decision above is a property of code that does not exist yet, and #116's own
conditions are where each becomes refusable: the refusal of an unprotected
connection is proven by a test that offers one, and the refusal on a changed
identity by a test that asserts a refusal rather than a warning. Until those
land, what stands behind this document is the document.
