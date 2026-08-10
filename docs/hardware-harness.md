# The hardware harness, and the rule for what may be added

Some things cannot be proven without hardware. A machine that cuts, a device
with a driver, a graphics stack that behaves differently from a software
renderer. Those tests are real, and #32 keeps them out of the suite that runs in
the ordinary gate.

The reason is not tidiness. A mixed suite reports one number, and that number is
read as covering everything in it, while the hardware tests skipped on every
machine that ran it. That is a run reporting a pass it did not earn.

This document is the fourth of #32's conditions. It says what belongs in the
harness, and it states the rule for adding something so that the harness does
not become the place hard tests go.

## Nothing here has been built

There is no harness, no test and no code of any kind. Every file tracked on the
mainline is a document, a workflow, a licence or a list:

    gh api "repos/iderex/reissbrett/git/trees/main?recursive=1" --jq '[.tree[] | select(.type=="blob") | .path | split(".") | last] | unique | join(" ")'
    DCO LICENSE gitattributes md txt yml

So the list below is what the harness is for rather than what is in it. It is
written before the harness exists on purpose, because a rule about what may be
added is worth least on the day somebody wants to add something.

## What belongs in it

Three classes, each with the issue that puts it there.

The machine harness from #78. A run that drives a real controller, cuts real
material, or measures a real result. This is the class the separation exists for
and the one that can hurt somebody.

Anything that needs a real graphics driver rather than a software renderer. Not
anything that draws, which is #66's problem and is solved by separating the
decision from the drawing. This class is the narrower thing: a test whose
subject is what the driver itself does.

Anything that needs a device this project does not own. A probe, a controller, a
dongle, a camera. The distinguishing property is that the test cannot be made to
pass on a machine that has only a processor, a disk and a network.

## The rule for adding a test

One question decides it, and it is asked in the pull request that adds the test:

**Is there a way to run this on a machine with no display, no device and no
elevation?**

If there is, the test does not go here, and the effort of writing it that way is
the work rather than an argument against it. If there is not, the pull request
says which of the three classes above it falls in and why, and that sentence is
what the review reads.

The rule refuses these by name, because each is a way of answering the question
with something that is not an answer:

- It is slow headless. Slowness is a cost, not an impossibility.
- It would need a fake, a stub or a recorded fixture to run headless, and
  writing that is work. That is the work.
- It is intermittent headless. An intermittent test is a defect in the test, and
  moving it here hides the defect behind a harness nobody runs.
- It needs elevation. Elevation is not hardware. #31 makes unelevated a property
  the run asserts about itself rather than a habit, and a test that requires
  elevation does not become admissible by being moved here.
- The hardware is available on the machine where it was written. What one
  machine has is not a property of the test.

The last of the five is the one that will be argued, so it is worth being exact.
A test may use hardware that happens to be present without needing it. If it
passes on a machine without that hardware, it belongs in the ordinary suite and
the hardware is an accident of where it ran.

## What this rule does not decide

It does not say what the harness is called, which is #32's second condition and
a decision about the name rather than about the contents.

It does not say what the ordinary suite prints when this harness was not run,
which is #32's third condition, or what the harness does when it is run with no
hardware present, which is its fifth. Both are behaviour and neither exists.

It does not decide anything about #78's own gating. #78 states what a recorded
run has to carry, and this document only says that #78's runs live here.

## What refuses this, and what does not

PROSE, NOT ENFORCEMENT for the rule above, and the two halves are different
cases.

The half about invocation is decidable. #32's first condition is that the
harness has its own command and that no pull request check calls it, and that is
a property of workflow files and a name, which is the shape `.github/invariants.txt`
already holds records for. No record there covers it today, because there is no
harness and no command to name.

The half this document is mostly about is not decidable by anything. Whether a
test could have been written headless is a claim about a version of the test
that was not written, and there is nothing in this tree, or in any tree, for a
check to read. That makes the mark terminal here rather than a placeholder for a
mechanism that is coming.

No issue on this board owes one either, and that is stated rather than left to
be assumed. #98 is where the architecture rules become tests, and it enumerates
the rules it takes; this one is not among them and neither is this harness:

    gh issue view 98 --repo iderex/reissbrett --json body --jq '.body' | grep -ci 'hardware'
    0

So what holds this rule is the review and the sentence the pull request is asked
to carry. That is weaker than a check and it is said here rather than discovered
by somebody who assumed otherwise.
