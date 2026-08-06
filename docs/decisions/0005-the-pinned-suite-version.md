# The version of the suite the first release stands on

This is the decision behind issue #5. A layer over another program has to say
which version of that program it stands on, or every bug report is
unanswerable.

## The decision

The first release stands on FreeCAD 1.1.3.

The bundle in #11 carries exactly that version and no other, so for anyone using
the product as delivered the pin is a property of the artefact rather than a
request.

The extensions route in #11 supports the 1.1 series: 1.1.0 and later, up to and
including the newest patch release of that series that the compatibility harness
in #34 has actually been run against. A version outside that range is refused at
load rather than tolerated, and #35 is what does the refusing.

No weekly build is supported, in either route.

## What the pin is read against

The tag and its date, and the newest four stable releases, read on 2026-08-06
while this document was being written:

    gh api repos/FreeCAD/FreeCAD/releases/latest --jq '{tag: .tag_name, published: .published_at}'
    {"published":"2026-07-25T04:53:36Z","tag":"1.1.3"}

    gh api repos/FreeCAD/FreeCAD/releases --jq '[.[] | select(.prerelease==false) | "\(.tag_name) \(.published_at)"] | .[0:7] | join("\n")'
    1.1.3 2026-07-25T04:53:36Z
    1.1.2 2026-07-23T15:01:22Z
    1.1.1 2026-04-14T22:22:12Z
    1.1.0 2026-03-25T02:19:44Z

The slice asks for seven and four come back, because the first page of the
releases endpoint holds mostly prereleases. That is the second half of why the
pin exists:

    gh api repos/FreeCAD/FreeCAD/releases --jq '.[0:4][] | "\(.tag_name) \(.published_at) \(.prerelease)"'
    weekly-2026.08.05 2026-08-05T00:46:24Z true
    weekly-2026.07.29 2026-07-29T00:48:02Z true
    1.1.3 2026-07-25T04:53:36Z false
    1.1.2 2026-07-23T15:01:22Z false

A weekly build every week, and a stable release when there is one. A policy of
taking the latest is a policy of standing on something that moves every week,
and #34 would spend its life chasing it instead of testing anything.

## Why one version rather than a range

Pinning makes every bug report answerable. The version is in the artefact, so
nobody has to ask for it and nobody has to trust the answer.

It makes #34 a fixed target. A range turns the compatibility harness into a
matrix, and a matrix costs build time on every commit and adds a way for a test
to be flaky that has nothing to do with the change under test.

It costs the person who is already on a newer build. They install the bundle
and they now have two installations of the suite on one machine, or they take
the extensions route and get what that route gets, which #11 states plainly.
That cost is real and it is paid by exactly the people most likely to be early
users, which is worth knowing rather than discovering.

The range on the extensions route exists because that route has no choice. The
operator's installation is whatever it is, and the only two things this project
can do are work or refuse.

## The floor, and why it is where it is

The floor is 1.1.0. Below it the layer refuses to load and says why.

The floor is not a round number picked for tidiness. The module surface this
project calls moved between the 1.0 series and the 1.1 series, and the move is
readable:

    gh api repos/FreeCAD/FreeCAD/contents/src/Mod?ref=1.0.2 --jq '[.[] | select(.type=="dir") | .name] | length'
    35

    gh api repos/FreeCAD/FreeCAD/contents/src/Mod?ref=1.1.3 --jq '[.[] | select(.type=="dir") | .name] | length'
    33

    diff <(gh api repos/FreeCAD/FreeCAD/contents/src/Mod?ref=1.0.2 --jq '.[] | select(.type=="dir") | .name' | sort) <(gh api repos/FreeCAD/FreeCAD/contents/src/Mod?ref=1.1.3 --jq '.[] | select(.type=="dir") | .name' | sort)
    1d0
    < AddonManager
    7d5
    < Drawing

A module directory present in one series and absent in the next is a surface
this project would have to carry two answers for, on a series it has no reason
to support and no way to test against the committed path. A directory listing is
a coarse instrument and this is not a claim that nothing else changed. It is one
readable difference that is enough on its own.

#35 owns the refusal. What it owes is a message naming the version found, the
range expected and what the person does next, and a refusal at load rather than
a failure later while somebody is modelling.

## The policy for moving the pin

The pin moves when all of these are true, and it does not move on a date.

The compatibility harness in #34 passes against the candidate version, with no
test disabled or skipped for it.

The committed path in #9 has been walked end to end on the candidate version, on
every platform the release covers, and the run is recorded.

The change summary and restore proofs in M3 pass against documents written by
the current pin and read on the candidate, because a version bump that cannot
read what the previous one wrote is a migration rather than a bump, and #46 is
where a migration is handled.

Nothing on the release notes of the candidate version touches a module named in
the delegated list in the shape document without somebody having read what it
touches.

When all four hold, the pin moves and the previous version leaves the supported
range on the extensions route at the same time, so the range never grows by
accident. When one of them does not hold, the pin stays and the reason is
written into the issue that proposed the move.

## Weekly builds

A weekly build is never supported, in either route.

The reason is not that weekly builds are bad. It is that the four conditions
above cannot be met weekly by this project, so supporting a weekly build would
mean supporting something nothing has been run against, which is a promise made
from a version string.

A user on a weekly build is told, at load, that this is a weekly build, that
this project supports the stable 1.1 series, and that the layer will not load
against it. They are not offered a flag to force it. A forced load produces bug
reports nobody can act on and a first impression this project would rather not
make, and the person who genuinely wants to try it can install the stable series
alongside.

This is a refusal rather than a warning, and #35 is where it lives. It is the
same code path as the floor, with a different reason in the message, because a
person who is told what is wrong and what to do about it should not have to care
which branch of a check produced the sentence.

## The relationship to the delivery form

#11 chose the bundle as the product and kept the extensions route alive with a
stated reduction in promise. That choice is what makes this document's two
answers coherent rather than contradictory: a single pinned version where this
project controls the installation, and a narrow range where it does not.

If the delivery form ever changes, this document changes with it in the same
landing. A pin the delivery form contradicts is worse than no pin, because it
reads as a supported configuration to everyone except the people who know it is
not.
