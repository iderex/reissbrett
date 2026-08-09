# Where the layer attaches, and the rule for touching the suite

This is the decision behind issue #4. #3 decided that this project is a layer
over the existing suite. This one decides where that layer attaches, and it
fixes what M5 can reach and what this project has to keep alive for as long as
it exists.

The choice itself is not made here. Entry 3 of #1 collects it as a decision the
plan does not make, and that entry now carries an answer:

    gh api repos/iderex/reissbrett/issues/1/comments --jq '[.[] | .created_at] | join(" ")'
    2026-08-06T02:39:26Z 2026-08-08T21:10:03Z 2026-08-08T22:39:02Z 2026-08-08T23:25:01Z 2026-08-09T04:11:22Z

The answer to entry 3 is in the fourth of those. What follows records it and
works out what it costs. It does not decide it, and #4 asks for that order
rather than the reverse.

## The decision

The layer attaches to the suite's documented extension surfaces, and to nothing
else. This repository carries no copy of the suite's source tree and no patch
against it. A change that cannot be made from an extension surface becomes an
upstream enhancement proposal, or it does not happen in the first release.

The rule is stated with no exception list, and that is the point rather than
severity for its own sake. A rule with a maintained list of permitted exceptions
cannot be refused by a check, because every violation sits one line in that list
away from being legal. What no check can refuse is an intention, and this board
says that about itself in `CONTRIBUTING.md` before it says it about anything
else.

Two things the decision does not say. It does not say this project never
contributes to the suite, since contributing upstream is the route it names for
everything it gives up. And it does not say the suite is never read: establishing
what an interface actually does, which is what #33 and #34 exist for, is reading.
The rule is about what this repository carries.

## The surfaces this project intends to use

Everything read at a tag below is read at the version #5 pins, which is 1.1.3.

The addon package and its manifest. An addon is a directory the suite discovers
through a `package.xml` manifest, and the manifest has a published schema with
its own repository:

    gh api repos/FreeCAD/Addon-Manifest-Schema/contents --jq '[.[].name] | join(" ")'
    .editorconfig .github .gitignore .vscode Assets Docs LICENSE_SCHEMA LICENSE_SCRIPTS README.md Scripts Source package.json

    gh api repos/FreeCAD/Addon-Manifest-Schema --jq '.license.spdx_id'
    Unlicense

This is what #108 and #109 package into, and what the extensions route in #11
delivers.

The addon manager, which is what installs a package for an operator who is not
using the bundle:

    gh api repos/FreeCAD/AddonManager --jq '{license: .license.spdx_id, pushed: .pushed_at}'
    {"license":"LGPL-2.1","pushed":"2026-07-31T22:59:00Z"}

The Python entry points a workbench is loaded through. A module in the suite's
own tree carries `Init.py` and `InitGui.py` beside its Python, and that is the
shape an addon takes as well:

    gh api repos/FreeCAD/FreeCAD/contents/src/Mod/Draft?ref=1.1.3 --jq '[.[] | select(.name | test("^Init"))] | map(.name) | join(" ")'
    Init.py InitGui.py

This is where the guided path in #61 and the workspace state in #62 attach.

The console entry point, which is what a run with no display uses. The suite
builds a second executable beside the windowed one, and the resource files name
them apart:

    gh api repos/FreeCAD/FreeCAD/contents/src/Main?ref=1.1.3 --jq '[.[].name] | join(" ")'
    CMakeLists.txt FreeCADGuiPy.cpp MainCmd.cpp MainGui.cpp MainPy.cpp core-main.dox freecad.rc.cmake freecadCmd.rc.cmake icon.ico

That listing is what the surface is read from. Nothing here has run either
executable, and the runner that will is #30, so the name a build actually
installs is #30's measurement rather than this document's claim.

The parameter surface, which is how a packaged set of settings reaches the suite
without editing anything a person owns:

    gh api search/code -X GET -f q='repo:FreeCAD/FreeCAD GetParameterGroupByPath path:src/App' --jq '.total_count'
    11

This is the surface #62 applies the known starting state through. The code search
endpoint reads the default branch rather than the pinned tag, and the number
above is therefore about the project and not about 1.1.3.

The theme format, for the appearance half of that starting state. It is a
first-party repository, and it carries a manifest of the same kind an addon does:

    gh api repos/FreeCAD/FreeCAD-themes/contents --jq '[.[].name] | join(" ")'
    .github .pre-commit-config.yaml Behave-dark Behave-dark_1_1_Plus Dark-contrast Dark-modern Darker LICENSE Light-modern ProDark README.md images_dark-light package.xml

The translation catalogues, which #118 has to decide against rather than around:

    gh api repos/FreeCAD/FreeCAD/contents/src/Gui/Language?ref=1.1.3 --jq 'length'
    115

What this list is not. It names the mechanisms this project attaches to. What may
be called on each one, function by function, is #33, and this document does not
pre-empt it.

## Where these surfaces are documented, and where that could not be read

One of them has documentation in a repository:

    gh api repos/FreeCAD/Addon-Manifest-Schema/contents/Docs --jq '[.[].name] | join(" ")'
    Development.md README.md Repository.md Versioning.md Workarounds.md

The workbench entry points, the parameter surface and the theme format are
explained in prose on the upstream wiki rather than in a tracked file, and that
prose was not read here. The route available on 2026-08-09 answered with an
access denied page instead of the article, at `https://wiki.freecad.org/Package_Metadata`.

So what is cited for those three is the tree at the pinned tag, which is what any
prose would have to agree with, and the prose itself is missing. Somebody who can
reach the wiki should add the article and the date it was read beside each
surface. This is a gap and it should not be read as anything else.

## What happens to a change that cannot be made from a surface

It is proposed upstream. The process exists, it is public, and it has a written
procedure of its own:

    gh api repos/FreeCAD/FreeCAD-Enhancement-Proposals/contents/FEPs --jq '[.[].name] | join(", ")'
    FEP-0000-template, FEP-0001-process, FEP-0003-release-schedule, FEP-0006-materials-editor, FEP-0007-consistent-language, FEP-0008-project-group-structure, FEP-0010-variant-parts

    gh api repos/FreeCAD/FreeCAD-Enhancement-Proposals/contents/FEPs/FEP-0001-process --jq '[.[].name] | join(" ")'
    README.md assets

Who writes it. The proposal is written and submitted from this project, under the
account `.github/GOVERNANCE.md` names as the one that decides here, so that an
upstream reviewer has one counterpart rather than a queue of individuals. That
stays true when the governance document's own succession section stops being
hypothetical.

What this project does while a proposal is pending, which is the half that gets
skipped. It does one of three things and never a fourth. It reaches the same end
through a surface that already exists and accepts that the result is worse. It
drops the step from the committed path in #9 and says in the interface that the
step is not covered. Or it records the change in the list #61 is required to
produce, and the guided path goes without it.

The fourth thing, which the decision refuses, is carrying the change locally
until the proposal lands. That is a patch with a plan attached, and a plan is not
what the rule is written against.

The timeline belongs to the suite and not to this board, and no issue here may be
written as though a proposal will be accepted. An M5 issue that only works if an
upstream change lands is an M5 issue that is wrong, and it is re-planned rather
than waited on.

## The rule, in a form a check can refuse

The rule has three parts, because they are decided by different means and
collapsing them produces a check that claims more than it does.

No tracked file in this repository is a file taken from the suite's source tree,
whether verbatim or edited afterwards.

No tracked file is a patch, a diff or a series of them whose target is a path in
the suite's source tree.

No build or packaging step fetches the suite's source and changes it before
building. Fetching a release artefact and shipping it unchanged, which is what
#108 does under the delivery form in #11, is not this.

What a pattern search decides, which is #91's half. A tracked file whose path
ends in a patch or diff extension, or a tracked series file, is decided by a
pattern over paths. So is a build step that clones the suite's repository, over
`.github/workflows/` and whatever #16 adds. So is a copied source file that still
carries the header the suite's own files carry, which is the shape a copy takes
in practice, because the header is the first thing a copy keeps and the last
thing somebody thinks to remove. `.github/invariants.txt` already expects this
rule to arrive with this issue, in its own words:

    git show a8a732ac0e195fca2c75db1d26f60d0d62e39ee2:.github/invariants.txt | grep -n 'copy-or-patch'
    42:# arrive with the thing they are about: the copy-or-patch rule with #4, the

What that leaves undecided, stated because the pattern half reads as though it
covers the rule. A file copied from the suite and stripped of its header matches
no pattern. Deciding that case needs a comparison against the upstream tree at
the pinned tag, which is a fetch and a diff rather than a search, and nothing on
this board owes one today. The residual is real and it is where a violation would
actually get through.

What the language's own view decides, which is #98's half. A unit of this project
that reaches into the suite's private headers or its internal modules rather than
the surfaces listed above is an import question, not a text question, and it is
the same shape as the rules #37 and #40 hand to #98. #98's body lists this rule
already, sourced to this issue.

The bar both halves are held to is the one in `CONTRIBUTING.md`: a check ships
with a fixture that violates the rule and is refused, and a near miss that is
accepted. For the header pattern the near miss is the one that matters, because
this repository will quote the suite's licence and its notices in `NOTICE.md` and
in `docs/decisions/0100-the-licence-split-and-the-suite.md`, and a rule that
refuses a document quoting a header is a rule somebody turns off.

Nothing in this section is implemented. The tree carries no such record and no
such test today, and this document adds neither.

## What it costs, and the cost is not small

Three classes of change become slow or impossible.

The parts of the interface that are compiled in. Anything the Python surface does
not expose cannot be changed from an addon, and M5 is the milestone that wanted
to change them.

The defaults no preference reaches. The parameter surface covers what upstream
chose to make a parameter, and a default sitting outside it is not settable by
#62 however reasonable the request is.

Behaviour inside the kernel and the solver. A refusal that should be raised
earlier, a tolerance that should be different, a message that should name the
task rather than the operation: #64 can restate what reaches it, and it cannot
change what is raised.

What this buys is the thing the earlier effort did not have. The study in
`docs/decisions/prior-collaboration-attempt.md` records that the patched build is
the one piece of that work that is archived and no longer pushed to, while the
pieces that were not a fork are still maintained. That is evidence about this
exact decision rather than an analogy to it.

The tree is consistent with the rule as it stands, which is worth recording now
because the rule is cheapest to keep while it is already true:

    git ls-tree -r --name-only a8a732ac0e195fca2c75db1d26f60d0d62e39ee2 | wc -l
    43

    git ls-tree -r --name-only a8a732ac0e195fca2c75db1d26f60d0d62e39ee2 | grep -E '\.(c|cc|cpp|cxx|h|hpp|patch|diff)$' ; echo "exit=$?"
    exit=1

Forty three tracked files, none of them a source file or a patch of the kinds the
rule names.

## What would have to become true to revisit this

The decision is revisited against the list #61 produces, and not in the abstract.
That list is the wanted changes that could not be made from a surface, and it is
the only artefact on this board that will hold the real cost rather than the
predicted one.

The condition is a shape rather than a number: changes on that list that were
proposed upstream and neither accepted nor refused, where the guided path in #9
is worse for their absence in a way #67 can measure against the bar in #13. Two
of those three are already measured by issues that exist, so the revisit is an
argument somebody can check rather than a feeling that upstream is slow.

What does not revisit it. A single refused proposal, an upstream release that
takes longer than hoped, and the general observation that a fork would be faster.
A fork is faster once and slower every release after that, which is the whole
argument above.

## What this document does not settle

It does not decide entry 3 of #1. That entry is where the answer lives and where
a later revisit is argued.

It does not say what may be called on each surface. That is #33, and this
document deliberately stops at naming the mechanisms.

It does not add the check. #91 holds the pattern half, #98 holds the import half,
and the residual named above is held by neither.
