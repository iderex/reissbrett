# The bar a coherent workflow is measured against

This is the decision behind issue #13. Coherent is the claim in the readme, and
it is the kind of word a project can spend years believing about itself. This
document turns it into numbers, and it is written before any interface exists,
so that it says what this project is aiming at rather than describing what it
happened to build.

The path being measured is the one in the committed workflow document, which is
the decision behind #9. Every element below is written against those six stages,
and an element that could not name a stage would not belong here.

## What kind of numbers these are

Every number in this document is a target chosen while the document was written.
None of them is a measurement of this project, because there is nothing here yet
to measure:

    git ls-tree -r --name-only origin/main | grep -cE '\.(go|py|cs|rs|ts|js|cpp|c|java)$'
    0

So they are claims about what would be good enough, not observations. #67 is
what turns them into measurements, and #67 reports a miss at the same size as a
pass, with no aggregate line that hides one.

A target that is missed is information. The response is either work on the path
or an argued change to the target, and #67's fourth condition already requires
that a change is recorded as a change with its reason and that this document is
updated rather than replaced. A target adjusted quietly after the number is
known is the failure this whole document exists to prevent, and it is the same
failure as an observation discarded after the fact, which the section below
handles.

## The elements

### 1. Time and steps to a first finished part

The headline number, and the easiest to game, so it carries the most definition.
#61 is what delivers it, because the time to a first finished part is the time
the guided path takes, and #69 is the worked example a person is taught on.

The numbers. A median of ninety minutes or less. At least seven observers out of
ten reaching a finished part at all. A median of three or fewer off-product
lookups per run.

The units. Minutes of wall clock, from the moment the program opens to the
moment the machine program exists on disk. A count of observers who finished,
out of the number who started. A count of off-product lookups, where a lookup is
any moment the person consults something that is not this product, including a
search engine, the suite's own community forums and the observer.

What counts as finished. Exactly the exit condition of stage 6: one solid, a
drawing page, and a post-processed program on disk for the machine class chosen
in entry 5 of #1, with the collision refusal in #75 not triggered. Not a shape
that looks right, and not a program the observer thinks would probably run.

What help is allowed. The guided path, the messages, the sample project from
#69, and this project's own documentation. Nothing else. No person, including
the observer, answers a question during the run.

The procedure. Ten observers, each observed once. A clean profile, on a platform
#11 covers. A written task that uses only the vocabulary from #63, handed over
without spoken elaboration. The observer records the clock and the lookups and
does not speak. Each observation is written up before the next one starts.

Why ninety. It is the length of a session somebody will sit through in one go,
and it is deliberately ambitious for a person who has never opened a CAD program
and is being asked to reach a machine program. Nothing measured produced it. If
it turns out to be the wrong number, the honest outcome is a recorded change to
this document with the observed distribution behind it, and not a quiet slide to
whatever was achieved.

### 2. Vocabulary consistency

The numbers. Zero terms used in this project's own strings that are absent from
the vocabulary in #63. Zero concepts in that vocabulary carrying more than one
term.

The units. Two counts, both with a target of zero.

The procedure. Mechanical, and no observer is involved. #95 checks the
documentation half against the vocabulary and reds on a term that is not in it.
The messages half is checked against the externalised string set from #118
rather than against the code, which is #63's own condition and matters because a
string sitting in source that no interface shows is not a term a person meets.
Terms the suite fixes and this project cannot change are in the vocabulary
marked as fixed, and they count as compliant, because #63's rule is that this
project uses them unchanged rather than inventing a better synonym.

Why zero rather than a proportion. A second word for one thing is the disease
being treated here, so a bar that tolerates a few of them is measuring something
else.

### 3. Interruption cost

The number. Three interactions or fewer to get back to the state before the
operation that failed, for every failure reachable on the committed path.

The unit. A count of interactions, taken as the worst case across the failure
list rather than as a median.

The procedure. #65 delivers the recovery and produces the list of failures. For
each one, a scripted run through the headless harness in #66 triggers the
failure and counts the interactions needed to reach the previous state, which
M3's restore in #43 is what makes reachable at all. No observer is needed and
the number is re-runnable in the gate.

Why worst case. A median here is satisfied by a lot of cheap recoveries beside
one that ends the session, and the one that ends the session is the whole
subject. This is where the suite loses beginners today.

### 4. Message quality

The numbers. At least ninety per cent of the failures reachable on the committed
path carry a message that names the task the person was doing. Exactly zero
messages assert a cause the code has not established.

The units. A proportion over the failure list from #64, and a count.

The procedure. #64's list of reachable failures is the denominator, and #64's
second condition already requires the record to state how the no-guessing check
was made. This element reads that record rather than repeating the work, and
where the record does not state it, the element is reported as not measured
rather than as passing.

Why ninety and not one hundred. The denominator is the set of failures somebody
has actually reached. A failure nobody has reached yet cannot have had a message
written for it, so a hundred per cent over a list known to be incomplete would be
a claim about the list rather than about the messages. The zero for asserted
causes is not softened in the same way, because a message that names a cause
which is not the cause is worse than one that admits it does not know.

### 5. Starting state

The number. One hundred per cent, on every launch, on every platform #11 covers.

The unit. A binary match between the applied state and the definition file,
counted over launches from a clean profile.

The procedure. #62's own test, run without a display. That test is already
required to fail when the definition and the applied state disagree rather than
only when the code errors, so this element is that test's result and adds
nothing to it. If #62's test is weakened, this element is weakened with it, and
that is worth seeing rather than discovering.

## What invalidates an observation

Element 1 is the only one with observers, so this section is about it. An
observation does not count if any of these is true.

The person has used this project or the underlying suite before.

Anybody answered a question during the run, including the observer.

The build under observation changed during the run.

The run ended for a reason outside the product, such as the machine failing or
the person having to leave.

The task was given in words that are not in the vocabulary from #63, or was
elaborated on out loud.

The person knew what the number was going to be used for.

The rule that makes the list work is the last one, and it is not on the list. An
observation is invalidated before its number is read, and the reason is written
down at that moment. An observation set aside after the number is known is not
an invalid observation. It is a number this project got and did not like, and it
is reported. #67 runs this rather than improvising when the first number is
disappointing.

## Who produces each number, and how many observations stand behind it

Elements 2 and 5 are produced by commands in the gate and anybody can run them.
The number of observations is every run of the gate, so there is no sampling
question.

Element 3 is produced by a scripted run over #65's failure list. The number of
observations is the length of that list rather than a sample of it, which is why
the target is a worst case rather than a median.

Element 4 is a proportion over a list somebody maintains, so it is produced by
whoever holds #64 and it is checkable by anybody re-running the count. The
observation count is the whole list.

Element 1 needs a person who has used neither this project nor the suite, and
each such person can produce exactly one observation, ever. Ten are needed. That
makes it the only scarce number on this bar, and it is the reason the invalidation
rules above are written before the first observation rather than after a
disappointing one.

## What is deliberately not measured

Whether the interface looks good. There is no unit for it, and a number invented
for it would be a number about whoever chose the scale.

Whether the product feels modern or professional. The same problem, and the word
would end up meaning whatever the last person to use it wanted it to mean.

A comparison against the incumbent's onboarding. This project cannot run that
product under the same conditions, so any comparison would be assembled from
other people's accounts and presented as a measurement. It would also be a
marketing artefact rather than a design instrument.

How fast an experienced user is. The committed path is not built for them, #68
is, and a bar that rewarded expert speed would pull the path away from the
person it exists for.

Anything done off the committed path. #68's fifth condition already requires #67
to record which measurements do not apply out there, and this document does not
set a target for work it does not guide.

Whether the produced program cuts a good part. That is #75 and #76, and it is a
safety question rather than a coherence one. Putting it in this bar would let a
coherence score stand in for a safety proof, which is the specific confusion
worth refusing by name.

## What this bar does not settle

It does not decide whether the workflow is coherent. It states what would have
to be true before that claim is made, and #67 reports what was found. Every
element passing and the product still being hard to use is a possible outcome,
and it would be information about this document rather than about the product.
The response to it is a recorded change here, with the reason, under the same
rule as a missed target.
