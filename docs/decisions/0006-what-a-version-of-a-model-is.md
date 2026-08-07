# What a version of a model is

Everything in M3 and M4 rests on one answer. When somebody says "version 7 of
this part", this document says what is being named, what restoring it
guarantees, and what it does not guarantee.

The study in `docs/decisions/prior-collaboration-attempt.md` records what the
earlier effort did here. This document argues against that record rather than
against memory, and where it departs from it the departure is argued.

## The decision

A version is a list of names of stored parts, plus a derived readable
description kept alongside it. The parts store is the authority for restore.
The readable description is derived from the same version and is never the
thing that is restored from.

Concretely, recording a version does three things. It takes the container the
suite wrote and unpacks it into its entries. It stores each entry under a name
derived from that entry's bytes, so an entry that did not change between two
saves is stored once. It records the version as the ordered list of those
names together with the container metadata needed to reassemble the file byte
for byte. Alongside that, and separately, it records a description of the
feature tree that a person can read, which is what #42 turns into a change
summary and what #57 shows to a reviewer.

The fourth of the four candidates in #6, in the order that issue lists them.

## What restoring a version guarantees

A recorded version, restored on the suite version pinned by #5, produces a
file whose bytes are identical to the bytes that were recorded.

That is stronger than the bar #6 proposed, and it is stronger deliberately. #6
asked for a document that the suite loads, byte-identical in its structured
description and geometrically identical in its shape data. Byte identity of
the whole file is easier to state, easier to test and easier to trust, and it
is what unpacking-then-reassembling can deliver as long as the reassembly is
faithful. It is what #43 tests, and the test is a comparison of the recorded
bytes against the restored bytes, not a comparison of two models.

The cost of taking the stronger bar is carried in one place, and it is worth
stating precisely because it is where the decision could turn out to be
expensive. Reassembly has to reproduce the container exactly, including entry
order and whatever the archive format records per entry. So what is stored per
entry is the entry's stored form as it appears in the container, not the
uncompressed content, and the container's own framing is stored beside the
entry list as one more part. Where some part of the framing turns out not to
be reproducible from what was recorded, the answer is to record more of it
verbatim rather than to weaken the guarantee. The guarantee does not move.

The consequence of storing entries in their stored form is that deduplication
depends on the suite producing the same stored bytes for content that did not
change, rather than on the content alone being unchanged. That is a stronger
requirement than it sounds, and it is measurable rather than arguable. It is
the first thing the condition below asks #38 to measure.

## What a version does not guarantee

A version is not a promise that rebuilding the model on a future suite version
yields the same shape. Rebuilding is the suite's behaviour and not this
project's, and what changes across suite versions is what #34 measures rather
than what this project claims.

A version is not a promise about identity inside the document. Whether the
face somebody selected in version 6 is the same face in version 7 is a
question this project cannot answer reliably, #39 is where the limit is
written down, and #42 is required to say it cannot tell rather than to guess.

A version is not a promise that the file opens in a suite version other than
the pinned one. It restores the bytes that were recorded. What a different
suite version does with those bytes is that suite version's business.

## The alternatives, and why each was rejected

The whole container, stored opaquely, addressed by its own hash. This is what
the earlier effort implemented, and the study records it: one stored object
per save, named by a fresh random identifier rather than by content, with no
part list and no diff. It restores exactly and it is the simplest thing that
can be correct. It was rejected because it deduplicates nothing on data that
is mostly unchanged between saves, and because it makes the readable history
in #42 impossible rather than merely hard. It stays as the named fallback
below.

The container unpacked into parts, with nothing derived alongside. Rejected
because a list of part names is not a history a person can read, and #42, #54
and #57 all need one. Unpacking without deriving anything pays the cost of the
dependency on the container layout and collects only half of what that cost
buys.

The feature tree as the version, with geometry treated as derived and rebuilt.
Rejected because a rebuild is not guaranteed to reproduce the same shape
across suite versions, and a version that does not restore exactly is not a
version. This is the most readable answer and it is the one that fails at the
only thing a version store must not fail at.

## Where this departs from the earlier effort

The study records that the earlier effort stored one opaque object per save,
under a name its own client comments describe as not a hash but a random
identifier, minted fresh even for a file the server already held. This project
departs from that in two ways: the name of a stored part is derived from its
bytes, and the unit stored is an entry inside the container rather than the
container.

The departure is argued rather than assumed. Deriving the name from the bytes
is what makes storing the same content twice cost nothing and makes alteration
on disk detectable, which is #40. Storing entries rather than containers is
what makes a version a small list, which is what makes #41 cheap and #42
possible. Neither is a criticism of what was built there: a service uploading
whole files over a network had different pressures from a store that has to
hold a project's whole history on an operator's disk.

## What this decision depends on, and what happens if the dependency fails

Unpacking somebody else's container means depending on its internal layout.
The suite's document module links a zip library, and the library sits in the
suite's own third party directory at the pinned version:

    gh api "repos/FreeCAD/FreeCAD/contents/src/3rdParty?ref=1.1.3" --jq '[.[].name] | join(", ")'
    .gitattributes, 3Dconnexion, CMakeLists.txt, GSL, OndselSolver, OpenGL, PyCXX, json, lazy_loader, libE57Format, libkdtree, lru-cache, salomesmesh, zipios++

    gh api search/code -X GET -f q='repo:FreeCAD/FreeCAD zipios path:src/App' --jq '.total_count'
    6

Both were run on 2026-08-07. They establish that the container is an archive
and that the archive handling lives in the document module. They establish
nothing about what is inside it, and this document claims nothing about that.
#38 owes that characterisation, and it owes one measurement in particular:
whether saving a document twice with no change produces the same stored bytes
for each entry.

So this decision is taken before its evidence is complete, and the condition
that would move it is written here rather than left to be argued later.

The condition. If #38 finds that a material share of the entries change on a
save that changed nothing, and that the change cannot be attributed and
normalised, then per-entry storage buys no deduplication and the dependency on
the layout is being paid for nothing. Material means more than the container
metadata and the thumbnail: an entry holding shape data or structured
description that moves without the model moving.

What happens then. The parts store from #40 stays exactly as it is, because
content addressing is worth having on its own. What changes is what is stored
in it: the whole container as one part rather than each entry as a part. The
version stays a list of names, of length one. The restore guarantee above is
unaffected, because it was never stated in terms of entries. The readable
description alongside it is what is lost, and #42 would then have to derive
its summary by opening both versions through the runner from #30 rather than
by comparing part lists, which is slower and still possible.

That fallback is a named branch with a trigger somebody can check, not a
hedge. It is written down because taking the decision after #38 would block M3
behind a study that needs the runner from #30, and because the part of the
decision that
#38 could overturn is smaller than it looks.

## What this imposes on other issues

#38 measures the per-entry stability described above and reports it against the
condition stated here.

#40 builds the store this rests on, and the hash choice is #40's to make and
record.

#41 records a version as a list of part names without rewriting the parts that
did not change.

#43 tests the restore guarantee as a byte comparison of the recorded file
against the restored file.

## Revisiting this

Two conditions, both checkable.

If #38 reports per-entry instability of the kind described above, the fallback
in this document applies and this document is rewritten to say so.

If the pin in #5 moves to a suite version whose container is a different
format, this decision is re-argued rather than carried over, because
everything above rests on the container being an archive of separable entries.
