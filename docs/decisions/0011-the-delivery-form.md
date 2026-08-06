# The delivery form an operator installs

This is the decision behind issue #11. The readme uses the word distribution and
this is where that word is cashed out.

## The decision

For the first release there are two artefacts and they are delivered
differently.

The desktop side is a bundle. One download carrying the CAD suite, this
project's extensions, the settings, the theme and the first run experience. The
bundle is the product, and it is the thing the readme is written about.

The collaboration server is a container image with a published compose example,
which is what #109 delivers. It is not part of the bundle and it is not
installed by a person who only wants to make a part on their own machine.

A set of extensions publishable into an installation the operator already has
is a third route. It exists, it is supported, and it carries a stated reduction
in what is promised. It is not the product.

## Why the bundle is the product

Everything M5 promises depends on the workspace opening in a known state. That
is #62, and it is a test rather than an opinion. A known state cannot be
delivered on top of an installation this project did not put there, because the
operator's own preferences, their installed addons and their suite version are
all inputs to what the person sees when the window opens. The extensions-only
route can improve that state. It cannot guarantee it.

The second reason is that the pin in #5 becomes true by construction. A bundle
carries one version of the suite and no other, so the compatibility harness in
#34 has one target and every bug report names a version without anyone having
to ask.

The third reason is the audience. A person who has never used the suite is not
the person who wants to install an addon manager entry into an existing
installation. They want one download.

## What the bundle costs

It means redistributing somebody else's large C++ application on every platform
this project supports, and doing it again on every version bump. Upstream ships
its own binaries and the shapes it ships are visible:

    gh api repos/FreeCAD/FreeCAD/releases/tags/1.1.3 --jq '[.assets[].name] | join("\n")'
    FreeCAD_1.1.3-Linux-aarch64-py311.AppImage
    FreeCAD_1.1.3-Linux-aarch64-py311.AppImage-SHA256.txt
    FreeCAD_1.1.3-Linux-aarch64-py311.AppImage.zsync
    FreeCAD_1.1.3-Linux-x86_64-py311.AppImage
    FreeCAD_1.1.3-Linux-x86_64-py311.AppImage-SHA256.txt
    FreeCAD_1.1.3-Linux-x86_64-py311.AppImage.zsync
    FreeCAD_1.1.3-macOS-arm64-py311.dmg
    FreeCAD_1.1.3-macOS-arm64-py311.dmg-SHA256.txt
    FreeCAD_1.1.3-macOS-x86_64-py311.dmg
    FreeCAD_1.1.3-macOS-x86_64-py311.dmg-SHA256.txt
    FreeCAD_1.1.3-Windows-x86_64-py311-installer.exe
    FreeCAD_1.1.3-Windows-x86_64-py311-installer.exe-SHA256.txt
    FreeCAD_1.1.3-Windows-x86_64-py311.7z
    FreeCAD_1.1.3-Windows-x86_64-py311.7z-SHA256.txt
    freecad_source_1.1.3.tar.gz
    freecad_source_1.1.3.tar.gz-SHA256.txt

Five platform artefacts across three operating systems, each of which would
become a bundle this project builds, tests, signs and publishes. That is the
continuing burden, and the platform list below is where it is held down to
something this project can actually carry.

It also costs a download in the hundreds of megabytes for somebody who already
has the suite installed, which is the reason the extensions route is kept alive
rather than dropped.

## Platforms for the first release

The first release covers Linux x86_64 and Windows x86_64.

Those two are where the people this project is aimed at are, and both are
shapes upstream already publishes for, so the bundle is an assembly rather than
a build of the suite from source.

macOS is not covered by the first release. The reason is not the operating
system, it is that a macOS artefact somebody can open without being told to
override a warning needs signing and notarisation through an Apple account with
its own release path, and this project has neither today. #113 is where signing
is built, and macOS is revisited when it exists rather than promised now.

Linux aarch64 is not covered either. Upstream publishes it, so the artefact
would assemble, but this project has no aarch64 machine to run the machining
and interface work in M5 and M6 against, and shipping a bundle nobody has run
the committed path on is a promise made from a build log.

A user on a platform the first release does not cover is told exactly that, and
pointed at the extensions route with the reduction in promise stated in the
same sentence rather than in a footnote. #110 is where the first run says it.

## What the extensions route does not get

Somebody choosing the extensions route is choosing to keep their own
installation, and this is what that costs them, stated plainly so it is not a
surprise after the fact.

They do not get the known starting state. The workspace opens on top of their
preferences and their installed addons, so #62 does not hold for them and the
guided path may look different from every screenshot in the documentation.

They do not get the pin. Their suite version is whatever they installed, so
#35 is what stands between them and a version this layer was not built for, and
a refusal to load is the correct outcome rather than a failure.

They do not get the theme and the first run experience as a guarantee, because
both can be overridden by settings this project did not write and will not
silently replace.

They do get the workflow, the vocabulary, the document model, the version store
and the collaboration client, because none of those depend on the starting
state.

A bug report from the extensions route names the operator's suite version and
their addon list, or it cannot be answered. That is a real cost and it belongs
to this route rather than to the product.

## The relationship to the pin in #5

The bundle makes the pin true by construction: one bundle carries one suite
version, and there is nothing for an operator to install alongside it that this
project treats as supported.

The extensions route is where the range question is real, and #5 answers it
there. The two documents are consistent by design and neither may be changed
without the other, because a pin the delivery form contradicts is worse than no
pin at all.

## The redistribution obligations this creates

Shipping the bundle is redistribution, and redistribution is the act that makes
a licence bite. Three obligations follow and #101 is where they are carried.

The suite is LGPL-2.1, which is a fact this document reads rather than assumes:

    gh api repos/FreeCAD/FreeCAD --jq '.license.spdx_id'
    LGPL-2.1

So the bundle carries the licence text and the copyright notices, and it makes
the corresponding source available on the terms that licence sets.

The bundle also carries everything the upstream artefact carries, which is a
large third party dependency set with its own notices. Those are reproduced
rather than summarised, and #101 is the issue that produces a notice file an
operator can actually read.

Whether this project's own code and the redistributed suite are one combined
work or an aggregation is a licensing question this document does not answer.
It is entry 2 of #1, it is open, and the bundle is precisely what makes it
concrete rather than theoretical. Nothing here decides it in either direction,
and #100 is where the answer lands when it is given.

## What builds this

#108 builds the bundle this document describes, #109 publishes the compose
example for the server, and #110 is what the first run says when something in
the operator's environment is wrong.
