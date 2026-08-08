# Governance

This says who decides in this repository, where a decision is written down, and
what to do about one you disagree with. It describes the arrangement that
exists. Where the arrangement is thin it says so, because a governance document
that describes a better structure than the one in place is the least useful
kind of document to read before contributing.

## Who decides

One person. The repository belongs to a single user account, and that account
is the only one with write access:

    gh api repos/iderex/reissbrett --jq '{owner: .owner.login, owner_type: .owner.type}'
    {"owner":"iderex","owner_type":"User"}
    gh api repos/iderex/reissbrett/collaborators --jq '[.[] | .login] | join(", ")'
    iderex

There is no committee, no steering group and no second maintainer. What makes a
decision reviewable here is not that a vote was held. It is that the reasoning
is written down where somebody can argue with it.

## Where a decision lives

A decision that shapes what everything else may assume is written as a numbered
document in `docs/decisions/`, one decision per file. The number is the issue
the decision was taken under, so a document can be read back to the argument
that produced it:

    git ls-tree -r --name-only origin/main docs/decisions
    docs/decisions/0003-a-layer-over-the-existing-suite.md
    docs/decisions/0005-the-pinned-suite-version.md
    docs/decisions/0006-what-a-version-of-a-model-is.md
    docs/decisions/0007-the-collaboration-model.md
    docs/decisions/0008-how-a-published-change-reaches-what-references-it.md
    docs/decisions/0009-the-committed-beginner-workflow.md
    docs/decisions/0011-the-delivery-form.md
    docs/decisions/0012-where-the-nearest-projects-fall-short.md
    docs/decisions/0013-the-bar-for-a-coherent-workflow.md
    docs/decisions/0037-the-process-boundary.md
    docs/decisions/prior-collaboration-attempt.md

The file without a number is a study rather than a decision. It records what an
earlier effort at this built and what state it is in, and the decisions above
reference it instead of restating what it found.

A decision is changed by writing the new one with its reasons, not by editing
the old one. The superseded document stays where it is, so that somebody
reading the replacement can see what was believed before and what moved it.

PROSE, NOT ENFORCEMENT, for both halves and for the numbering above. Nothing in
this repository reads `docs/decisions/`, nothing refuses a decision recorded
somewhere else, and nothing refuses an edit to a landed decision document. This
paragraph names no issue that owes a mechanism for any of it.

## How a change reaches the mainline

Every change starts as an issue and lands as a pull request. `CONTRIBUTING.md`
states it and the ruleset on `main` refuses a direct push:

    gh api repos/iderex/reissbrett/rulesets/20485819 --jq '{enforcement, bypass: .bypass_actors, rules: [.rules[].type]}'
    {"bypass":[],"enforcement":"active","rules":["deletion","non_fast_forward","pull_request"]}

The bypass actor list is empty, so the account that owns the repository goes
through a pull request as well.

That output names no status check. The checks in `.github/workflows/` run and
go red, and none of them stands between a change and the mainline today.
`.github/required-status-checks.md` is the authority for which names have
actually run, and #128 is the issue that holds the change to the ruleset.

## Disagreeing with a decision

Open an issue. Name the decision, the part of it you disagree with, and the
evidence that moves it. Every decision here is a document with its reasons in
it, so there is something specific to argue against.

If the disagreement is about a change that is in flight, it goes in that pull
request's body rather than in a comment underneath it. That is the rule
`CONTRIBUTING.md` states for everything else about a change, and a refusal is
not an exception to it.

The answer will come from one person, and that person may be the one who wrote
the decision being argued with. Knowing that before you start is worth more than
a procedure that pretends otherwise.

## If the arrangement changes

If write access is given to somebody else, this document is updated in the
change that grants it, and the collaborator command above is what shows whether
that has happened.

If the single maintainer stops, nothing here continues on its own. There is no
second person with access, no organisation behind the repository, and no
succession arrangement, and this document does not invent one. What remains in
that case is what the licence already gives you, which is the right to take the
code and carry on under the same terms. `LICENSE` is the GNU Affero General
Public License version 3.

## There is no code of conduct here yet

This document does not stand in for one. #106 is where it is being added, and
the missing input is the address a report would be sent to.

Do not send a conduct report through the private security advisory route in
`.github/SECURITY.md`. That route exists so a vulnerability is not public
before it is fixed. A conduct report is not a vulnerability, it produces no
advisory, and sending one there would have it handled as something it is not.

Until an address is published, this repository offers no private route for a
conduct report. That is a gap and it should not be read as anything else.
