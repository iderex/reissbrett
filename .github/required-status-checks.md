# The checks the protected branch should require

This file is the authority for the literal check run names. When the ruleset on
`main` is changed, the names are copied from here and not retyped from memory,
because a required check is matched as a string: a name that does not match any
check run that actually happens blocks every merge until somebody notices, and a
name that is quietly changed in a workflow removes the gate without failing
anything.

Nothing in this file changes a repository setting. It produces the list, so that
the change to the ruleset is a single reviewed act rather than an accumulation of
half remembered names.

Why each of these checks exists at all, and where this repository's gate departs
from the one it is held to, is `docs/gate-parity.md`. That document argues the
deviations and points back here for the names, so the names are maintained in one
place and not two.

## Where the branch stands today

    gh api repos/iderex/reissbrett/rulesets --jq '.[] | {id, name, enforcement}'
    {"enforcement":"active","id":20485819,"name":"gate"}
    gh api repos/iderex/reissbrett/rulesets/20485819 --jq '{enforcement, bypass: .bypass_actors, rules: [.rules[].type], required: [.rules[] | select(.type=="required_status_checks") | .parameters.required_status_checks[].context]}'
    {"bypass":[],"enforcement":"active","required":["DCO sign-off","dependency-review","Reject Trojan Source Unicode","Audit workflows (zizmor)","Text encoding and line endings","Enforce greppable invariants"],"rules":["deletion","non_fast_forward","pull_request","required_status_checks"]}

A pull request is required, deletion and non-fast-forward are refused, and six
status checks are named. The six are the six rows under `## The names`, taken
from those rows rather than retyped, and each row carries the date it was
required, so a row and the ruleset cannot drift apart in silence.

The ruleset was changed on 2026-08-09, under #128, against this file as it stood
at `21592f8e4517580297f25769ef09da9d216df9c2`. That commit is the merge that
added the `Enforce greppable invariants` row, which is the sixth and the only one
read back from a run other than the reference pull request.

Two things in that output are worth reading rather than skipping. Neither of the
names under `## The names that must not be required` is in the required list, and
the bypass list is empty, both shown by the same command rather than asserted
beside it.

Before this, every gate this repository had built was advisory: it ran, it went
red, and the merge button stayed green. That is what changed. What it costs is now live and no longer
hypothetical: a required name that no run produces blocks every merge until
somebody with settings access changes the ruleset back, and no commit can undo
it.

## The names

Each name below was read back from a check run that happened on a real pull
request, not from the workflow file that was meant to produce it. Pull request
#125 is the reference, at head `df01f3c6b0989a6452fc3feda74f395f37fdfeb0`:

    gh api repos/iderex/reissbrett/commits/df01f3c6b0989a6452fc3feda74f395f37fdfeb0/check-runs --jq '.check_runs[] | "\(.name) || app=\(.app.slug)"'
    zizmor || app=github-advanced-security
    Text encoding and line endings || app=github-actions
    Audit workflows (zizmor) || app=github-actions
    dependency-review || app=github-actions
    Reject Trojan Source Unicode || app=github-actions
    DCO sign-off || app=github-actions
    Text encoding and line endings || app=github-actions
    Reject Trojan Source Unicode || app=github-actions

Two names appear twice because two of these workflows run on a branch push as
well as on a pull request, so the same name arrives from two runs. Which run
belongs to which workflow file, and which event produced it:

    for r in 31208280469 31208279678 31208279273 31208279031 31208278657; do gh api repos/iderex/reissbrett/actions/runs/$r --jq '"\(.id) \(.name) event=\(.event) path=\(.path)"'; done
    31208280469 text-determinism event=pull_request path=.github/workflows/text-determinism.yml
    31208279678 Workflow Security Analysis event=pull_request path=.github/workflows/zizmor.yml
    31208279273 Dependency review event=pull_request path=.github/workflows/dependency-review.yml
    31208279031 unicode-guard event=pull_request path=.github/workflows/unicode-guard.yml
    31208278657 DCO event=pull_request path=.github/workflows/dco.yml

### DCO sign-off

Emitted by `.github/workflows/dco.yml`. It runs on `pull_request` only, on the
types `opened`, `synchronize` and `reopened`, so it is capable of running on
every pull request that could be merged. Required on `main` since 2026-08-09.

What it costs to require: a pull request whose commits are not signed off cannot
be merged, which is the point.

### dependency-review

Emitted by `.github/workflows/dependency-review.yml`. It runs on `pull_request`
only. The name is the job id rather than a `name:` field, and the workflow says
in its own comment why it has none.

Required on `main` since 2026-08-09. Note that the tree declares no dependency
manifest of any kind, so the check passes over a diff with nothing in it to
review:

    git ls-files | grep -E 'package.json|go.mod|requirements.txt|Cargo.toml|\.csproj|pyproject.toml|Gemfile|pom.xml' ; echo "exit=$?"
    exit=1

When #21 lands lock files the same check starts having something to refuse,
under the same name.

### Reject Trojan Source Unicode

Emitted by `.github/workflows/unicode-guard.yml`. It runs on `push` to every
branch and on `pull_request` against every base branch, so it is capable on
every pull request. Required on `main` since 2026-08-09.

### Audit workflows (zizmor)

Emitted by `.github/workflows/zizmor.yml`. It runs on `push` to `main` and on
`pull_request` against every base branch, so it is capable on every pull
request. Required on `main` since 2026-08-09.

### Text encoding and line endings

Emitted by `.github/workflows/text-determinism.yml`. It runs on `push` to every
branch and on `pull_request` against every base branch, so it is capable on
every pull request. Required on `main` since 2026-08-09.

### Enforce greppable invariants

Emitted by `.github/workflows/greppable-invariants.yml`. It runs on `push` to
every branch and on `pull_request` against every base branch, so it is capable
on every pull request. Required on `main` since 2026-08-09.

This row is read back from a different pull request than the rest, because it
did not exist when #125 ran. Pull request #140, at head
`f1d0295d57bb09717ad637cc19266beaca82b213`:

    gh api repos/iderex/reissbrett/commits/f1d0295d57bb09717ad637cc19266beaca82b213/check-runs --jq '.check_runs[] | "\(.name) || app=\(.app.slug)"' | sort -u
    Audit workflows (zizmor) || app=github-actions
    DCO sign-off || app=github-actions
    dependency-review || app=github-actions
    Enforce greppable invariants || app=github-actions
    Reject Trojan Source Unicode || app=github-actions
    Text encoding and line endings || app=github-actions
    zizmor || app=github-advanced-security

What it costs to require: a pull request that carries a tracked file matching
one of the rules in `.github/invariants.txt` cannot be merged. What it costs
that the other rows do not is that the rules are data, so requiring this name
requires whatever that file says on the day the merge happens, and a rule added
there lands with the same weight as one argued here. That is the point of the
file and it is worth seeing before the name is required rather than after.

## The names that must not be required, and why

### Scorecard analysis

Emitted by `.github/workflows/scorecard.yml`, whose triggers are
`branch_protection_rule`, a weekly `schedule` and `push` to the default branch.
There is no `pull_request` trigger, and the workflow's own comment says why: the
pull request path is experimental upstream and cannot publish results.

So this check never runs on a pull request, and it has never run on one:

    gh api "repos/iderex/reissbrett/actions/workflows/scorecard.yml/runs?event=pull_request" --jq '.total_count'
    0

Requiring it would block every merge on this repository forever, with no way to
make the required check appear short of changing the ruleset back. That is the failure the rows below are written
against, and it is one name away.

### zizmor

This one is not emitted by a workflow job. It is created by the
`github-advanced-security` app when the SARIF that `Audit workflows (zizmor)`
uploads is ingested, which is why its app slug differs from every other row
above. The string comes from the analysis tool rather than from a `name:` field
in a workflow:

    git grep -n 'name:' -- .github/workflows/zizmor.yml
    .github/workflows/zizmor.yml:24:name: Workflow Security Analysis
    .github/workflows/zizmor.yml:44:    name: Audit workflows (zizmor)
    .github/workflows/zizmor.yml:53:      - name: Checkout Repository
    .github/workflows/zizmor.yml:58:      - name: Install uv
    .github/workflows/zizmor.yml:61:      - name: Audit workflows (SARIF for code scanning)
    .github/workflows/zizmor.yml:70:      - name: Upload SARIF
    .github/workflows/zizmor.yml:84:      - name: Fail on actionable findings

Requiring it would make the merge depend on the code scanning ingestion path
rather than on the gate the workflow already fails on, and the rule in #28 about
not renaming a check does not protect a name that no file in this tree sets.
`Audit workflows (zizmor)` is the row to require; this one is not.

## What has to happen when a gate is added

A workflow that adds a check adds its literal name here in the same change. A
name that lives only in a workflow file is a name nobody can require without
reading the file, and a name here that no run has produced is the Scorecard
failure above waiting to happen. Every row in this file names the run it was
read from, and a row that cannot name one does not belong in it yet.

The issue that adds the check also answers whether the name is being required,
in that issue, and the row here records which way it went. Required and not
required are both answers and silence is not, because a check missing from the
ruleset reads the same whether somebody weighed it and declined or nobody looked.
The six names above went in as one act against a list that was already complete.
That is available once. From here a name arrives on its own and is argued on its
own, and the issue that brings it is the only place that reading happens.

PROSE, NOT ENFORCEMENT, for both rules in this section, and no issue on this
board owes a mechanism for either. A ruleset is a repository setting, so no
commit can hold what it requires and nothing in this tree can compare the two.
Whether an issue answered a question is a judgement about an issue body, and
nothing in this repository reads one. What stands behind both sentences is a
person reading a pull request, and saying so is the whole of what this mark
does.

M1 has gates that are not built. #17 owes a format and a lint check, #19 owes a
unit test check, and #21 owes a restore that refuses a stale lock. Each of them
fixes its own check run name, adds its row here when it lands, and says whether
that name is being required. Until then this file lists what exists, which is not
the same as listing what is wanted.
