# Public funding routes, and what each one requires

Entry 7 of #1 asks whether to seek a public funding route and under whose
umbrella. This document is the groundwork that makes that question answerable.
It takes no position on whether to apply, and it does not rank the routes.

Every route below was read on the funder's own pages on 2026-08-08, and each
quoted line carries the command that produced it. A funder's conditions move
without notice, so nothing here is quoted into an application without re-running
the command first.

The quoted outputs pass through `iconv -f UTF-8 -t ASCII//TRANSLIT` because
tracked text in this repository is ASCII. It turns a currency sign into EUR and
a curly apostrophe into a straight one, and it changes nothing else.

## The FreeCAD Project Association grant program

The umbrella entry 7 of #1 names as the existing foundation in this space. Its
proposals are issues in a public repository:

    gh api repos/FreeCAD/FPA-grant-proposals --jq '{description, pushed_at}'
    {"description":"Submit grant proposals to the FPA by creating an issue in this repository.","pushed_at":"2025-11-16T23:30:06Z"}

The money and the calendar, from the program announcement:

    curl -sS https://fpa.freecad.org/programs/fpadf-announcement | sed 's/<[^>]*>/\n/g' | grep -E 'reserved 40,000|one fourth|submission deadline'
    The FreeCAD Grant Program aims to foster rapid development of FreeCAD and its ecosystem by giving small grants to developers and non-programming contributors to work on specific individual projects. Their work is funded by the community: grants are issued by the FreeCAD Project Association, which collects donations. The FPA has reserved 40,000 EUR for the program in 2026.
    Grants are evaluated and issued quarterly, with one fourth of the total budgeted funding available during each grant period, on the following schedule:
    Q1: March 10 submission deadline, voting complete by March 31
    Q2: June 1 submission deadline, voting complete by June 30
    Q3: Sept 1 submission deadline, voting complete by Sept 30
    Q4: Nov 15 submission deadline, voting complete by Dec 19

So the pool an application competes for in one period is a quarter of that
figure, shared with every other application approved in the same period, and the
page calls them small grants for specific individual projects in its own first
sentence. Individual grants outside the quarterly program are accepted at any
time and go to the same vote of the general assembly.

What it asks of an applicant before an award. A public track record, which is
the condition this project has to read carefully:

    curl -sS https://fpa.freecad.org/handbook/process/grant-program-rules.html | sed 's/<[^>]*>/\n/g' | grep -E 'Public track record' | iconv -f UTF-8 -t ASCII//TRANSLIT
    Public track record that confirms their qualification to accomplish the proposed project, such as commits or merged pull requests to the Git repository of FreeCAD or its ecosystem projects.

And, where the work is on a project the FPA does not control, an agreement with
that project:

    curl -sS https://fpa.freecad.org/handbook/process/grant-program-rules.html | sed 's/<[^>]*>/\n/g' | grep -E 'unwilling to accept' | iconv -f UTF-8 -t ASCII//TRANSLIT
    As neither the grant committee nor FPA members have control over ecosystem projects, the applicant is expected to demonstrate familiarity with external project's source code, e.g. merged pull requests, as well as a written agreement that ecosystem project maintainers are aware of the proposal, support the development, and are ready to review patches. The FPA cannot issue a grant for work on an ecosystem project if the said project is unwilling to accept proposed patches.

It asks for no legal entity. The rules put the contract between the FPA and an
individual, and a group applies through one person who represents it. That is
the one route read here where the absence entry 7 of #1 names as the cost of the
independent path does not block an application.

What it asks after an award:

    gh api repos/FreeCAD/FPA-grant-proposals/readme --jq '.content' | base64 -d | grep -A2 '^## Reporting' | sed '/^$/d'
    ## Reporting
    In most cases grantees will submit periodic reports to the FPA detailing their work to date. Those reports are submitted as Pull Requests to the [FPA's main git repository](https://github.com/FreeCAD/FPA/tree/main/reports).

Payouts are per stage, with the planning treated as the first stage, and the
review is a vote of the FPA general assembly on a proposal that is a public
issue in another organisation's repository. So an application puts the plan on
this board into somebody else's public process, which is a cost worth seeing
before it is paid rather than afterwards.

Two things about this route that are judgements rather than facts, and the
document says which. Whether the FPA reads a separate layer over FreeCAD as part
of its ecosystem is the FPA's call and no reading of these pages settles it. And
the ecosystem clause above is written for an applicant proposing work on somebody
else's project. Where the applicant maintains the project being funded, the
written agreement it asks for has nobody else to be with, so what the clause
requires in that case is a question for the grant committee and not something
this document answers.

## NLnet

The form of eligibility is wide, and the licence condition is absolute:

    curl -sS https://nlnet.nl/funding.html | sed 's/<[^>]*>//g' | tr -s ' \n' ' \n' | grep -oE 'In principle anyone can apply[^.]*\.'
    In principle anyone can apply: individuals, research organisations, non-profits, public institutions, companies of any size and type, etc.

    curl -sS https://nlnet.nl/funding.html | sed 's/<[^>]*>//g' | tr -s ' \n' ' \n' | grep -oE 'All projects must be free/libre/open source[^.]*entirety'
    All projects must be free/libre/open source: all scientific outcomes must be published as open access, and any software and hardware developed must be published under a recognised free and open source license in its entirety

The size and the calendar:

    curl -sS https://nlnet.nl/funding.html | sed 's/<[^>]*>//g' | tr -s ' \n' ' \n' | grep -oE 'We provide grants between [0-9.]+ and [0-9.]+ euro'
    We provide grants between 5.000 and 50.000 euro

    curl -sS https://nlnet.nl/funding.html | sed 's/<[^>]*>//g' | tr -s ' \n' ' \n' | grep -oE 'There is a deadline on the 3[^.]*month'
    There is a deadline on the 3rd of every odd month

What decides this route is not eligibility but the theme of the fund open at the
time. The one this project would have been read against has closed:

    curl -sS https://nlnet.nl/commonsfund/ | sed 's/<[^>]*>//g' | tr -s ' \n' ' \n' | grep -oE 'The thirteenth and final call[^.]*\.'
    The thirteenth and final call of NGI Zero Commons Fund closed on June 1st 2026.

What is active, and what the site says is coming:

    curl -sS https://nlnet.nl/funding.html | sed 's/<[^>]*>//g' | grep -oE 'Open Social Fund|Research and Higher Education Technology Fund' | sort -u
    Open Social Fund
    Research and Higher Education Technology Fund

    curl -sS https://nlnet.nl/funding.html | sed 's/<[^>]*>//g' | tr -s ' \n' ' \n' | grep -oE 'New funds will become active[^.]*\. We.re currently transitioning[^.]*\.' | iconv -f UTF-8 -t ASCII//TRANSLIT
    New funds will become active after the summer. We're currently transitioning from our Next Generation Internet programmes to the Open Internet Stack.

    curl -sS https://nlnet.nl/ | sed 's/<[^>]*>//g' | tr -s ' \n' ' \n' | grep -oE 'The goal of Restack[^.]*\.'
    The goal of Restack is to build a healthy Open Internet Stack.

So the position today is that an application is possible in form and there is no
fund open whose subject covers a CAD collaboration server. Neither active fund
does, and the fund named as coming is about the internet stack. This route is
therefore a re-read when the announced funds open, and the commands above are
what to re-run rather than this paragraph. Nothing is claimed here about the
scope of a fund that has not published its conditions.

## The Sovereign Tech Agency

Read because it is the German public route for open source infrastructure and
because entry 7 of #1 leaves the umbrella open. Its Fund states three conditions
that decide this project as it stands:

    curl -sS https://www.sovereign.tech/programs/fund | sed 's/<[^>]*>/\n/g' | grep -E 'must exceed|prototypes|looking for user-facing' | iconv -f UTF-8 -t ASCII//TRANSLIT
     The cost of the work described in the application must exceed EUR50,000 (current minimum).
    We do not finance the development of prototypes.
     looking for user-facing applications, such as messaging apps or file storage services. If this changes, we will announce it here.

The third line is the tail of a sentence whose subject is what the Fund is not
looking for, and the word the tag split off is a negation. The full sentence on
the page reads that they are currently not looking for user-facing applications.

Its licence condition, and what it invests in:

    curl -sS https://www.sovereign.tech/programs/fund | sed 's/<[^>]*>/\n/g' | grep -E 'OSI-approved' | iconv -f UTF-8 -t ASCII//TRANSLIT
     all code and documentation to be supported must be licensed such that it may be freely reusable, changeable, and redistributable. OSI-approved or FSF Free/Libre licenses are acceptable for code. Creative-Commons-like licenses for documentation may not include non-commercial or "no derivative" clauses.

What it invests in, in its own words, is open digital base technologies that are
vital to the development of other software or enable digital networking, with
libraries, package managers and protocol implementations as its examples.

The criteria it reviews against are prevalence, relevance, vulnerability, public
interest, activities and expertise, and the relevance criterion names health care
and industry among the sectors it counts. That is the closest this project comes
to the criteria and it is not enough while two of the three conditions above
stand against it. This project is user-facing by design, since the committed path
in `docs/decisions/0009-the-committed-beginner-workflow.md` is an interface a
person is walked through, and it has no code at all, which is the stage the
sentence about prototypes is written for.

The Fellowship program is the same wall approached from another side:

    curl -sS https://www.sovereign.tech/programs | sed 's/<[^>]*>//g' | tr -s ' \n' ' \n' | grep -oE 'paying maintainers of critical open source components[^.]*\.'
    paying maintainers of critical open source components for their work.

What it asks after an award is not reached here, because the requirements before
one already decide the route. What the same page states about the process is that
review takes up to ten weeks, scoping and selection up to eight more, the legal
step up to eight more, and that the time from submission to a possible contract
start is about six months.

## The Prototype Fund

This entry records a route that could not be read on this route, and it stays
that way rather than being filled in from anywhere else. Both fetch attempts were
refused by a bot check in front of the site:

    curl -sSL -o /dev/null -w '%{http_code} %{url_effective}\n' https://prototypefund.de/en/
    403 https://www.prototypefund.de/en/

So its eligibility conditions, its size, its round dates and its obligations are
not established here. The source is prototypefund.de and the date checked is
2026-08-08. Somebody who can open it in a browser should add the entry, and until
then this survey has one route named and unread. Nothing about its conditions
should be inferred from the fact that it is listed.

## The Open Technology Fund

Read and ruled out on subject rather than on eligibility:

    curl -sS https://www.opentech.fund/funds/internet-freedom-fund/ | sed 's/<[^>]*>//g' | tr -s ' \n' ' \n' | grep -oE 'The Internet Freedom Fund is[^.]*\.'
    The Internet Freedom Fund is the primary opportunity through which Open Technology Fund supports innovative global internet freedom projects.

A CAD workflow with a collaboration server for a workshop is not an internet
freedom project, and applying into that scope would waste the reviewers' time as
well as the applicant's.

## Routes named and not read

The European framework instruments and the national research and startup
instruments were not read on this route. They are named so that they are ruled
out deliberately rather than by omission, and nothing is stated here about their
conditions, because stating it would be describing a page nobody opened.

How this survey was bounded, so a later reader can widen it on purpose: it starts
from the umbrella entry 7 of #1 already names, adds the funders that publish
conditions for open source work in this region, and stops there. Four routes were
read on their own pages, one was named and could not be read, and the rest were
not read at all.

## What this project satisfies today, and what it does not

Checked against the mainline rather than against intent.

A recognised free and open source licence, which every route read here requires:

    gh api repos/iderex/reissbrett --jq '.license.spdx_id'
    AGPL-3.0

What the licence still needs around it is #100, and the entry in #1 about whether
the extensions carry the same licence is open.

A public repository and a public record of the work, which most routes ask for in
one form or another:

    gh api repos/iderex/reissbrett --jq '{visibility, created_at}'
    {"created_at":"2026-08-06T01:51:17Z","visibility":"public"}
    gh issue list --repo iderex/reissbrett --state all --limit 400 --json number --jq 'length'
    122
    git ls-tree -r --name-only origin/main -- docs/ | wc -l
    30

The last two numbers move as work lands, this file included, so a reader re-runs
them rather than quoting these. They were 118 and 14 when this section was
written.

No code, which is the fact two of the conditions above turn on:

    git ls-tree -r --name-only origin/main | grep -cE '\.(go|py|cs|rs|ts|js|cpp|c|java)$'
    0

It is why the Sovereign Tech Fund's sentence about prototypes lands on this
project, and it is why the FPA's public track record criterion cannot be
satisfied out of this repository. Whether it is satisfied from anywhere else is
not a fact of this tree, and this document does not claim either way.

A governance document and a code of conduct, and no citation file, which is the
third thing the pattern below looks for:

    git ls-tree -r --name-only origin/main | grep -iE 'CODE_OF_CONDUCT|GOVERNANCE|CITATION' ; echo "exit=$?"
    .github/CODE_OF_CONDUCT.md
    .github/GOVERNANCE.md
    exit=0

This paragraph said the repository had neither, and it said so from the day #143
landed both under #106 until this edit. What #106 is still open on is narrower
and is not the files existing: no message has been sent through the contact
route the code of conduct names, so whether a report would arrive is untested.
A route that has not been tested is not a route a funder is owed a claim about.
Whether a legal entity exists is not a fact of this tree either, so it is not
asserted here in either direction. What is recorded above is what
each route requires: the FPA contracts with an individual, and the routes that
need more than that say so on their own pages.

## Requirements that map to work this board already holds

Nothing found in this survey needs a new issue. Each requirement that this
project would reasonably want whatever it decides about funding is already
carried:

    for n in 100 106 111; do gh issue view $n --repo iderex/reissbrett --json number,state,title --jq '"#\(.number) \(.state) \(.title)"'; done
    #100 OPEN Land what the licence needs around it: headers, notice consistency, and the analysis against the suite
    #106 OPEN Add governance and a code of conduct
    #111 OPEN State the version policy and what a version number promises

The two requirements that no issue carries are a public upstream track record and
a legal entity, and neither is something to do anyway. The first is constrained by
#4 and by entry 3 of #1, which are both open and which decide whether this
project ever proposes changes upstream. The second is entry 7's own subject.

## What this document does not decide

Whether to apply, and under whose umbrella. Entry 7 of #1 is where that is
argued, and it references this document for what the routes require. It also does
not rank the routes, because the ranking depends on the answer to entry 7 rather
than the other way around.
