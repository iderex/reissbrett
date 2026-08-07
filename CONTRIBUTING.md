# Contributing

## Running the gate before you push

There is no command that runs this project's gate on your own machine, because
there is no code yet and nothing to run. The checks live in
`.github/workflows/` and they run on GitHub when you push a branch or open a
pull request. What each one refuses is printed by the check itself, and this
file does not repeat it, because a list here drifts against the thing it
describes and the check is the authority for what it refused.

When there is a local command it will be one command, it will be the same one
the workflow calls, and it will be a courtesy that shortens the feedback loop
rather than the thing that stands behind a merge. #19 owes the test harness and
the command that runs it, and #16 owes the build from a clean checkout. Until
those land, a push is the first moment anything is checked.

Whether a check is a condition of the merge is a separate question from whether
it runs, and today the answer is that it is not: the ruleset on `main` requires
a pull request and names no status check. `.github/required-status-checks.md`
holds the list and the state, with the commands that read it.

## No work without an issue

Every change starts as an issue and lands as a pull request. Direct pushes to
`main` are refused by the ruleset.

An issue says what is wrong, what the evidence is, and what done means. Where
the evidence is a number, it carries the command that produced it. An issue
without a done condition is a wish, and nobody can tell when it is finished.

PROSE, NOT ENFORCEMENT, issue #92. Nothing in this repository reads a pull
request body or refuses one that names no issue. The template asks for the
issue, and a template is a prompt rather than a gate.

## Every asserted fact carries the command that produced it

Run the command at the commit you are pushing and against the reference the
reader will have, not against your working tree. A number quoted from a working
checkout and reported as though it came from the mainline is the defect that
costs most here, because it reads exactly like a measurement.

Where a claim cannot be backed by a command, write it as a claim and say so.
"Verified", "not measured" and "not evaluated on this route" are three different
statements and they are not interchangeable.

A disclosure that something was not done stays a disclosure. If a body says a
change had no second reader, no later edit turns that into a sentence saying it
did.

PROSE, NOT ENFORCEMENT, and the two halves are marked for different reasons.
Part of this is mechanically decidable and #95 owes it: a command in a fenced
block that does not parse, a link to a file that is not in the tree, a reference
to an issue that does not exist. Whether a pasted output is the one that command
actually produced is not decidable from anything in this tree, no mechanism is
owed for it, and the review is where a wrong one is caught.

## A rule nothing refuses is marked where a reader will see it

A sentence in a document is not a rule. It is an explanation of one. If you
cannot name the check that refuses a thing, mark it `PROSE, NOT ENFORCEMENT`
at the point the rule is stated, and name the issue that owes the mechanism, or
say plainly that no issue does.

Some rules can have no mechanism at all. A rule about how the work was done has
nothing in the tree for a check to read, and marking it does not make it
enforceable. There the mark is the end of the matter rather than a placeholder
for a check that is coming, and it should say so.

## The means check

Before an artefact is built, whether the chosen means fits is argued and the
answer is written down, in the issue or in the pull request body. The means is
the language, the format, the tool or the runtime. Every time, and never carried
over from habit, because a means that was right for the last artefact is an
assumption about this one.

What the argument answers: whether the means can carry a property a machine
refuses, proof that executes and a claim that carries its command. Whether
anything outside this repository forces it, and whether that force is real and
held to its smallest surface. Whether it adds a language, a runtime or a
dependency this tree does not already carry, and whether that cost is paid
knowingly. Whether the artefact is testable by the harness this board plans, or
needs a parallel apparatus nobody will maintain.

What can be seen from outside is whether the question was asked, and only
because the answer is written down. Whether the answer was right is a judgement,
and the review is where a wrong one is caught.

PROSE, NOT ENFORCEMENT, issue #92, which owes a check that the sentence is
there. Whether the answer in it was right is the judgement half, and nothing
here decides that.

## The pull request body is the transcript

Everything about a change goes in its body. If the body is wrong, incomplete or
out of date, edit the body. A comment underneath a pull request is not where
anybody looks, and it is not where a refusal belongs either: the reason a change
is sent back goes in the body, not under it.

PROSE, NOT ENFORCEMENT, and terminal. This is a rule about how the work is done,
so there is nothing in the tree for a check to read and no mechanism is owed.

## Sign off, and sign

Sign off every commit with `git commit -s`. The trailer is the assertion in
`./DCO`, and it has to match the commit's own author name and address. This one
is refused by a machine: `.github/workflows/dco.yml` walks every non-merge
commit in the pull request and reds on the first one that is missing it.

Sign your commits as well, and treat a signing failure as a stop rather than an
obstacle to get around. The way around it is one word, `--no-gpg-sign`, and a
commit that took it builds and reviews exactly like one that did not, which is
what makes it worth naming here.

PROSE, NOT ENFORCEMENT for the signature. The ruleset on `main` does not require
verified signatures, so nothing refuses an unsigned commit today. #97 is the
issue that owes it. The sign-off check is a different case: it runs and it goes
red, but it is not yet a required check, so a red DCO run does not by itself
stop a merge. `.github/required-status-checks.md` is where that changes.

## Commit messages

Say what changed and what failure it prevents. Where you are correcting
something, say what was wrong and how it was found. One topic per commit and per
pull request, because a commit carrying two unrelated changes has a message
describing one of them.

Reference the issue the change belongs to.

PROSE, NOT ENFORCEMENT, issue #92 for the part a machine can decide, which is
whether the commits carry what this file says they carry. Whether a message
describes what changed, and whether two changes in one pull request are really
one topic, are judgements and no check here makes them.

## What is written in tracked files

English. No attribution to a tool, and no generated-by markers, in anything
tracked.

Tracked text is stored as UTF-8 without a byte order mark and with LF line
endings, and `.gitattributes` is what fixes that on the way into git. This one
is refused by a machine: `.github/workflows/text-determinism.yml` reads how each
tracked file is stored rather than how it looks in your working tree.
`.github/text-determinism-exemptions.txt` is the only way a path is skipped, and
adding a path to it is a change somebody reviews.
