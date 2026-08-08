# The gate this repository is held to, and every deviation from it

The standard for this repository's gate is the one already running on the SSO
plugin at github.com/iderex/jellyfin-plugin-sso. It is a public repository, its
ruleset is readable, and holding to something that exists is worth more than
holding to a standard invented here.

Parity is not copying. That gate protects a plugin loaded into somebody else's
process. This repository is three things at once: extensions loaded into
somebody else's process, a desktop workflow, and a network service holding a
company's designs. Some of the reference does not apply here, and some of what
this project needs is not there at all. Both directions are deviations and both
owe a reason.

This document records the deviations. It does not hold the list of literal check
run names that the branch should require, because that list already has an
owner: `.github/required-status-checks.md` is the authority, every row in it
names the run the name was read back from, and a second list here would drift
against it. Where this document needs a name, it points there.

## How this was read

Everything below was observed on 2026-08-08, at this repository's `main`:

    git rev-parse origin/main
    efc494a53cf05f9daa3b6318fe37eef12ede9396

The reference gate's ruleset, by literal check run name:

    gh api repos/iderex/jellyfin-plugin-sso/rulesets --jq '.[] | select(.name=="Protect main and 5.0") | .id'
    18802863
    gh api repos/iderex/jellyfin-plugin-sso/rulesets/18802863 --jq '{enforcement, bypass: .bypass_actors, required: [.rules[] | select(.type=="required_status_checks") | .parameters.required_status_checks[].context]}'
    {"bypass":[],"enforcement":"active","required":["build","ABI floor build","Package (JPRM) / Build package","Package (JPRM) / Generate SBOM","CodeQL","Analyze (csharp)","DCO sign-off","Deterministic PR-hygiene checks","Enforce greppable invariants","Reject Trojan Source Unicode","Audit workflows (zizmor)","prettier","dependency-review"]}

What this repository has today, which is workflows and no code:

    gh api repos/iderex/reissbrett/git/trees/main?recursive=1 --jq '[.tree[] | select(.path|startswith(".github/workflows/")) | .path] | join(" ")'
    .github/workflows/dco.yml .github/workflows/dependency-review.yml .github/workflows/scorecard.yml .github/workflows/text-determinism.yml .github/workflows/unicode-guard.yml .github/workflows/zizmor.yml

The reference gate requires thirteen contexts. It also runs work that is not
required, and the cheapest honest reading of what that work is, is the list of
its workflow files:

    gh api "repos/iderex/jellyfin-plugin-sso/git/trees/HEAD?recursive=1" --jq '[.tree[] | select(.path|startswith(".github/workflows/")) | .path] | join(" ")'
    .github/workflows/build.yml .github/workflows/codeql.yml .github/workflows/dco.yml .github/workflows/dependency-review.yml .github/workflows/dotnet.yml .github/workflows/e2e-login.yml .github/workflows/fuzz.yml .github/workflows/manifest-freshness.yml .github/workflows/nightly-betas.yml .github/workflows/opengrep.yml .github/workflows/pr-hygiene.yml .github/workflows/prettier.yml .github/workflows/publish-beta.yml .github/workflows/publish-failure-alert.yml .github/workflows/publish-jf12-beta.yml .github/workflows/publish-jf12-stable.yml .github/workflows/publish.yml .github/workflows/regenerate-manifest.yml .github/workflows/scorecard.yml .github/workflows/stryker-mutation.yml .github/workflows/unicode-guard.yml .github/workflows/wiki-lint.yml .github/workflows/zizmor.yml

The limit on that last command, stated because it is easy to read past. A file
name is not a description. What each of those workflows does was not read for
this document, so where a row below is placed against one of them, the placement
is from the name and from what the reference's own required list already says,
and it is written as such.

Some of the reasoning below was written later than the date above, and the
commands in it were run at a different commit:

    git rev-parse origin/main
    3754f521eec8c2e74c4d253c83f88f382feb9f23

Each of those blocks says so where it pastes its output. Moving the date at the
top of this section instead would restamp readings that were never re-run, which
is the defect this file is most exposed to and the one it is least able to
afford.

## The required contexts, one by one

Thirteen rows. Each says adopt, adapt or drop, why, and which issue on this
board delivers it.

**build.** Adapt. The reference builds one artefact from one language. This
repository will have more than one surface and the languages are #14's to name,
so one build job becomes one per surface. #16 delivers the build from a clean
checkout that this attaches to.

**ABI floor build.** Adapt. Its purpose here is the same and its mechanism is
not: the reference proves it still loads into an older host, and the equivalent
question is what happens when the CAD suite is not the version this layer was
built for. #35 refuses such a version and #34 measures what moves across
versions. Whether that belongs in the pull request gate or in a scheduled run is
the argument, and it is not settled here, because the answer depends on how long
#34's harness takes and that has not been measured.

**Package (JPRM) / Build package.** Adapt. The reference packages a plugin with
one tool. What an operator installs here is decided by #11 and built by #108,
and #108 carries both the bundle and the container image, so whether this is one
check or two follows from what #108 produces rather than from the reference's
shape.

**Package (JPRM) / Generate SBOM.** Adopt in substance and not in name. #22
produces a bill of materials on every build, and it owes a harder version than
the reference's, because a bundle that redistributes the suite has to list what
it redistributes rather than only this project's own dependencies.

**CodeQL** and **Analyze (csharp).** Adopt as one thing, and this is the row
where the reference is not followed exactly. #89 delivers code scanning for
whatever language #14 names, which produces the `Analyze (...)` row. The other
name is the deviation. On the reference's default branch head, `Analyze
(csharp)` appears among the check runs and is produced by `github-actions`,
while `CodeQL` does not appear at all:

    gh api "repos/iderex/jellyfin-plugin-sso/commits/$(gh api repos/iderex/jellyfin-plugin-sso --jq '.default_branch')/check-runs" --jq '.check_runs[] | "\(.name) || app=\(.app.slug)"' | sort -u
    ABI floor build || app=github-actions
    Analyze (actions) || app=github-actions
    Analyze (csharp) || app=github-actions
    Analyze (javascript-typescript) || app=github-actions
    Audit workflows (zizmor) || app=github-actions
    build || app=github-actions
    Enforce greppable invariants || app=github-actions
    Package (JPRM) / Build package || app=github-actions
    Package (JPRM) / Generate SBOM || app=github-actions
    prettier || app=github-actions
    Reject Trojan Source Unicode || app=github-actions
    Report any workflow that concluded non-success on the default branch || app=github-actions
    Scorecard analysis || app=github-actions
    submit-nuget || app=github-actions
    wiki-lint || app=github-actions

That is consistent with `CodeQL` being produced by the code scanning ingestion
rather than by a workflow job, which is the same shape this repository already
argued about `zizmor` against `Audit workflows (zizmor)` in
`.github/required-status-checks.md`. It is consistent with it and it does not
establish it, because a check run absent from a push head can also be one that
runs only on a pull request. So the deviation is recorded as a question with a
default rather than as a finding: #89 requires the row a workflow file names,
and if the ingestion row is wanted as well, the reason is written down then.

**DCO sign-off.** Adopted already, under the same literal name, and the
certificate the failure message points at is in the tree from #24. The name is
in `.github/required-status-checks.md` with the run it was read back from.

**Deterministic PR-hygiene checks.** Adopt. #92 delivers it, and it checks this
project's own rules rather than the reference's, so the name may match while the
content does not.

**Enforce greppable invariants.** Adopt. #91 delivers it. The sentence that
stood here said the invariants cannot be written until there is code to grep,
and put this row later in the order than its position suggests. That is wrong
for two of the six rules #91 names. Both are about tracked text rather than
about code, and both are decidable over the tree as it stands. Read at
`3754f521eec8c2e74c4d253c83f88f382feb9f23`.

The first is that the pinned suite version appears in no file other than
`docs/decisions/0005-the-pinned-suite-version.md`, which is the file that pins
it. It appears in eight:

    git grep -l '1\.1\.3' origin/main -- . | wc -l
    8
    git grep -l '1\.1\.3' origin/main -- .
    origin/main:docs/decisions/0003-a-layer-over-the-existing-suite.md
    origin/main:docs/decisions/0005-the-pinned-suite-version.md
    origin/main:docs/decisions/0006-what-a-version-of-a-model-is.md
    origin/main:docs/decisions/0009-the-committed-beginner-workflow.md
    origin/main:docs/decisions/0011-the-delivery-form.md
    origin/main:docs/decisions/0012-where-the-nearest-projects-fall-short.md
    origin/main:docs/decisions/prior-collaboration-attempt.md
    origin/main:docs/version-policy.md

Seven of the eight carry it inside evidence: a command run against the upstream
release list, a table of release dates, a sentence about which series the
extensions route supports. A rule written to refuse a hardcoded version refuses
all seven, so what #91 has to get right is not the pattern but the scope that
separates a document quoting the version from a unit that depends on it, and
that scope is what decides whether the rule earns its place at all. The near
miss to spend the effort on is the document citing the version legitimately, not
the file hardcoding it obviously. This row does not quote the pinned version, so
the count above does not move because the row exists, and the fact that it had
to be written that way is the scoping problem in miniature.

The second finds nothing, which is the state #91's third condition is written
against rather than a clean result. No absolute path from somebody's machine is
in a tracked file:

    git grep -nE '[A-Z]:\\Users|/home/[a-z]+/|/Users/[a-z]+/' origin/main -- . ; echo "exit=$?"
    exit=1

A pattern that has never matched anything is a pattern nobody has watched bite,
so that rule ships with a fixture it refuses or it does not ship.

The remaining four do wait on code, and naming which is what keeps this row
honest in the other direction. The import rule from #37 needs units to import,
the display rule from #31 needs a test job, the vocabulary rule needs #63's
list, and the rule about a copy of or a patch against the suite's source needs
#4.

**Reject Trojan Source Unicode.** Adopted already, under the same literal name,
and it runs on this repository today.

**Audit workflows (zizmor).** Adopted already, under the same literal name, and
it runs on this repository today.

**prettier.** Adapt. One formatter for one language becomes one per language
named in #14, delivered by #17. A single required context covering several
formatters is a choice #17 makes rather than one this document makes for it.

The row as first written sent the whole of it to #17, and the reference's own
formatter is wider than that. Markdown is inside its glob, read at
`3754f521eec8c2e74c4d253c83f88f382feb9f23`:

    gh api repos/iderex/jellyfin-plugin-sso/contents/.github/workflows/prettier.yml --jq '.content' | tr -d '\n' | base64 -d | grep -n 'prettier_options'
    36:          prettier_options: '--check **/*.{js,html,md,css,scss}'

and `prettier` is a required context there, so the reference formats its
documentation under the same required name it formats its code under. Split here
rather than followed. #17 takes the code half, because it is per language and
waits on #14. #95 takes the documentation half, because #95 is the issue that
owes documentation checking and because that half is deliverable before a
language is named. There is something for it to run over and nothing for #17's
half to run over:

    git ls-tree -r --name-only origin/main -- docs/ | wc -l
    18
    git ls-tree -r --name-only origin/main | grep -cE '\.(go|py|cs|rs|ts|js|cpp|c|java)$'
    0

Whether the two halves end up behind one required name or two is #17's to settle
when its gates land, and nothing here decides it.

**dependency-review.** Adopted already, under the same literal name. It has
nothing to review until #21 lands a lock file, which
`.github/required-status-checks.md` records against the row rather than leaving
as a surprise.

## What the reference runs that it does not require

Read off the workflow list above, with the same limit: these are placements from
names, and each names the issue here that would deliver the same thing.

`stryker-mutation` is mutation testing, which is #93. `fuzz` is fuzzing, which is
#94. `e2e-login` is an end to end harness on its own trigger, which is the shape
#99 takes here. `opengrep` is a second analyser with a different lens, which is
#90. `wiki-lint` is documentation linted the way code is, which is #95.
`scorecard` exists in both trees already. The coverage threshold the reference
pins on the modules that decide security outcomes, rather than on the whole
codebase, is #20, and the shape matters more than the number: a floor on
everything is a floor nobody defends.

Two of those placements are a name and nothing more, and both issues ask this
document for the line of reasoning behind them rather than for the placement. It
is written here. Both blocks were read at
`3754f521eec8c2e74c4d253c83f88f382feb9f23`.

`stryker-mutation`, and #93. Mutation testing is worth its cost where the test
suite is the only thing standing between a defect and an outcome nobody can
re-run: the version store in M3, where a restore that silently drops an entry
looks exactly like a restore that worked, and the refusal paths in M6, where a
collision check that silently passes looks exactly like a clean job. Coverage
says a line executed and says nothing about whether removing it would have been
noticed, and that gap is the whole reason for a second measurement rather than a
higher threshold on the first. It reports rather than gates, and the reference
holds the same position by its triggers:

    gh api repos/iderex/jellyfin-plugin-sso/contents/.github/workflows/stryker-mutation.yml --jq '.content' | tr -d '\n' | base64 -d | sed -n '/^on:/,/^permissions:/p' | grep -vE '^\s*#|^$'
    on:
      schedule:
        - cron: "41 5 * * 1"
      workflow_dispatch:
    permissions:

A weekly run and a manual one, and no pull request trigger. A mutation score
moves slowly and carries a tail of survivors nobody can act on inside a review,
so a gate on it refuses changes for reasons that are not about the change. Which
surfaces it runs over, and what happens to a score that falls, are #93's and
this row does not take them.

`wiki-lint`, and #95. The placement was made from a file name and the name
points somewhere else, so what the reference runs there was read rather than
assumed:

    gh api repos/iderex/jellyfin-plugin-sso/contents/.github/workflows/wiki-lint.yml --jq '.content' | tr -d '\n' | base64 -d | sed -n '3,6p;41,44p'
    # The wiki is a separate repository with no pull-request gate, so it can drift
    # from the code silently (#873). This is the executable guard for that drift: it
    # clones the live wiki and fails on broken internal anchors, file:line citations
    # (the rule is to cite modules/types, not paths that move), and dead source links.
          - name: Clone the wiki
            # The wiki is public; a read-only anonymous clone needs no credentials.
            run: git clone --depth 1 "https://github.com/${GITHUB_REPOSITORY}.wiki.git" "${RUNNER_TEMP}/wiki"
          - name: Lint the wiki against the code tree

So it resolves references from a separate repository against the code tree and
formats nothing. That is one of the mechanical items #95 lists, the broken link
to a file in the tree, and it is not the formatter half, which is the `prettier`
row above. Both belong to #95 and they arrive from two different places in the
reference, which is why neither row alone accounts for it.

The publishing workflows have no counterpart here yet and are not a gate. What
they correspond to is #113 for signing and publishing artefacts and #96 for
holding the supply chain around them.

## What this repository adds that the reference does not have

**Text encoding and line endings.** Running here already, and no workflow in the
reference's list corresponds to it. It exists because M3 stores model data and
compares it byte for byte, so a file rewritten in transit is a fixture that
proves nothing. The reference has no such register and does not need one.

**A refusal of a suite version this layer was not built for.** #35. The
reference's nearest neighbour is its ABI floor build, and the two are placed
against each other above rather than counted twice.

**Evidence under concurrent editors.** #60. A plugin does not hold a company's
designs while several people edit them, so the reference has nothing here. This
is a deviation upward and it is the clearest one on the list.

**Architecture rules turned into tests.** #98. What it refuses comes from #4's
rule about where this layer attaches, and from #40's rule that the store never
loads the suite. Neither has a counterpart in a single-plugin tree.

**The machining checks.** M6, and #75, #76 and #78 in particular. This is the
deviation that needs the most care rather than the least, because it goes in two
directions at once. Software that makes a machine move needs more evidence than
a plugin, and the evidence that matters most cannot sit in a pull request gate:
#78 keeps the harness that touches real hardware separate and named for what it
is, precisely so that a green pull request is never read as a proven machine.
Adding those checks to the gate would produce the opposite of what they are for.

**Verified signatures on the protected branch.** #97, and
`docs/verified-signatures.md` is where the request is argued. The reference's
ruleset holds four rule types and signatures is not one of them, read on
2026-08-08:

    gh api repos/iderex/jellyfin-plugin-sso/rulesets/18802863 --jq '{bypass: .bypass_actors, required: [.rules[].type]}'
    {"bypass":[],"required":["deletion","non_fast_forward","required_status_checks","pull_request"]}

So this is a deviation upward rather than a row this repository is behind on,
and the reference is not weaker for lacking it. It belongs here because #113
signs release artefacts and a chain of custody that starts at the artefact does
not reach the commits it was built from.

**A vocabulary check and a check run name check over the documentation.** #95.
The `wiki-lint` and `prettier` rows above account for the link resolution and
the formatting, and these two have no counterpart in the reference at all. The
vocabulary check reads #63's list and refuses a term used in a document that is
not in it; it matters here because the words the guided path uses are what
element 2 of the bar in #13 counts, and a document drifting from the list makes
that count a measurement of nothing. The check run name check refuses a
reference to a check run name that no workflow emits, and the failure it
prevents is the one `.github/required-status-checks.md` is most exposed to: a
renamed check leaves a required name behind that nothing produces, and the
branch then holds a gate that can never go green or one that never runs,
depending on which side the rename fell. Both wait on what they read, #63's list
and a set of workflows emitting names, and neither is a row this repository is
behind the reference on.

**The real suite round trip on a schedule.** #99. It is listed here as well as
against `e2e-login` above, because the reference's end to end run and this one
are the same shape holding different content, and #99's own body calls it a
deviation upward.

## Where this leaves the branch today

Nothing above is required by the ruleset. This repository's ruleset requires a
pull request, refuses deletion and non-fast-forward, and names no status check:

    gh api repos/iderex/reissbrett/rulesets/20485819 --jq '{enforcement, bypass: .bypass_actors, rules: [.rules[].type], required: [.rules[] | select(.type=="required_status_checks") | .parameters.required_status_checks[].context]}'
    {"bypass":[],"enforcement":"active","required":[],"rules":["deletion","non_fast_forward","pull_request"]}

So the deviation that matters most today is not any row above. It is that the
reference's ruleset requires thirteen contexts with no bypass actor, and this
one requires none, which makes every gate here advisory.
`.github/required-status-checks.md` holds the names that are ready to be
required and the two that must not be, and it says in its own words that nothing
in it changes a repository setting. Changing the ruleset is an act somebody
takes against that list, and #128 is where that act is recorded. It was opened
because this section, as first written, said no issue held it and left the gap
as a sentence in a document, which is the shape this whole file exists to avoid.
#97 is the neighbouring act on the same ruleset for verified signatures, argued
in `docs/verified-signatures.md`, and the two are separate so they can be argued
and reversed separately.

## What this document does not settle

It does not decide whether an element belongs in the pull request gate or in a
scheduled run. Two rows above say so explicitly, the suite compatibility check
and the machining harness, and both wait on a measurement rather than on an
argument.

It does not name languages. Four rows depend on #14 and say so rather than
guessing.

It does not restate the check run names. That list is
`.github/required-status-checks.md` and it is maintained there.
