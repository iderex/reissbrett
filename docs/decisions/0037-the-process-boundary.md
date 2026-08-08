# The process boundary between the server and the modelling kernel

This is the decision behind issue #37. The collaboration server accepts
connections from a network, and the documents it holds are parsed by somebody
else's C++ geometry kernel. This document says how those two facts are kept out
of one process, what the boundary is, what crosses it, and what happens when the
thing on the far side of it dies or stops answering.

The threat this addresses is T3 in the threat model, which already names this
issue as what addresses it:

    git grep -n 'Addressed by #37' origin/main -- docs/threat-model.md
    origin/main:docs/threat-model.md:73:project's whole job is to accept documents. Addressed by #37, which keeps the

The threat model also records the case this does not address, which is a
malicious document opened in the desktop process, where the kernel and the
document are in one process by construction. Neither is restated here.

## The rule

The process that listens on a socket never loads the suite.

Work that needs the kernel is done by a separate process, started for one job
and ended when the job is over, which this document calls the worker.

Both halves are needed. The first on its own is satisfied by a server that
shells out to something that then listens on its own socket, and the second on
its own is satisfied by a worker that is really the server with a different
name.

## Why the lever is where the kernel runs, rather than what it does

A geometry kernel is large, old, written in C++ and fed complex structured input
that arrives from outside:

    gh api repos/FreeCAD/FreeCAD --jq '.language'
    C++

This project implements none of it and cannot fix a defect in it. The shape
decision says so in the list a later issue is placed against:

    git grep -n 'This project delegates the geometry kernel' origin/main -- docs/decisions/0003-a-layer-over-the-existing-suite.md
    origin/main:docs/decisions/0003-a-layer-over-the-existing-suite.md:101:This project delegates the geometry kernel, the constraint solver, the

So the only control this project actually holds over that code is where it runs
and what it can reach from there. Auditing it is not available, patching it is
#4's and entry 3 of #1's question rather than this document's, and refusing to
accept documents is not available either, because accepting documents is the
product.

## The first release may need no worker at all

This is the part of the decision that costs least and is easiest to lose later,
so it is written down as a decision rather than left as an accident of the first
implementation.

Recording a version does not need the kernel. #6 records a version by unpacking
the container the suite wrote and storing its entries under names derived from
their bytes, and it is explicit about which half is the authority:

    git grep -n 'The parts store is the authority for restore' origin/main -- docs/decisions/0006-what-a-version-of-a-model-is.md
    origin/main:docs/decisions/0006-what-a-version-of-a-model-is.md:14:description kept alongside it. The parts store is the authority for restore.

Unpacking an archive and hashing its entries is not modelling. So the server can
record and restore a version, answer a lock request under #7's model, apply
#50's permissions and write #58's audit trail, without the kernel being present
on that machine at all.

The readable description that #6 records alongside the version is produced on
the desktop, by the client, where the kernel is already loaded because that is
what the suite is, and it is uploaded with the version. That is the decision.

It has a cost and the cost is bounded by #6's own sentence above. The server
cannot check that the description matches the parts, so a client that uploads a
wrong one produces a wrong summary in #42 and a wrong thing to look at in #57.
It cannot produce a wrong restore, because the description is never what is
restored from. A wrong description is a display defect and a wrong store is a
corruption, and the boundary is drawn so that the untrusted side can only cause
the first.

The worker therefore exists in this document as the shape that anything
server-side needing the kernel must take, rather than as a component the first
release is committed to building. If M3 or M4 turns out to need one, the rest of
this document is what it is built to.

## What crosses the boundary

Into the worker: a job description naming what is to be done, and a working
directory the server has already filled with exactly the files the job needs.
Nothing else.

Out of the worker: whatever it wrote into that working directory, an exit
status, and a diagnostic stream. Nothing else.

What never crosses, in either direction: a network connection, a credential or
token, an open handle on the store or its index, and the ability to name a file
the job description did not name. The worker does not ask the server for
anything during a job, because a worker that can ask is a worker that can be
made to ask for something else.

The encoding of the job description, the mechanism that starts the process and
the shape of the diagnostic stream are all deliberately absent. #14 has not
named the language or the runtime for either surface, and choosing an encoding
here would be choosing one on its behalf.

## What the worker may reach

A working directory created for the one job and removed after it.

The stored parts the job names, reassembled into that directory by the server
before the worker starts. The worker never reads the store, so an attempt to
reach a part the job did not name is not a permission failure inside the store;
it is a file that is not there.

No network. This document states that as a requirement. Nothing has established
it, and #37's third condition asks for the record to say how it was established
rather than to assert it, so this sentence is the requirement and not the
evidence.

Nothing else. In particular not the operator's own documents, not the server's
configuration, and not the credentials the server holds.

## When the worker dies in the middle of a job

The server treats it as a job that did not happen. Nothing the worker wrote is
admitted to the store.

That is possible only because of the rule above that the worker writes into its
own working directory and never into the store. The server is the only thing
that writes to the store, and it does so after the worker has exited
successfully.

This is worth stating as the reason rather than as a consequence, because it is
what makes #37's fourth condition provable at all. If the worker could write to
the store, proving that a killed worker leaves the store in a state #47 accepts
would be a proof about the crash behaviour of a process holding somebody else's
C++ kernel, which is not a thing anybody can prove. Because it cannot, the proof
is about the server's own commit step, which is the interrupted-write guarantee
#47 already owes and already has a test planned for.

## When the worker does not die but stops answering

Every job carries a deadline, set by the server before the worker starts. When
it expires the server ends the process and treats what follows as a death, which
is the paragraph above.

The worker is never asked to police its own deadline. A worker that has stopped
answering is exactly the one that will not notice it has, so a self-imposed
timeout covers every case except the one it exists for.

The deadline is configurable and no number is set here, for the same reason
#51's lease duration is not set in #7's document: a number chosen before
anything has been measured is a number somebody will quote later as though it
had been.

## The rule in a form a test can refuse

#37's second condition asks for the rule in a form #98 turns into a test, and
#98 refuses an import of the suite from the listening process. Three rules, and
the first is the one the issue names:

No unit belonging to the listening process may import, link against, or load the
suite's interface, directly or through anything it imports.

No unit belonging to the worker may open a listening socket or make an outbound
connection.

The only route from the listening process to the suite is the job mechanism
above, so a call site that starts a process is a call site the layout puts in
one place and a test can count.

Each of those is a statement about which units of the tree may reach which, so
each needs the layout to have named the units. #15 owes that layout and its own
body already says it exists to express this separation. This document states the
rules in a form that does not depend on the directory names #15 chooses, and
#15's second condition is where they become concrete. Until then nothing refuses
any of them, and the marker for that is this paragraph rather than a claim
elsewhere that the boundary is enforced.

## What this document does not decide

The layout. #15 names the directories and turns the three rules above into rules
about paths.

The language and runtime for either surface. #14 owes that, and the absence of
an answer is why no encoding, no process API and no transport appears above.

Whether a worker is built for the first release at all. This document says the
first release may not need one and says why, and it says what one has to look
like if it turns out to be needed. Which of those happens is decided by M3 and
M4 when they meet a job that needs the kernel.

The desktop side. On the desktop the kernel and the document are in one process
because that is what the suite is, the threat model accepts that with its
reason, and nothing here changes it.
