# Security policy

This project will hold somebody's product designs and run a server on their
network. A problem in it needs a way to reach us that is not a public issue.

## Reporting a vulnerability

Use GitHub's private reporting on this repository:

https://github.com/iderex/reissbrett/security/advisories/new

The report stays private to the people who can act on it until an advisory is
published. Do not open a public issue for a vulnerability, and do not put the
details in a pull request body.

If you cannot use that route, say so in a public issue without the details and
ask for another one.

## What is in scope

This project's own code, and the way it configures, packages and ships
everything else. That includes the collaboration server and its protocol, the
version store and what it writes to disk, the extensions this project loads into
the CAD process, the packaging and the defaults an operator gets on a first run,
and this repository's own workflows and the supply chain behind them.

Configuration counts even where the code is somebody else's. A default that
exposes a port, a permission that is wider than it needs to be, or a dependency
pinned to something known bad is this project's defect, not the upstream
project's.

## What goes upstream

This project is a layer over an existing CAD suite and implements no geometry
kernel, no constraint solver and no toolpath generator. A defect in the suite
itself is reported to the suite, which asks for the same private route:

https://github.com/FreeCAD/FreeCAD/security/advisories/new

If you are not sure which side a defect is on, report it here and say why you are
unsure. Sorting that out is this project's work rather than yours.

## What you can expect

The report is read and answered. If it is a defect, the answer says so, says what
is being done, and says when an advisory will be published. If it is not, the
answer says why, in enough detail to argue with.

No response time is promised. Nobody is committed to one, and a missed promise in
a security policy costs more than an absent one. If a report has had no answer
and you want to know whether it arrived, ask on the report itself.

Credit is given in the advisory unless you ask for it not to be.

## Supported versions

There is no released version of this project yet, so nothing is supported and
nothing is being patched.

The rule that applies once there is one: the current release series receives
security fixes, and a series stops receiving them when the next one is published
unless the version policy says otherwise. #111 is where that policy is written
and what this section is filled in from. Until #111 lands, this paragraph states
the rule and not a promise about a version that does not exist.

## Coordinated disclosure

If a defect is confirmed, an advisory is published with the versions affected and
the versions that carry the fix. A reporter who has waited on a fix can publish
their own account whenever they choose, and this project does not ask anybody to
hold a finding indefinitely.
