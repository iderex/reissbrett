# The collaboration attempt that has already been made

Somebody built the thing this board is planning, with funding and a team, and it
stopped. This document records what was built, what the collaboration model
actually was, what the client had to do to a document to take part, what can be
reused and on what terms, and what the public record does and does not say about
why it ended.

It is a study and not a design. It decides nothing. Issues #6, #7, #8 and #11
each make a decision this document is meant to inform, and each of them should
point here instead of restating any of it.

## How to read this

Every fact a command produced carries that command with its output, pasted as the
command printed it. Where something is inferred from what was read rather than
stated by a source, the sentence says so. Where something could not be
established on the route available, it is recorded as not established rather than
paraphrased from somewhere weaker.

Two sentences from the issue that asked for this are worth keeping at the top,
because they cut in opposite directions and both are easy to forget. A company
failing is not evidence that the technical approach was wrong. Code surviving is
not evidence that it was right.

## The commits this was written against

Everything below was observed at these commits on 2026-08-07. They move, so a
reader checking this later should re-run the command rather than trust the output
pasted here.

    for r in FreeCAD/Ondsel-Server FreeCAD/FC-Worker FreeCAD/OndselSolver FreeCAD/Ondsel-Lens-Addon Ondsel-Development/FreeCAD; do br=$(gh api "repos/$r" --jq '.default_branch'); echo "$r $br $(gh api "repos/$r/commits/$br" --jq '.sha') $(gh api "repos/$r/commits/$br" --jq '.commit.committer.date')"; done
    FreeCAD/Ondsel-Server main 54b241265d2eafb57a48f952f61abe4e45efc127 2026-06-14T12:30:01Z
    FreeCAD/FC-Worker main 188432d283c8997d22bb35360f434bf620df366a 2026-06-06T19:40:30Z
    FreeCAD/OndselSolver main 3afcb30353084d402e3c34760cd662cf22d88759 2026-07-23T18:28:20Z
    FreeCAD/Ondsel-Lens-Addon develop 4998dfad43ff320ec5dd55f1b2c47df7cf5b29b1 2025-12-22T16:43:11Z
    Ondsel-Development/FreeCAD main 7d2c10f4c103e565c377b2bacd7388e60d2f6e4d 2024-12-18T08:40:36Z

Two of the commands below need the server tree on disk rather than the contents
API, because they search across files. Both were run from a checkout made this
way:

    gh api "repos/FreeCAD/Ondsel-Server/zipball/54b241265d2eafb57a48f952f61abe4e45efc127" > server.zip
    unzip -q server.zip -d server
    cd server/FreeCAD-Ondsel-Server-54b2412

## What was built, and what state each piece is in

    for r in Ondsel-Development/FreeCAD FreeCAD/Ondsel-Server FreeCAD/FC-Worker FreeCAD/OndselSolver FreeCAD/Ondsel-Lens-Addon; do gh api "repos/$r" --jq '{full_name, archived, license: .license.spdx_id, language, pushed_at, created_at}'; done
    {"archived":true,"created_at":"2023-01-17T14:26:38Z","full_name":"Ondsel-Development/FreeCAD","language":"C++","license":"NOASSERTION","pushed_at":"2025-01-13T19:50:43Z"}
    {"archived":false,"created_at":"2025-01-19T15:51:27Z","full_name":"FreeCAD/Ondsel-Server","language":"JavaScript","license":null,"pushed_at":"2026-06-14T12:30:01Z"}
    {"archived":false,"created_at":"2025-01-19T15:50:49Z","full_name":"FreeCAD/FC-Worker","language":"Python","license":"LGPL-2.1","pushed_at":"2026-06-06T19:40:30Z"}
    {"archived":false,"created_at":"2025-05-04T18:17:51Z","full_name":"FreeCAD/OndselSolver","language":"C++","license":"LGPL-2.1","pushed_at":"2026-07-23T18:28:20Z"}
    {"archived":false,"created_at":"2025-06-22T17:18:19Z","full_name":"FreeCAD/Ondsel-Lens-Addon","language":"Python","license":null,"pushed_at":"2025-12-22T16:44:22Z"}

The patched build is the one piece that is dead. It is archived and its last push
predates the transfer. The other four sit in the CAD project's own organisation,
and three of them have been pushed to within the last few months.

### The service

`FreeCAD/Ondsel-Server`, JavaScript. Its own documentation states the stack:

    gh api "repos/FreeCAD/Ondsel-Server/contents/docs/technical.md?ref=54b241265d2eafb57a48f952f61abe4e45efc127" --jq '.content' | base64 -d | grep -n -A5 '^### Backend Server'
    12:### Backend Server
    13-- Node.js/Feathers.js REST API server
    14-- Handles core business logic, authentication, and data management
    15-- Integrates with MongoDB for data persistence
    16-- Manages user accounts, organizations, workspaces, and files
    17-- Integrates with FC-Worker service for model processing

and where the bytes go:

    gh api "repos/FreeCAD/Ondsel-Server/contents/docs/technical.md?ref=54b241265d2eafb57a48f952f61abe4e45efc127" --jq '.content' | base64 -d | grep -n -A3 '^### Storage Systems'
    32:### Storage Systems
    33-- MongoDB for structured data and metadata
    34-- Configurable file storage backend: local filesystem or S3
    35-

The headless runner is pulled in as a submodule rather than vendored:

    gh api "repos/FreeCAD/Ondsel-Server/contents/.gitmodules?ref=54b241265d2eafb57a48f952f61abe4e45efc127" --jq '.content' | base64 -d | tail -3
    [submodule "FC-Worker"]
        path = FC-Worker
        url = https://github.com/FreeCAD/FC-Worker.git

The repository is not only a backend. It carries a Vue single page frontend, a
docker compose stack, an analytics component and an admin panel, so reusing the
server is not a small act and the boundary would have to be drawn deliberately.

### The headless runner

`FreeCAD/FC-Worker`, Python, LGPL-2.1 by the API. A containerised headless runner
built on the CAD suite's own libraries, used by the service for rendering, format
conversion and export. Its module layout says what it covers:

    gh api "repos/FreeCAD/FC-Worker/contents/fc_worker?ref=188432d283c8997d22bb35360f434bf620df366a" --jq '[.[].name] | join(", ")'
    __init__.py, api_utils.py, assemblies_handler.py, code_runner.py, config.py, errors.py, exporter.py, freecad_libs, logger.config, model_configurer.py, sandbox, utils

This sits directly in front of what M2 plans, and it is the piece where reading
somebody else's working answer is most likely to be worth what the reading costs.

### The assembly solver

`FreeCAD/OndselSolver`, C++, LGPL-2.1. It has been absorbed upstream rather than
left standing alone. The suite's own third party directory carries it at the
version this project is looking at:

    gh api "repos/FreeCAD/FreeCAD/contents/src/3rdParty?ref=1.1.3" --jq '[.[].name] | join(", ")'
    .gitattributes, 3Dconnexion, CMakeLists.txt, GSL, OndselSolver, OpenGL, PyCXX, json, lazy_loader, libE57Format, libkdtree, lru-cache, salomesmesh, zipios++

So this project gets the solver by depending on the suite and has no reason to
carry it. That same listing is also the confirmation, from the suite's own tree,
of the container library #6 rests part of its argument on.

### The client

`FreeCAD/Ondsel-Lens-Addon`, Python. An ordinary addon: it ships `package.xml`,
`Init.py` and `InitGui.py`, installs through the suite's addon manager, and puts
its own tab in the interface.

    gh api "repos/FreeCAD/Ondsel-Lens-Addon/contents?ref=4998dfad43ff320ec5dd55f1b2c47df7cf5b29b1" --jq '[.[].name] | join(", ")' | tr ',' '\n' | grep -E 'package.xml|Init'
     Init.py
     InitGui.py
     package.xml

It is the only one of the four whose last push is more than a year old.

## The licence position, and a correction

The issue that asked for this recorded that two of the four carry no detectable
licence, and called that a blocker on reuse. The observation is right and the
conclusion is too strong. What is null is GitHub's detection, not the licence.
Both repositories declare their licensing in REUSE form, with a `REUSE.toml` at
the root and a `LICENSES/` directory holding the full texts:

    gh api "repos/FreeCAD/Ondsel-Server/contents/REUSE.toml?ref=54b241265d2eafb57a48f952f61abe4e45efc127" --jq '.content' | base64 -d | head -3
    # SPDX-FileCopyrightText: 2024 Ondsel <development@ondsel.com>
    #
    # SPDX-License-Identifier: AGPL-3.0-or-later
    gh api "repos/FreeCAD/Ondsel-Server/contents/LICENSES?ref=54b241265d2eafb57a48f952f61abe4e45efc127" --jq '[.[].name] | join(", ")'
    AGPL-3.0-or-later.txt, BSD-2-Clause.txt, CC-BY-4.0.txt, CC-BY-SA-4.0.txt, LGPL-2.1-only.txt

    gh api "repos/FreeCAD/Ondsel-Lens-Addon/contents/REUSE.toml?ref=4998dfad43ff320ec5dd55f1b2c47df7cf5b29b1" --jq '.content' | base64 -d | head -3
    # SPDX-FileCopyrightText: 2024 Ondsel <development@ondsel.com>
    #
    # SPDX-License-Identifier: LGPL-2.0-or-later
    gh api "repos/FreeCAD/Ondsel-Lens-Addon/contents/LICENSES?ref=4998dfad43ff320ec5dd55f1b2c47df7cf5b29b1" --jq '[.[].name] | join(", ")'
    Apache-2.0.txt, CC-BY-SA-2.0.txt, CC-BY-SA-4.0.txt, CC0-1.0.txt, LGPL-2.0-or-later.txt

Both `REUSE.toml` files also carry per-path annotations for third party material,
which is what makes the declaration worth something rather than a slogan at the
top of a file.

Three things follow, and the third is a limit rather than a finding.

The service is AGPL-3.0-or-later, which is the licence this repository already
carries, so in that direction there is no compatibility question at all.

The client addon is LGPL-2.0-or-later, the weaker side of the same family. Taking
that code into an AGPL work is the direction ordinarily available and the reverse
is the direction that is not. That sentence summarises the general shape and is
not a legal opinion; #100 is where the analysis is owed.

A REUSE declaration is a statement by the project that wrote it. It records what
that project believes about its own files. It is not a provenance audit and it
does not prove that every contributor had the right to contribute what they did.
Anything actually copied out of these trees needs its own look, and the fact that
the declaration is machine readable makes that look cheap rather than
unnecessary.

The patched build reports `NOASSERTION`, which is the detector saying it found a
licence file it could not resolve to one identifier. Nothing here plans to reuse
that tree, so the question is not pursued.

## The collaboration model that was implemented

This is the section #7 and #8 should read before deciding anything.

### The shape of the domain

    gh api "repos/FreeCAD/Ondsel-Server/contents/backend/src/services?ref=54b241265d2eafb57a48f952f61abe4e45efc127" --jq '[.[].name] | join(", ")'
    account-event, agreements, auth-management, code-runs, directories, download, email, file, groups, hooks, index.js, keywords, macros, models, notifications, org-invites, org-secondary-references, organizations, preferences, publisher, runner-logs, shared-models, site-config, upload, user-engagements, users, workspaces

An organization is a collection of workspaces and carries one of four types,
personal, open, private and admin, which set whether its contents are public by
default. A workspace is the working unit and holds directories, which hold files.
A file holds an array of versions. A model is the renderable object derived from
a file. A shared model is a share link with its own protection type and its own
permissions. All of that is described in the repository's own
`docs/services.md`, which is worth reading whole rather than quoted at length
here.

### What a version was

This is the part that matters most to #6, and it is smaller than one expects.

    gh api "repos/FreeCAD/Ondsel-Server/contents/backend/src/services/file/file.schema.js?ref=54b241265d2eafb57a48f952f61abe4e45efc127" --jq '.content' | base64 -d | sed -n '17,27p'
    export const fileVersionSchema = Type.Object({
      _id: ObjectIdSchema(),
      uniqueFileName: Type.String(),
      userId: Type.Optional(ObjectIdSchema()), // this is now optional because of some privacy issues
      message: Type.Optional(Type.String()),
      createdAt: Type.Optional(Type.Number()), // this is now optional because of some privacy issues
      thumbnailUrlCache: Type.Optional(Type.String()),
      fileUpdatedAt: Type.Optional(Type.Number()),
      lockedSharedModels: Type.Optional(Type.Array(sharedModelsSummarySchema)),
      additionalData: Type.Object({}),
    })

A version names one stored object by name, plus a message, a time and a thumbnail
cache. There is no content hash, no parent pointer, no part list and no diff. The
name is not derived from the content either, and the client says so where it
generates it:

    gh api "repos/FreeCAD/Ondsel-Lens-Addon/contents/APIClient.py?ref=4998dfad43ff320ec5dd55f1b2c47df7cf5b29b1" --jq '.content' | base64 -d | grep -n -A6 'def uploadFileToServer'
    593:    def uploadFileToServer(self, uniqueName, filename):
    594-        logger.debug(f"upload: {filename}")
    595-        # files to be uploaded need to have a unique name generated with uuid
    596-        # (use str(uuid.uuid4()) ) : test.fcstd ->
    597-        # c4481734-c18f-4b8c-8867-9694ae2a9f5a.fcstd
    598-        # Note that this is not a hash but a random identifier.
    599-        endpoint = "upload"

and the call site says a fresh identifier is minted on every upload, including
uploads of a file the server already holds:

    gh api "repos/FreeCAD/Ondsel-Lens-Addon/contents/Workspace.py?ref=4998dfad43ff320ec5dd55f1b2c47df7cf5b29b1" --jq '.content' | base64 -d | sed -n '671,675p'
        def upload(
            self, fileName, fileId=None, message="Update from the Ondsel Lens addon"
        ):
            # unique file name is always generated even if file is already on the
            # server under another uniqueFileName.  fileId is only used for updates

    gh api "repos/FreeCAD/Ondsel-Lens-Addon/contents/Workspace.py?ref=4998dfad43ff320ec5dd55f1b2c47df7cf5b29b1" --jq '.content' | base64 -d | grep -n 'uniqueName = f'
    691:        uniqueName = f"{str(uuid.uuid4())}.fcstd"  # TODO replace .fcstd by {extension}

So the answer that was implemented is the first of the four #6 lists: the whole
container, stored opaquely, one stored object per save. It restores exactly and
it is the simplest thing that can be correct. It deduplicates nothing, and the
history it produces says who saved what and when and with what message, and
nothing at all about what changed inside the document. That is the observation
#6 should argue against. It is not by itself an argument against it, because the
cost of the alternative is a dependency on the container's internal layout that
#38 has not yet characterised.

### There was no lock

The service has no exclusive-access mechanism. Every occurrence of the vocabulary
in the backend source is either a version-pinning mode or a restore-to-version
command, and none of it is a claim on a file by a person:

    grep -rniE '\block(s|ed|ing)?\b|\bcheckout\b|\bleases?\b|\bmutex\b|\bsemaphore\b' --include='*.js' backend/src
    backend/src/migrations/add-follow-support-to-shared-models.command.js:113:        changes.versionFollowing = versionFollowTypeMap.locked;
    backend/src/migrations/add-follow-support-to-shared-models.command.js:125:          // else assume versionFollowing is 'locked'; which is the default
    backend/src/migrations/add-follow-support-to-shared-models.command.js:290:      sm.versionFollowing = versionFollowTypeMap.locked;
    backend/src/migrations/update-directories-with-titles.command.js:25:      versionFollowing: `Locked`,
    backend/src/migrations/update-directories-with-titles.command.js:74:      // find all related sharedModels of versionFollowing 'Locked' and make sure correct and inserted into `currentVersion`
    backend/src/migrations/update-directories-with-titles.command.js:101:            console.log('        DELETED or NON-LOCKED sharedModel found; removing it.');
    backend/src/services/file/file.distrib.js:184:  case VersionFollowTypeMap.locked:
    backend/src/services/file/file.distrib.js:231:  case VersionFollowTypeMap.locked:
    backend/src/services/file/file.distrib.js:271:    case VersionFollowTypeMap.locked:
    backend/src/services/file/file.js:253:    throw new BadRequest('Pass versionId to checkout')
    backend/src/services/models/models.js:475:    versionFollowing: versionFollowTypeMap.locked,
    backend/src/services/models/models.js:487:    versionFollowing: versionFollowTypeMap.locked,
    backend/src/services/preferences/commands/checkoutToVersion.js:9:    throw new BadRequest('You need to mention `versionId` in order to checkout.')
    backend/src/services/shared-models/shared-models.curation.js:40:    if (context.result.versionFollowing === VersionFollowTypeMap.locked) {
    backend/src/services/shared-models/shared-models.schema.js:212:    return VersionFollowTypeMap.locked;  // default to locked
    backend/src/services/shared-models/shared-models.subdocs.schema.js:24:  locked: 'Locked',
    backend/src/services/shared-models/shared-models.subdocs.schema.js:30:    VersionFollowTypeMap.locked,

Read the matches rather than a count. `Locked` is one of the two values of
`versionFollowing`, and `checkout` means moving a file or a preference set back to
a named version.

The finding for #7, stated as a fact about what existed rather than as a
recommendation: the implemented model was last write wins over whole files, with
conflict recorded by the version list rather than prevented. Two people editing
one part produced two versions, and the one that survives as current is whichever
was uploaded second. Whether that is acceptable is #7's decision and this
document does not make it.

### Propagation had exactly two positions

    gh api "repos/FreeCAD/Ondsel-Server/contents/backend/src/services/shared-models/shared-models.subdocs.schema.js?ref=54b241265d2eafb57a48f952f61abe4e45efc127" --jq '.content' | base64 -d | sed -n '23,26p'
    export const VersionFollowTypeMap = {
      locked: 'Locked',
      active: 'Active',
    }

    gh api "repos/FreeCAD/Ondsel-Server/contents/backend/src/services/shared-models/shared-models.schema.js?ref=54b241265d2eafb57a48f952f61abe4e45efc127" --jq '.content' | base64 -d | grep -n 'default to locked'
    212:    return VersionFollowTypeMap.locked;  // default to locked

A share link either follows whatever version is current or is pinned to one named
version, and the default is pinned. Read against the options #8 sets out, that is
the first option and the second offered as a per-link switch, with the third
absent. Nothing computes whether accepting a newer version would break the thing
referencing it. The choice is made once by whoever made the link, and the person
downstream lives with it.

There is a second thing for #8 here. The pinning lives on the share link, and a
file version carries `lockedSharedModels`, the list of share links pinned to it,
so the graph that exists runs between a file version and the links pointing at
it. That is not a graph between documents, and what uses this part, where a part
is used inside another document rather than through a share link, is not answered
by these schemas. That is an inference from reading the schemas rather than
something any document there states, and it should be re-checked before #41 is
designed on top of it.

## What the client had to do to a document to take part

Nothing is written into the document to enrol it. The addon keeps its state in
the suite's user cache directory, outside the file:

    gh api "repos/FreeCAD/Ondsel-Lens-Addon/contents/DataModels.py?ref=4998dfad43ff320ec5dd55f1b2c47df7cf5b29b1" --jq '.content' | base64 -d | grep -n 'CACHE_PATH ='
    18:CACHE_PATH = FreeCAD.getUserCachePath() + "Ondsel-Lens/"

and the upload path opens the file, posts the bytes and records the result on the
server rather than in the document. Taking part therefore costs a document
nothing and leaves it loadable by anybody without the addon. That is a property
worth keeping and an easy one to lose.

The client does read into the container. To find links from one document to
another it opens the file as a zip, reads `Document.xml`, and looks for link
properties:

    gh api "repos/FreeCAD/Ondsel-Lens-Addon/contents/check_links.py?ref=4998dfad43ff320ec5dd55f1b2c47df7cf5b29b1" --jq '.content' | base64 -d | sed -n '13,37p'
    def find_paths_links_xml(xml_content):
        """Find the paths of (missing) links in XML content."""

        root = ET.fromstring(xml_content)
        paths_links = []
        for prop in root.findall(".//Property[@name='LinkedObject']/XLink"):
            file_path = prop.get("file")
            if file_path:
                paths_links.append(file_path)
        return paths_links


    def find_paths_links_file(file_path):
        """Find the paths of (missing) links in a FreeCAD file."""

        if not zipfile.is_zipfile(file_path):
            raise FreeCADFileException(f"File {file_path} is not a valid FreeCAD file.")

        with zipfile.ZipFile(file_path, "r") as z:
            if "Document.xml" not in z.namelist():
                raise FreeCADFileException("Document.xml is not present")

            with z.open("Document.xml") as f:
                xml_content = f.read().decode("utf-8")
                return find_paths_links_xml(xml_content)

and it used that reading to refuse an upload rather than to handle the case:

    gh api "repos/FreeCAD/Ondsel-Lens-Addon/contents/Workspace.py?ref=4998dfad43ff320ec5dd55f1b2c47df7cf5b29b1" --jq '.content' | base64 -d | sed -n '679,687p'
        file_path = Utils.joinPath(self.getFullPath(), fileName)
        if (
            check_links.find_paths_links_file(file_path)
            and self.apiClient.is_user_solo()
        ):
            raise APIClient.APIClientTierException(
                "This document contains links to parts in another document. Only "
                "single-document files are supported in your current lens account. "
            )

The one place the addon writes into a document is a separate feature. Its
reloadable-file integration adds properties to an object so an imported file can
be refreshed in place:

    gh api "repos/FreeCAD/Ondsel-Lens-Addon/contents/integrations/reloadablefile/reloadable.py?ref=4998dfad43ff320ec5dd55f1b2c47df7cf5b29b1" --jq '.content' | base64 -d | grep -n 'obj.addProperty('
    43:        obj.addProperty(
    50:        obj.addProperty(
    54:        obj.addProperty(
    58:        obj.addProperty(

That is an ordinary scripted-object pattern and it belongs to that feature rather
than to participation.

On the question this section was asked, whether any of it needed something
outside the surface #33 will describe. Everything above uses addon interfaces the
suite documents, with one exception that is not an interface at all: reading the
container as a zip and parsing `Document.xml` is a dependency on somebody else's
file format, not a call into their software. It was stable enough that the addon
shipped on it, and it is exactly what #38 owes a characterisation of, because a
format nobody promised can change without anybody breaking a promise.

## Why it ended, and what is not established here

What the tracker shows is a transfer rather than a deletion. The company's own
fork of the suite is archived with a last push of 2025-01-13, and the four
surviving repositories were created in the CAD project's organisation from
2025-01-19 onward. Both dates are in the metadata output above.

The FreeCAD project's own blog records the money and the intent, in a post dated
2025-02-14 by Jo Hinchliffe, "The Ondsel Onwards Fund", at
https://blog.freecad.org/2025/02/14/the-ondsel-onwards-fund/, read on 2026-08-07.
It states that "a one-time donation of EUR40,000 has been given to the FPA with a
request that it be spent on modifying some of the Ondsel codebase", and that "the
fund would be allocated towards work that refactors the code for the software
developed by Ondsel so that it is more useful for the wider community". That post
does not say why the company stopped.

The company's own statement is at https://www.ondsel.com/blog/goodbye/. It was
not read for this document: the fetch returned HTTP 403 on the route available
here on 2026-08-07. So nothing here paraphrases it and nothing here summarises a
reason for the shutdown. Somebody who can reach that page should add the
quotation and its date to this document. Until then the reason this effort ended
is recorded as not established, rather than as commercial, technical or anything
else.

One thing can be said without that page, and it is worth saying because it is
what the code shows. None of the four surviving repositories was dropped for
being broken. Three are still being pushed to, and the solver was taken into the
suite itself, which is not what happens to work nobody wanted.

## What this project can reuse, adapt or only read

- `FreeCAD/Ondsel-Server`. Reusable in principle, and the licence is the one this
  repository already carries. Adapting it means taking on a Node and MongoDB
  stack with a Vue frontend attached, which is a means decision and not a free
  one, and it belongs to whichever issue proposes it. Read it in any case: its
  schemas are the cheapest available description of what a working answer had to
  hold.
- `FreeCAD/FC-Worker`. LGPL-2.1 and the closest existing thing to what M2 plans.
  This is where reuse should be examined seriously rather than assumed away,
  because a containerised headless runner is a large amount of work to repeat.
- `FreeCAD/OndselSolver`. Nothing to do. It is inside the suite this project
  depends on.
- `FreeCAD/Ondsel-Lens-Addon`. LGPL-2.0-or-later. The most useful part of it is
  not code to copy but the shape of what a client had to do, which is recorded
  above. Its container reading is the piece most worth reusing directly, and it
  is under forty lines.
- `Ondsel-Development/FreeCAD`. Read only, and mostly not worth reading. It is
  the patched build, it is archived, and #4 argues against this project ever
  carrying one.

## What this document does not settle

It makes no decision. #6 decides what a version is, #7 decides the collaboration
model, #8 decides propagation and #11 decides the delivery form. Each should
reference this document rather than restate part of it, and each is free to
depart from what was done here as long as the departure is argued rather than
assumed.

Two gaps are open in the document itself. The reason the effort ended is not
established, for the reason given above. And the inference about the reference
graph was read out of the schemas rather than out of any statement, so #41 should
confirm it rather than build on it.
