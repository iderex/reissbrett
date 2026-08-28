# The architecture rules

Issue #98. Written 2026-08-27.

Every rule about structure this board has decided, with the issue it comes from
and the document that holds it, in one place, so that a rule whose source
decision was revised is found rather than left enforcing something nobody
believes any more.

## Nothing here is enforced

No rule below is a test. There is no code in any language for a test to read:

    git ls-tree -r --name-only origin/main | sed -n 's/.*\.//p' | sort -u | tr '\n' ' '
    gitattributes md txt yml

This list is therefore what #98 owes, written before the tests rather than after
them, in the same order `docs/hardware-harness.md` took for its own rule. A reader
who takes a row here for a control is reading it wrong, and the row says so by
sitting under this heading.

## What a row is, and what it is not

A row names the rule in one sentence, names the issue that decided it, and names
the document that argues it. The document is the authority. Where this list and a
decision document disagree, the document is right and this file is stale.

A row is not a restatement of the argument. The reason a rule exists is in the
document, and copying it here would put the same reasoning in two places that can
drift.

## The division with #91

A rule a pattern search decides belongs to `.github/invariants.txt`, because it is
cheaper: one record, two fixtures, no units required. A rule about what imports
what, or what calls what, needs the language's own view of the code and belongs
here.

No rule is enforced in both places today, and that is shown rather than asserted.
These are every record in the invariants file:

    git grep -n '^id:' origin/main -- .github/invariants.txt
    origin/main:.github/invariants.txt:79:id: no-absolute-home-path
    origin/main:.github/invariants.txt:85:id: no-display-server-in-workflows
    origin/main:.github/invariants.txt:91:id: no-suite-source-header
    origin/main:.github/invariants.txt:97:id: no-suite-source-clone-in-workflows

None of them is a rule below. The two that come closest are the two halves of the
copy-or-patch entry, and `docs/decisions/0004-where-the-layer-attaches.md` splits
that rule between the two files by name rather than leaving the boundary to be
guessed:

    git grep -n "is #98's half" origin/main -- docs/decisions/0004-where-the-layer-attaches.md
    origin/main:docs/decisions/0004-where-the-layer-attaches.md:190:What the language's own view decides, which is #98's half. A unit of this project

The other direction is shown by the same file. Nothing in the invariants file
reaches any of the rules below:

    git show origin/main:.github/invariants.txt | grep -cE 'purgeTouched|internal name|private path|listening process'
    0

## The rules

### A1. The listening process does not reach the suite

No unit belonging to the process that listens on a socket may import, link
against, or load the suite's interface, directly or through anything it imports.

From #37. Argued in `docs/decisions/0037-the-process-boundary.md`, which states it
in this form under its own section on a rule a test can refuse.

### A2. The worker does not reach the network

No unit belonging to the worker may open a listening socket or make an outbound
connection.

From #37, and from the same document as A1. It is the other direction of the same
boundary, and it is a separate row because a test can pass one and fail the other.

### A3. The version store does not reach the suite

No file under the version store may import the suite's interface.

From #40. Its fifth condition states it, and there is no decision document, so the
issue is the authority until one exists.

### A4. Interface decisions are not made in the drawing layer

No decision about what the interface does is made inside the layer that draws it.

From #66. Its first condition states it. The rule exists so that the behaviour is
testable with no display attached, which is that issue's subject.

### A5. Every operation passes the permission decision point

No operation reaches its effect without passing through the single function that
answers whether the person asking may do it.

From #50. Argued in `docs/decisions/0050-the-permission-model.md`, which names #98
as what refuses a second route:

    git grep -n 'refuses an operation that does not pass' origin/main -- docs/decisions/0050-the-permission-model.md
    origin/main:docs/decisions/0050-the-permission-model.md:298:#98 refuses an operation that does not pass through the decision point, and #20

### A6. A test uses a fixture rather than generating a document

No test generates a document at test time in place of reading a fixture.

From #36. There is no decision document; the issue is the authority.

### A7. No unit reaches into the suite's private surfaces

No unit of this project reaches into a path the suite declares private, or into a
header, rather than the surfaces `docs/extension-surface.md` lists.

From #4, argued in `docs/decisions/0004-where-the-layer-attaches.md` and restated
as a call rule in `docs/extension-surface.md`. This is the import half of the
copy-or-patch rule. The pattern half is #91's, and the residual neither half
reaches is named below.

### A8. No interface string is written inline

No string a person reads is written into the code rather than into the
externalised set.

From #118, and it is the one row here that is conditional rather than settled:

    git grep -n 'architecture rule becomes a test' origin/main -- docs/decisions/0118-the-interface-language.md
    origin/main:docs/decisions/0118-the-interface-language.md:260:#98 is where an architecture rule becomes a test, and the rule that no interface

That document says the rule lands here if the check #118's own third condition
owes lands here rather than beside it, and that has not been decided. The row is
listed rather than left out, because a rule waiting for a home is found by
somebody reading this file and is not found by anybody reading nothing.

## What no rule here reaches

Two things, and both are named so that this list is not read as the whole of what
protects the boundary.

`0004`'s residual. A file copied out of the suite's source tree with its banner
removed matches no pattern #91 can write, and it is not an import, so it is not a
rule here either. Deciding that case needs a comparison against the upstream tree
at the pinned tag, which is a fetch and a diff. No issue on this board owes one.

The document rules in `docs/extension-surface.md` that are about what a call does
rather than about what a unit reaches. Writing into a document to enrol it,
assigning to an internal name, modifying a file the suite has open, and calling
`purgeTouched` are all legal calls that leave a document in a state it should not
be in. No reading of this tree decides that, which is what that document says of
them, and no mechanism is owed:

    git grep -n '#98' origin/main -- docs/extension-surface.md
    origin/main:docs/extension-surface.md:307:they are written so that #98 can turn the ones about imports into tests.
    origin/main:docs/extension-surface.md:408:#98, which has landed nothing, and nothing in the rules file is about writing
    origin/main:docs/extension-surface.md:417:the copy-or-patch entry named above, issues #91 and #98. The permission list is

## How a rule stays attached to the issue that decided it

Every row names its issue, and every row that has one names the decision document.
The test that enforces a row names the same issue in its own name, so a person who
reads a red test reaches the argument in one step rather than by searching.

Nothing checks that the two agree. A row naming an issue that was closed as
superseded looks exactly like a row naming a live one, and a test whose source
decision was rewritten still passes. What can be decided mechanically is the
narrower half, that a reference resolves at all, and that is #95's:

    git grep -n '#95 owes it' origin/main -- CONTRIBUTING.md
    origin/main:CONTRIBUTING.md:54:Part of this is mechanically decidable and #95 owes it: a command in a fenced

So the link is kept by the row and read by a person. That is the whole of the
answer to #98's fifth condition, and it is weaker than it sounds: this file is
maintained by whoever changes a rule remembering to change it here.

## What this file does not do

It adds no test. Every row is a sentence, and #98's first two conditions are the
tests and their fixtures.

It decides no rule. Each row is sourced to the issue that decided it, and a
disagreement with a row is a disagreement with that issue.

It does not claim to be complete. A rule decided tomorrow and not written here
leaves this file exactly as it is, and nothing goes red.

PROSE, NOT ENFORCEMENT for all of it, issue #98, which owes every mechanism named
above.
