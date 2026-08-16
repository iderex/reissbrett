# Verified signatures on the protected branch

Below are a request, the reasoning behind it and what it costs. No repository
setting changes here. Somebody changing the ruleset can work from this instead
of from memory, and can argue with the reasons before the change and not
reconstruct them afterwards.

## The correction this is written after

Issue #97 opens by saying that the reference gate requires verified signatures
on its protected branch. It does not. Read against the reference rather than
from anywhere else:

    gh api repos/iderex/jellyfin-plugin-sso/rulesets --jq '.[] | {id, name, target, enforcement}'
    {"enforcement":"active","id":18802863,"name":"Protect main and 5.0","target":"branch"}

    gh api repos/iderex/jellyfin-plugin-sso/rulesets/18802863 --jq '{bypass: .bypass_actors, required: [.rules[].type]}'
    {"bypass":[],"required":["deletion","non_fast_forward","required_status_checks","pull_request"]}

That is the only ruleset on the reference repository, so there is no second one
holding the rule, and `required_signatures` is not among its four rule types.

The correction was recorded in #97 before this document was written. It is
repeated here rather than only there because this document is what the ruleset
change is made against, and somebody reading it should not have to find out from
the issue that the sentence which started the whole thing was wrong. What it
changes is the argument's footing: this is a deviation upward from the
reference, argued on its own merits, rather than parity with something that
already holds the rule.

## The request

Add `required_signatures` to the ruleset that protects `main` on this
repository, with the bypass actor list left empty.

The current state of that ruleset, which nothing here changes:

    gh api repos/iderex/reissbrett/rulesets/20485819 --jq '{enforcement, bypass: .bypass_actors, required: [.rules[].type]}'
    {"bypass":[],"enforcement":"active","required":["deletion","non_fast_forward","pull_request"]}

The empty bypass list is part of the request rather than an incidental property
of it. A rule with a bypass actor exempts whoever is most able to be in a hurry,
and that is the case the rule is for.

## Why

The threat model already names this issue as part of what addresses T6:

    git grep -n 'T6. Somebody who tampers with what this repository publishes' origin/main -- docs/threat-model.md
    origin/main:docs/threat-model.md:92:**T6. Somebody who tampers with what this repository publishes.** The artefact

The argument is about where a chain of custody starts. #113 signs and publishes
release artefacts. #96 holds the supply chain around them. Neither says anything
about how the source got into the repository, so a chain that begins at the
artefact has a gap exactly where the commits are, and a gap in the middle of a
chain is the part somebody attacks rather than the ends.

The second reason is what this project produces. The committed path in #9 ends
at a program that makes a machine move. Whether a line of that machinery was
written by the person the history says wrote it is the same question, in a
different register, as whether the program should be run at all.

## What this does not buy

Worth writing down at the point the control is asked for, so it is not credited
later with more than it does.

A signature says who wrote a commit. It does not say the commit is good. N5 of
the threat model is somebody trusted to merge here, and it is accepted rather
than addressed, because there is one maintainer and their own commits verify. A
signature requirement does nothing for that case and it is not offered as if it
did.

It also does not reach a compromised workstation belonging to somebody who
legitimately has access, which is N3. A signing key on that machine signs
whatever that machine is told to sign.

So this closes the gap in T6's chain and leaves N3 and N5 exactly where the
threat model already put them.

## What it costs

Every contributor has to sign. That is a real barrier to a first contribution
and it is the kind that loses people quietly, because somebody who cannot make
the tooling work does not usually say so.

A signing failure in the middle of work reads as an obstacle with a one word way
around it. `CONTRIBUTING.md` already says that a signing failure is a stop
rather than something to work around, and it names both spellings of the way
around it so that neither looks like the clever answer.

Setting signing up is a fact of a clone and a machine. Nothing in this tree can
read whether a given clone did it, so no check here could warn somebody before
their first push, and the first time most people will meet the rule is a refused
merge.

## What a contributor has to do

Two things are easy to confuse and only one of them is refused today.

The sign-off is a text trailer, added by `git commit -s`, asserting the DCO in
`./DCO`. `.github/workflows/dco.yml` refuses a commit without one. That is
unaffected by this request and stays exactly as it is.

The signature is cryptographic and is what this request is about. The shortest
route that adds no new tool is an SSH key the contributor already has:

    git config --global gpg.format ssh
    git config --global user.signingkey ~/.ssh/id_ed25519.pub
    git config --global commit.gpgsign true

The same key has to be added to the GitHub account as a signing key, and that
step is the one people miss. A signature the platform cannot attribute to an
account is signed but not verified, and the rule being asked for reads verified.

The result is checkable before anything is pushed, against the platform rather
than against the local clone, which is what the ruleset will read:

    gh api repos/iderex/reissbrett/commits/1d38d149beb956fcd321d9ef13e91bc30dbe9035 --jq '.commit.verification | {verified, reason}'
    {"reason":"valid","verified":true}

Locally, `git show -s --format='%G? %an <%ae>' HEAD` prints `G` for a good
signature, and that is a convenience rather than the authority.

## What happens to work already on branches when this lands

Every commit that is on a branch here and not on the mainline is signed and
verified today. Read from the API the ruleset would use rather than from a local
checkout, on 2026-08-08:

    for b in $(git for-each-ref --format='%(refname:short)' refs/remotes/origin | grep -v 'origin/HEAD\|origin/main\|^origin$'); do for c in $(git rev-list origin/main..$b); do echo "$b $c verified=$(gh api repos/iderex/reissbrett/commits/$c --jq '.commit.verification.verified')"; done; done
    origin/proof/dco-bites 9d352fd739f2e53bf984b54567988296494b9e8d verified=true
    origin/proof/runner-facts a9c25ff747f68cd99e69ef7c6f46717861b706dd verified=true
    origin/proof/text-determinism-bom cf719f7507463296302afbec5948f07112f646e8 verified=true
    origin/proof/text-determinism-crlf bd361ab8df331bf50390be77fd8b966dff7cffc8 verified=true
    origin/proof/text-determinism-encoding a69d8552eda4fcf65e0b337c16eb41ff361d5fe7 verified=true

That is a moving population. Re-run the command before the ruleset is changed
rather than quoting this output, because the answer it gives is about the
branches that existed when it ran.

What happens to a branch that does contain an unsigned commit: the merge is
refused, and the repair is not to bypass the rule. It is the branch replaced end
to end, every commit cherry-picked onto a fresh one with signatures, the content
proven identical by comparing patch ids, the superseded pull request closed with
the reason in its body, and the checks waited out a second time. That cost has
been paid once elsewhere, after the build and the review were already done, and
it is written here so that whoever meets it knows the shape of the repair rather
than discovering that the one word alternative exists.

## What this document does not settle

It does not change a repository setting, and nothing in this repository does.
Changing the ruleset is an act somebody takes, against this document.

It does not make the requirement true. Nothing refuses an unsigned commit here
today, `CONTRIBUTING.md` says so at the point it states the rule, and it names
#97. Until the ruleset changes, that mark stays as it is and this document does
not soften it.

It says nothing about the required status checks on the same ruleset. That is a
separate act with its own issue, #128, and `.github/required-status-checks.md`
holds the names. The two are kept apart so they can be argued and reversed
separately.
