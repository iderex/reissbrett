# What may appear in a log, and what may not

Issue #82. Taken on 2026-08-27.

An operator needs to know what the server did. Nobody needs the geometry, and the
claim the readme makes about where models stay stops being true the moment a
model's contents appear in a file that gets sent to somebody for support.

## Nothing here has been built

There is no server and no log line. The tree carries no code in any language:

    git ls-tree -r --name-only origin/main | sed -n 's/.*\.//p' | sort -u | tr '\n' ' '
    gitattributes md txt yml

So every sentence below is a requirement. Nothing here reports what any software
does, and the test that would hold it is this issue's second condition and is
code.

## The claim this exists to keep true

    git grep -n 'never leave' origin/main -- README.md
    origin/main:README.md:3:A distribution and orchestration layer over the open modular CAD suite that adds a guided path through one workflow (sketch to feature to drawing to CAM), with team collaboration and versioning. It runs on your own hardware; the models and the project history never leave it.

`docs/threat-model.md` places this with the two controls it works like:

    git grep -n 'logs what' origin/main -- docs/threat-model.md
    origin/main:docs/threat-model.md:186:the confidentiality item and works by not holding the data. #82 logs what

The control is not holding the data. A log that never carried the geometry cannot
leak it, and that is a stronger position than one that redacts, because redaction
fails on the field nobody thought of.

## The rule

A log record is an allowlist of fields. A value reaches a log because a field
permitting it was written down, and never because it was in a structure that
happened to be printed.

The permitted fields are the event, the time, the level, the actor, the subject,
the outcome, a size, a duration, and an identifier that correlates records of one
operation. Nothing else.

The refused fields are model content, document contents, anything derived from
geometry beyond what an event needs to be meaningful, and any value #81 marks as a
secret.

Allowlist rather than denylist, and that is the decision rather than a detail. A
log built by collecting a structure and removing what is recognised is the second
approach #81 refuses for the support bundle, one artefact over. It fails the same
way, on the field somebody added last week, and it fails silently.

What this costs, stated because it is not free. A field that turns out to be
needed is a change rather than a print statement, so the first weeks of debugging
are slower than they would be with a structure dump. That is the price of the
property and it is paid knowingly.

## Document names, which is the hard case

Names are recorded. A document name is often the part number, which is business
information, and this decision is taken here with its reasons rather than left to
drift.

The reason is that the question an operator brings to a log is what happened to
this document, and a log that cannot answer it is a file nobody reads. Every other
permitted field above is useless without a subject.

The alternative was considered and refused. Logging a stable identifier and
resolving it to a name through this software gives the operator an opaque file at
exactly the moment the software is not running, which is when a log is read. It
also moves the mapping into the store, so a log and the thing that explains it are
two artefacts that can be separated, and the one that leaves the building is the
one that is useless.

Where the risk actually is: a log that leaves the operator's building. Under the
decision that this project runs self-hosted only, a log sits on the operator's own
host, and the route out is a support bundle somebody sends. That is #81's
allowlist rather than this document's rule, and the two are the same shape on
purpose.

No switch is offered for this. A setting that removes names produces a log that
cannot answer the question it exists for, and the operator who turns it on
discovers that during an incident rather than before one. If an operator asks for
one, it is configuration and belongs in #80, decided against their case rather
than against this paragraph.

#103 is where an operator is told that names are in the log, and that is a
condition of this issue rather than a courtesy.

## The levels, and what each one means

Four, and the meaning is what an operator filtering by one is entitled to expect.

**error**: an operation did not happen and somebody has to do something. Every
error names what failed and what the next step is, which is the same rule #64
applies to what a person sees.

**warning**: the operation happened and something about it will stop working. A
message emitted on every operation is not a warning, whatever it says. This is the
level that rots first, because a warning nobody can act on trains an operator to
filter the level out, and then the one that mattered is filtered out with it.

**info**: an operation happened. This is the level the events above are recorded
at and it is the level an operator runs at.

**debug**: what the software was doing between operations. Off by default. It is
still bound by the rule above: debug is not a licence to print a structure, and a
level is not a permission.

What these are called in whatever library the server ends up using is code's, and
the mapping is stated where that choice is made. Naming a library here would
decide the means in a document that has no business deciding it.

## The interaction with #81

The two rules meet at one place and it is worth stating from this side rather than
leaving it to be inferred.

No permitted field above may carry a secret. The one that could is the subject,
where a subject is something #81 marks, and the rule is that a marked value is
never a subject: it is named by what it is rather than by its value.

A secret is a type whose printed form does not contain the value, which is #81's
first condition. This rule does not rely on that. The allowlist is what stops a
secret reaching a log, and #81's type is what stops one reaching everything else,
so a failure of either one is not a failure of both. Two controls that fail
together are one control with a longer description.

## What this does not decide

The log format, the destination, and whether records are lines or structures.
Those follow the means in #14 and the configuration in #80.

Retention. How long an operator keeps a log is theirs, and what #103 tells them
about it is #103's.

The audit trail. #58 is a different artefact with different rules, a different
retention answer and a different reason to exist, and nothing here applies to it.
A log is what the server did; the trail is who changed what.

Metric labels. #84 has its own constraint on labels derived from model content,
and it is stated there rather than borrowed from here.

What an operator may see on their own host. That is N1 in the threat model and is
outside this rule:

    git grep -n "N1. The operator" origin/main -- docs/threat-model.md
    origin/main:docs/threat-model.md:103:**N1. The operator's own administrator.** Somebody who runs the server can read

## What refuses any of this today

Nothing. PROSE, NOT ENFORCEMENT, and the issue that owes the mechanism is this
document's own, #82, whose second condition asks for a test driven by a fixture
with recognisable geometry that asserts no model content reaches a log.

No record in `.github/invariants.txt` can carry it instead. Every rule there is a
pattern handed to `git grep`, and a field written into a log record matches no
pattern that a permitted field does not.

## Revisiting this

The document names decision is the one with a condition attached. If an operator
states a case where a part number in a log is a problem they cannot solve by
controlling the file, that is the configuration route named above and it is
decided against their case.

The level meanings are revisited when there is a log to read. A level whose
records nobody acts on is the warning failure above arriving, and it is a
measurement rather than an argument.
