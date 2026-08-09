# What this layer may call into the suite

This is the note issue #33 asks for. `docs/decisions/0004-where-the-layer-attaches.md`
decided that the layer attaches to the suite's documented extension surfaces and
to nothing else, and it stopped at naming the mechanisms. It says so in its own
words, so this note is the half it left:

    git show origin/main:docs/decisions/0004-where-the-layer-attaches.md | grep -n -A2 'It does not say what may be called'
    264:It does not say what may be called on each surface. That is #33, and this
    265-document deliberately stops at naming the mechanisms.
    266-

The purpose is narrow. A reviewer should be able to look at a call and decide
whether it is legal without opening the suite's source. What follows is
therefore a permission list rather than a survey: a name that is not here is not
permitted, and the way to add one is to add it here with the evidence that puts
it in a tier, in the same change that first calls it.

## Nothing here describes code that exists

This tree holds no unit that calls anything:

    git ls-tree -r --name-only origin/main | grep -cE '\.(go|py|cs|rs|ts|js|cpp|c|java)$'
    0

So the first done-when condition of #33, which asks for every interface this
project calls, is answered by a list of what it may call and by that count.
Nothing below is a report about a running program, and no entry here should be
read as saying that a call has been made or measured.

## The version everything is read at

FreeCAD 1.1.3, which is the pin in `docs/decisions/0005-the-pinned-suite-version.md`.
Every `gh api` command below carries `ref=1.1.3` for that reason. Where a route
reads a default branch instead of the tag, the sentence says so.

## What upstream promises about its Python interface, and what it does not

There is one written commitment and it is worth quoting whole rather than
summarising, because the strength of every assessment below is derived from it:

    gh api repos/FreeCAD/FreeCAD/contents/CONTRIBUTING.md?ref=1.1.3 --jq '.content' | tr -d '\n' | base64 -d | sed -n '58p'
    10. Changes that break python API used by extensions SHALL be avoided. If it is not possible to avoid breaking changes, the amount of them MUST be minimized and PR MUST clearly describe all breaking changes with clear description on how to replace no longer working solution with newer one. Contributor SHOULD search for addons that will be broken and list them in the PR.

Read for what it is. It binds contributors to the suite to a standard of care,
it requires a break to be described where it is made, and it asks a contributor
to look for the addons a change breaks. It is not a compatibility guarantee, it
names no interface as stable and no interface as internal, it sets no
deprecation window, and it promises no release at which a removal will not
happen. A layer built on it is built on care rather than on a contract, and the
consequence is #34's: the only thing that tells this project a break has
happened is a harness that runs against a new release.

That is the ceiling. Nothing below is more stable than that sentence, whatever
tier it sits in.

## The three tiers used below

**Declared.** The interface has a machine readable declaration in the pinned
tree, a `.pyi` stub or a published schema, so what it is called and what it
takes can be read without reading an implementation. A change to it is visible
in a diff of a declaration file.

**Named in the tree.** The interface is a literal in the suite's own source, a
method table entry or a filename the loader looks for, and there is no
declaration file. It can be read, and a change to it looks like an ordinary code
change rather than an interface change.

**Prose only.** The suite's own documentation explains it and the tree carries no
declaration. This is where a string that is really an interface lives.

A tier is a statement about how a change would be seen, not about how likely one
is. None of the three carries a promise, because the sentence above is the only
promise there is.

## Surface 1. The addon package and its manifest

What may be produced: a `package.xml` manifest conforming to the published
schema, in the base directory of the branch that is published.

    gh api repos/FreeCAD/Addon-Manifest-Schema/contents/Docs --jq '[.[].name] | join(" ")'
    Development.md README.md Repository.md Versioning.md Workarounds.md

The upstream documentation reference is the wiki article `Package Metadata`,
read at the address below on 2026-08-09:

    curl -s 'https://wiki.freecad.org/index.php?title=Package_Metadata&action=raw' | sed -n '19p' | cut -c1-186
    The metadata file must be a valid, well-formed XML 1.0 document. It must be called "package.xml", and must exist in the base directory of each of the software package's displayed branche

`0004` records that this address answered with an access denied page rather than
the article, and that somebody who could reach it should add the article and the
date it was read. That is what the two commands here do. The route recorded
there and the route used here are different routes and this note does not
explain the difference; what it asserts is only what the command above returned.

What may be read on this surface from inside the suite: `App.Metadata`, which is
the suite's own reader for the same file.

    gh api repos/FreeCAD/FreeCAD/contents/src/App/Metadata.pyi?ref=1.1.3 --jq '.content' | tr -d '\n' | base64 -d | grep -n '^class'
    20:class Metadata(PyObjectBase):

Tier: declared, in both directions. The schema is versioned in two parts, and the
first of the two is what a consumer of the manifest has to watch, because the
second moves when the schema is rebuilt without the format changing:

    gh api repos/FreeCAD/Addon-Manifest-Schema/contents/Docs/Versioning.md --jq '.content' | tr -d '\n' | base64 -d | sed -n '12p'
    `<Manifest Format>.<Schema Build>`

## Surface 2. The workbench entry points

What may be relied on: a directory carrying `Init.py`, executed when the suite
starts, and `InitGui.py`, executed after it and only when the suite starts with
an interface. Those two names are the whole of the contract.

The names are in the suite's own modules at the pinned tag:

    gh api repos/FreeCAD/FreeCAD/contents/src/Mod/Draft?ref=1.1.3 --jq '[.[] | select(.name | startswith("Init")) | .name] | join(" ")'
    Init.py InitGui.py

and the upstream documentation says what each is for:

    curl -s 'https://wiki.freecad.org/index.php?title=Workbench_creation&action=raw' | sed -n '25p' | cut -c1-198
    You need a folder, with any name you like, placed in the user Mod directory, with an {{incode|Init.py}} file, and, optionally an {{incode|InitGui.py}} file. The {{incode|Init.py}} file is executed w

Tier: named in the tree, with prose behind it. The two filenames are literals
the loader looks for, and a rename upstream would be a break rule 10 covers.

What this surface costs is stated in the article rather than inferred: work done
in `InitGui.py` at load time is work done before the person can do anything, so
the guided path in #61 attaches its commands here and does its work when the
workbench is activated, not when it is imported.

## Surface 3. The document interface

This is the surface most of M3 and M5 sit on, and it is the one with the best
declaration. The module level functions are a table in the suite's source, and
the whole set at the pinned tag is 42:

    gh api repos/FreeCAD/FreeCAD/contents/src/App/ApplicationPy.cpp?ref=1.1.3 --jq '.content' | tr -d '\n' | base64 -d | grep -oE '^\s*\{"[A-Za-z_]+"' | grep -oE '[A-Za-z_]+' | sort -u | wc -l
    42

Of those, this project may call the following and nothing else on the module.

Opening, creating and closing a document, and knowing which one is active:
`newDocument`, `openDocument`, `closeDocument`, `getDocument`, `listDocuments`,
`activeDocument`, `setActiveDocument`.

Being told that a document changed rather than polling for it:
`addDocumentObserver`, `removeDocumentObserver`. This is what #41 and #53 attach
to.

The parameter surface, which is surface 5 below: `ParamGet`.

Where this project's own state goes, which is outside any document:
`getUserAppDataDir`, `getUserCachePath`, `getUserConfigDir`.

Reading which suite is running, for the refusal #35 owes: `Version` and
`ConfigGet`. Which of the two carries the number #35 compares is #30's
measurement from inside a running suite and not a claim this note makes.

The reference graph inside a document, which #55 needs before it can say what a
change breaks: `getLinksTo`, `getDependentObjects`.

Everything else in the 42 is outside the permission list. The registration
functions for import and export formats are the ones most likely to be wanted
later, by #71 for a drawing and #77 for a produced program, and the rule is that
whichever issue needs one adds it here in the same change rather than calling it
and leaving this list behind.

On a document object, the declaration is a stub file:

    gh api repos/FreeCAD/FreeCAD/contents/src/App/Document.pyi?ref=1.1.3 --jq '.content' | tr -d '\n' | base64 -d | grep -cE '^    def '
    40

The permitted set on a document: `save`, `saveAs`, `saveCopy`, `isSaved`,
`getFileName`, `getProgramVersion`, `recompute`, `mustExecute`, `isTouched`,
`addObject`, `removeObject`, `getObject`, `getObjectsByLabel`, `findObjects`,
`getLinksTo`, `getDependentDocuments`, `openTransaction`, `commitTransaction`,
`abortTransaction`, `getUniqueObjectName`.

`purgeTouched` is declared and is refused here. It is in the list under
"what this project must never do" below, with what breaks if it is called.

Tier: declared. There are 21 stub files beside `Document.pyi` in the same
directory, which is what makes this surface readable without reading an
implementation:

    gh api repos/FreeCAD/FreeCAD/contents/src/App?ref=1.1.3 --jq '[.[].name | select(endswith(".pyi"))] | length'
    21

## Surface 4. Object identity on a document object

`Name` and `Label` are the two names an object carries and they are not the same
kind of thing. The stub says which is which, and it says it in the type rather
than in prose:

    gh api repos/FreeCAD/FreeCAD/contents/src/App/DocumentObject.pyi?ref=1.1.3 --jq '.content' | tr -d '\n' | base64 -d | sed -n '31,32p'
        Name: Final[Optional[str]] = ""
        """Return the internal name of this object"""

`Final` is the declaration that it is not assignable. `Label` is the one a person
edits and it is not unique. #39 decides what a version pins to and this note
gives it the fact rather than the decision: the internal name is the only one of
the two the suite itself declares as fixed, and it is fixed only for as long as
the object lives in that document.

Tier: declared.

## Surface 5. The parameter surface

What may be called: `App.ParamGet(<path>)`, where the path is a string naming a
group, and the group object it returns. This is how a packaged set of settings
reaches the suite without editing a file the operator owns, and it is what #62
applies the known starting state through.

The function is in the same method table as surface 3:

    gh api repos/FreeCAD/FreeCAD/contents/src/App/ApplicationPy.cpp?ref=1.1.3 --jq '.content' | tr -d '\n' | base64 -d | sed -n '50p'
        {"ParamGet", (PyCFunction)Application::sGetParam, METH_VARARGS, "Get parameters by path"},

and the suite's own modules call it with a literal path at the pinned tag:

    gh api repos/FreeCAD/FreeCAD/contents/src/Mod/CAM/Path/Preferences.py?ref=1.1.3 --jq '.content' | tr -d '\n' | base64 -d | sed -n '255p'
        grp = FreeCAD.ParamGet("User parameter:BaseApp/Preferences/Macro")

Tier for the function: named in the tree. Tier for the paths: prose only, and
this is the entry on this list most likely to be misread as safer than it is.
A path is a string. Renaming a group upstream breaks every consumer of it and
does not look like a Python interface change to the person making it, so rule 10
above is unlikely to be applied to it. The upstream documentation for the paths
is an article rather than a schema:

    curl -s 'https://wiki.freecad.org/index.php?title=Fine-tuning&action=raw' | sed -n '17p' | cut -c1-140
    This page lists parameters that are not accessible via the preferences editor, but that you can set manually to fine-tune your FreeCAD installation

The consequence and who absorbs it are in the section below.

## Surface 6. The console entry point and the modules the committed path needs

The suite builds a second executable with no interface, and `0004` names the
listing that shows it. Nothing here has run it, and the name a build installs is
#30's measurement rather than this note's claim.

The modules the committed path in `docs/decisions/0009-the-committed-beginner-workflow.md`
reaches are four of the directories in the suite's module tree:

    gh api repos/FreeCAD/FreeCAD/contents/src/Mod?ref=1.1.3 --jq '[.[] | select(.type=="dir") | .name] | length'
    33

Sketcher, PartDesign, TechDraw and CAM. What may be called inside each of them is
not settled here, and the reason is in the last section.

One fact about CAM belongs here because #74 pins one of them: the post processors
are files in a directory at the pinned tag, and there are 40 of them.

    gh api repos/FreeCAD/FreeCAD/contents/src/Mod/CAM/Path/Post/scripts?ref=1.1.3 --jq 'length'
    40

Tier: named in the tree, for the module names. The directory listing is a coarse
instrument, which is the same thing `0009` says about its own use of it.

## What is used with no stability statement, and who absorbs a change

Three things on this list carry no upstream promise at all. They are listed
separately because #33 asks for that, and because the consequence of each is
different.

**The legacy shim in the machining module.** Upstream declares it legacy and
says it may be removed, in the directory's own README:

    gh api repos/FreeCAD/FreeCAD/contents/src/Mod/CAM/PathScripts/README.md?ref=1.1.3 --jq '.content' | tr -d '\n' | base64 -d | sed -n '1,5p'
    This is a legacy directory. The remaining files in here should either get refactored so
    they can move into the new structure or removed.

    The PropertyBag diversion files are here in order to keep existing ToolBit files working.
    They might get removed in future versions.

Consequence: a call into it stops working at a release nobody has to announce,
because removing something already announced as removable is not a break under
rule 10. Who absorbs it: this project, at the release it happens on. It is
therefore not on the permission list. Where a thing is only reachable through it,
the change that needs it says so here and takes the cost knowingly.

**The parameter group paths.** Consequence: settings silently stop being
applied, which is worse than failing, because the workspace opens in a state
that looks deliberate. Who absorbs it: this project, and the person who notices
is the operator whose starting state moved. #62's test is what turns that into a
failure rather than a surprise, and #34's harness is what sees it at a new
release rather than in a bug report.

**The document container and the XML inside it.** This one is not an interface at
all and it is the most important entry here. It is a file format, and the
earlier attempt studied in #2 depended on exactly it:

    git show origin/main:docs/decisions/prior-collaboration-attempt.md | grep -n 'not a call into their software'
    384:file format, not a call into their software. It was stable enough that the addon

Consequence: the layout can change without anybody breaking a promise, because
no promise covers it. Who absorbs it: this project, and #38 is what owes the
characterisation that says how much of the layout is being relied on. Nothing in
that direction is on the permission list above, because the permission list is
about calls and this is not one.

## What this project must never do

Each entry says what breaks. These are the reverse direction #33 asks for, and
they are written so that #98 can turn the ones about imports into tests.

**Never write into a document in order to enrol it in collaboration.** What
breaks: a document that only opens correctly for somebody who has this project
installed. The earlier attempt kept its state outside the file and its documents
stayed loadable by anybody, which the study calls a property worth keeping and an
easy one to lose.

**Never assign to the internal name of a document object.** What breaks: every
reference that pins an identity, which is #39's whole subject. The stub above
declares it `Final`, so the attempt would fail rather than corrupt anything, and
the failure would be at the point a version is being recorded.

**Never modify a document file the running suite has open, and never write one
behind its back.** What breaks: the suite's in-memory document and the file on
disk disagree, and the next save the person makes discards whatever was written,
silently and with no error anywhere.

**Never call `purgeTouched` to make a document look recomputed.** What breaks: a
recorded version whose geometry does not follow from its parameters. #43 proves
that a restore returns the same bytes, which is a different claim, so nothing
downstream would catch it.

**Never reach into a path the suite declares private, or into a header.** The
suite has such a path at the pinned tag:

    gh api repos/FreeCAD/FreeCAD/contents/src/App/private?ref=1.1.3 --jq '[.[].name] | join(" ")'
    DocumentP.h

What breaks: rule 10 covers the Python interface used by extensions, and nothing
covers this, so the break arrives without a description and without a
replacement.

**Never load the suite in the process that listens on a socket.** This is
`docs/decisions/0037-the-process-boundary.md`'s rule and it is restated here only
as a call rule, because this note is where somebody looks up whether a call is
legal. What breaks is argued there and is not restated.

**Never carry a copy of or a patch against the suite.** That is `0004`'s rule and
this note adds nothing to it.

## What the earlier attempt needed, and whether this project needs the same

`docs/decisions/prior-collaboration-attempt.md` answers the first half directly,
in the section on what the client had to do to a document to take part. The
answer it records is that everything the client did used addon interfaces the
suite documents, with one exception, which is the container reading in the
section above.

Whether this project needs the same. It needs the exception and it does not need
one part of what the earlier client did.

It needs the container reading, and for more than the earlier client did. That
client opened the container to find links to other documents and used the answer
to refuse an upload. #39 has to pin an identity inside a document and #42 has to
produce a change summary a person can read, and neither is answerable from the
outside of the file. So the dependency is larger here, not smaller, and #38 is
where its size is established rather than assumed.

It does not need to write into a document to take part, and the entry above makes
that a rule rather than an intention.

One thing the earlier client did that is not needed and should not be copied by
habit: it added properties to an object for its reloadable file feature. The
study places that with the feature rather than with participation, and this
project has no such feature in the committed path.

## What refuses any of this today

Nothing. There is no unit to check, and neither of the two mechanisms that would
carry these rules holds one yet.

    git show origin/main:.github/invariants.txt | grep -cE '^id: '
    2

The two rules that exist are an absolute home path and a display server in a
workflow file, and neither is about this note. The import half belongs to #98,
which has landed nothing, and the header pattern half belongs to #91, whose own
rules file names the copy-or-patch rule as arriving with #4.

PROSE, NOT ENFORCEMENT for every rule in this document, issues #91 and #98. The
permission list is the kind of rule a machine could refuse once there are units,
and the entries about what must never be done to a document are not: no reading
of this tree decides whether a call left a document in a state it should not be
in. The first half is owed a mechanism. The second half is a rule that ends
where it is written, and marking it does not change that.

## What this note does not settle

It does not go below the module. What may be called inside Sketcher, PartDesign,
TechDraw and CAM is a list nobody can write honestly before there is a unit
calling one of them, and a list written now would be a guess that later work has
to be argued against. The rule that replaces it is the one at the top: the change
that first calls something adds its line here, with the command that puts it in a
tier.

It does not decide what #38 finds inside the container. It records that the
dependency exists and that it is larger here than it was for the earlier attempt.

It does not measure anything about a running suite. Every command above reads a
published tree or a published article. The first measurement from inside a
running suite is #30's.

No second person read this. The commands above stand in place of one, and what
they cannot back is whether this is the right permission list.
