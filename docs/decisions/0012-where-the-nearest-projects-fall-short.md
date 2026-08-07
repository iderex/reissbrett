# Where the nearest projects fall short

This project's reason to exist is a claim about other projects. This document
measures that claim once so that every later argument points here instead of
repeating it.

## How to read this

Every claim about a licence, a price or availability carries its source and
the date it was checked. Every claim about behaviour names the version it was
observed on and how it was observed. Where a comparison is a matter of taste
rather than fact, the sentence says it is a judgement. Where something could
not be established on the route available, it is recorded as not established
rather than filled in from somewhere weaker.

Two failure modes are worth naming at the top because both are easy and both
are fatal here. Overstating the gap makes this project look dishonest to the
people who know this field, and those are the people it needs. Understating it
makes the project pointless. Two of the findings below cut against this
project's own thesis, and they are in the document for that reason.

## How the observations were made, and the limit that puts on them

Nothing below was observed by using either program. No modelling suite and no
browser session was driven for this document. What was observed is source
trees at named refs through the GitHub API, repository metadata through the
same API, and vendor pages read on the date given.

That is a real limit and it decides what this document is allowed to say. A
claim about what a program does when somebody uses it is not made here. A
claim about what a program ships, at a named version, is. Where the
distinction matters the sentence says which kind it is. #13 is where the
workflow bar gets measured by somebody using the thing, and that measurement
is not this document.

The version observed throughout is the suite release this project's pin issue
#5 is about, `1.1.3`, and the observation date throughout is 2026-08-07.

## The axes

The six axes are the ones this project claims something on, and no others.
Adding a seventh axis on which this project happens to be strong would be a
way of winning the comparison rather than making it.

- Continuity of the workflow from sketch to machine program
- Consistency of the interface
- Whether collaboration exists and what it can do
- Whether a change propagates to what references it
- Where the data lives and who can reach it
- Whether it runs without an account

## The suite this project builds on

### What it ships at 1.1.3

    gh api "repos/FreeCAD/FreeCAD/contents/src/Mod?ref=1.1.3" --jq '[.[] | select(.type=="dir") | .name] | join(", ")'
    Assembly, BIM, CAM, Cloud, Draft, Fem, Help, Idf, Import, Inspection, JtReader, Material, Measure, Mesh, MeshPart, OpenSCAD, Part, PartDesign, Plot, Points, ReverseEngineering, Robot, Sandbox, Show, Sketcher, Spreadsheet, Start, Surface, TechDraw, TemplatePyMod, Test, Tux, Web

    gh api "repos/FreeCAD/FreeCAD/contents/src/Mod/CAM/Path/Post/scripts?ref=1.1.3" --jq 'length'
    40

No count of workbenches is quoted from the first listing, because several of
its entries are not workbenches a person meets, and a count that treats `Test`
and `TemplatePyMod` as things a beginner opens would be a number invented to
make a point.

### Continuity

Every stage this project's readme names is present in one program at 1.1.3.
`Sketcher`, `PartDesign`, `TechDraw` and `CAM` are in the listing above, and
the CAM module ships forty post processor scripts. So the capability is
continuous and the pieces are all there. What this project claims is missing
is not a stage, and any sentence that says the suite cannot go from sketch to
machine program is wrong.

### Interface consistency

Judgement, and this document does not measure it. The listing shows that the
stages are separate modules, each supplying its own workbench, which is an
architectural fact rather than a statement about how the interface feels.
Whether moving between them is coherent for a beginner is what
#13 has to measure with a person, and until it does, this project's claim on
this axis rests on nothing this document can back.

### Collaboration

The suite ships a `Cloud` module, which is easy to miss and would make an
unqualified claim of "no collaboration" false. What it is, read at the
observed version:

    gh api "repos/FreeCAD/FreeCAD/contents/src/Mod/Cloud/App/AppCloud.cpp?ref=1.1.3" --jq '.content' | base64 -d | grep -c 'AmzS3v4\|AmzS3v2'
    24

It is a reader and writer for S3 style object storage, signing requests with
the Amazon v2 and v4 schemes, so a document can be saved to and loaded from a
bucket the user supplies. That is remote storage. It is not a version store,
there is no lock in it, and nothing in it notifies anybody. So the honest
statement on this axis is narrower than "there is none": the suite ships a way
to keep a file somewhere else, and ships nothing that arbitrates two people
editing it.

### Propagation

Nothing observed in the tree at 1.1.3 computes what a changed document does to
a document that references it. This is an absence in a listing rather than a
positive observation, which is weaker evidence, and it is written as the
weaker thing it is.

### Where the data lives

On the machine that runs the program, unless the user points the `Cloud`
module at a bucket, in which case it goes where they pointed it. This is the
axis on which the suite and this project agree, and this project claims no
advantage over it.

### Account

No account or authentication module appears in the listing above. Same limit
as the propagation paragraph: an absence in a directory listing is weaker than
an observation of the program starting without one.

### What this project claims against the suite, once the above is subtracted

Two things, and they are narrower than the readme currently reads. A guided
path through stages the suite already has, and a collaboration and versioning
layer the suite does not have at all. Neither is a claim that the suite lacks
a capability, and the first is a claim about presentation that #13 owes a
measurement of before anybody should believe it.

## The cloud incumbent

Onshape. Read at https://www.onshape.com/en/pricing on 2026-08-07, quoted as
the page has it.

### Price and terms

The Free plan is "$0", carrying "Unlimited public storage", with the
qualification "For non-commercial use only. All Onshape Documents are
accessible to the public." Standard is "$1,500 per user per year" and adds
"Unlimited private storage". Professional is "$2,500 per user per year" and
adds "Company-managed data". Enterprise is "Contact us for custom pricing", so
no number is quoted for it. The page's own FAQ states "Only paid plans support
private data and commercial use."

The page sets the words "public" and "private" in bold inside the two storage
lines. The emphasis is dropped in the quotations above rather than reproduced,
and the words themselves are quoted as the page has them.

### Continuity

Onshape ships integrated manufacturing. From
https://cad.onshape.com/help/Content/CAMStudio/cam_studios.htm read on
2026-08-07: "CAM Studio is included with an Onshape Professional (or above)
subscription and includes basic toolpath strategies and options for 2.5 and 3
axis machining", and the strategies "are posted into a language that a CNC
machine interprets called Gcode."

So on the axis this project's readme leads with, the incumbent is not weak. A
paying customer gets sketch to machine program in one place. Any argument that
this project is the only continuous path is false, and this is the first of
the two findings that cut against this project's thesis.

### Collaboration and propagation

Not established here. These are behavioural properties of a hosted product,
and observing them means driving the product, which is outside what this
document did. Vendor marketing describes both, and vendor marketing is not
evidence of behaviour. This document does not claim the incumbent is weak on
either axis, and no later argument may cite it for that.

### Where the data lives

On the vendor's servers, and on the free plan every document is public to
everybody, which the quoted sentence says outright. That is the real gap and
it needs no exaggeration: an operator who wants a model that is neither on
somebody else's hardware nor public has to pay 1,500 US dollars per user per
year, and after paying, the data is still on somebody else's hardware.

### Account

Required. The plans are priced per user, which the quoted prices show.

### What happens if you stop paying

Not established. The terms page at https://www.onshape.com/en/terms-of-use
returned HTTP 404 on the route available here on 2026-08-07, and no other
route was read. So nothing is claimed about retention, export or deletion
after a subscription ends. The one thing the pricing page does support is that
the free plan carries no private storage, so whatever a lapsed account keeps,
it does not keep privately. Anybody who can read the current terms should add
the clause and its date here.

## The earlier collaboration attempt

Recorded in full in `docs/decisions/prior-collaboration-attempt.md` and not
restated. What it establishes for these axes, briefly, with the detail and the
commands there.

It was a hosted service with a client addon. A version was one whole container
stored opaquely under a random identifier, so versioning existed and
deduplication did not. There was no lock, and the model was last write wins
over whole files. Propagation had two positions, following the current version
or pinned to one, with pinned the default, and nothing computed whether an
update would break the thing referencing it. The service is AGPL-3.0-or-later
and the client addon LGPL-2.0-or-later, both declared in REUSE form, and three
of the four surviving repositories are still being pushed to.

The axis it fell short on, and it is the one this project is built around, is
propagation with a check. The axis it did not fall short on is that it existed
and worked, which is more than anything else on this list can say.

## Other efforts, found by looking

Searched on 2026-08-07 through a general web search for self-hosted and open
source CAD collaboration, version control and PDM, and each candidate the
search returned was then checked against the GitHub API rather than against
the page that named it.

### CADCloud

`opencomputeproject/CADCloud` is the closest thing to this project that
exists, and the most important entry here. It is a FreeCAD-integrated sharing
and version tracking platform, and the suite's own `Cloud` workbench above is
its client half.

    gh api repos/opencomputeproject/CADCloud --jq '{archived, license: .license.spdx_id, pushed_at, stargazers_count}'
    {"archived":false,"license":"MIT","pushed_at":"2022-01-12T15:43:05Z","stargazers_count":108}

MIT licensed and self-hostable, which is exactly this project's position. Its
last push is 2022-01-12. The Open Compute Project page at
https://www.opencompute.org/blog/building-a-cad-collaborative-open-source-platform,
read on 2026-08-07, describes it in the present tense as an active project
with core developers currently working on document links and project forks.
The repository metadata and that page disagree, and this document takes the
metadata, because a date a machine produced is evidence and a page nobody
updated is not. What is not established is whether development moved somewhere
this search did not find.

### DocDokuPLM

A general PLM rather than a CAD collaboration server, listed because searches
for this problem return it and somebody will ask.

    gh api repos/docdoku/docdoku-plm --jq '{archived, license: .license.spdx_id, pushed_at}'
    {"archived":false,"license":"AGPL-3.0","pushed_at":"2021-05-10T08:58:15Z"}
    gh api repos/docdoku/docdoku-plm-server --jq '{archived, license: .license.spdx_id, pushed_at}'
    {"archived":false,"license":"AGPL-3.0","pushed_at":"2023-07-24T15:58:52Z"}

### OpenPLM

The same category and in a worse state. The name resolves to several unrelated
forks rather than to one project, and the most recently pushed one carries no
licence the API can detect:

    gh api repos/amarh/openPLM --jq '{archived, license: .license.spdx_id, pushed_at}'
    {"archived":false,"license":null,"pushed_at":"2024-01-30T10:02:43Z"}

### Proprietary self-hosted PDM

It exists and is mature, and leaving it out would be dishonest. It is not
compared on these axes because it is tied to CAD systems this project is not a
layer over, so a comparison would be about the CAD system rather than about
the collaboration layer. That it exists is enough to refuse any sentence
claiming self-hosted CAD collaboration has never been built.

### The negative result

No actively maintained open source self-hosted CAD collaboration server was
found by this search other than the pieces of the earlier attempt studied
above. That is a statement about what one search on one day returned, and it
is exactly the kind of claim the issue that asked for this warned against
assuming. Anybody who finds one should add it here.

## The readme's thesis, checked against the above

The readme carries two sentences of thesis. Checked against this document, one
is supported, one is supported only in part, and three specific words are not
supported by anything here.

Supported. "It runs on your own hardware; the models and the project history
never leave it." Read as a statement about what this project intends to build,
nothing above contradicts it, and the incumbent quote is what makes it worth
saying at all.

Supported in part. "A distribution and orchestration layer over the open
modular CAD suite" is supported: the module listing shows a modular suite with
the stages present, and a layer over it is what the shape decision in #3 is
about.

Not supported by this document, and listed for #114:

- "continuous" carries an implication this document cannot support, because the
  stages are already continuous in the suite at 1.1.3 and the paying incumbent
  ships sketch to G-code in one product. What is missing is a guided path, not
  continuity, and the readme should say the narrower thing.
- "beginner-friendly" is a judgement with no measurement behind it anywhere in
  the tree today. #13 is where the bar is set and measured. Until it is, this
  word is a promise rather than a claim, and the readme should not be read as
  comparing against anything.
- "stable UI" implies that something else is unstable. Nothing above measures
  interface consistency in either direction. Same treatment as the word above.

That is the list #114 is asked to act on. It is a request to narrow three
words to what can be backed, not a request to remove the thesis, and the
second sentence of the readme should not be touched at all.

## What this document does not settle

It measures no behaviour, because it drove neither program. Everything on the
consistency axis and everything about how the incumbent handles collaboration
and propagation is open, and no later argument may cite this document for any
of it. #13 owes the first. Somebody with an account and a reason to have one
owes the second, and until that exists this project compares itself to the
incumbent on price, on terms, and on where the data sits, which is all this
document establishes.
