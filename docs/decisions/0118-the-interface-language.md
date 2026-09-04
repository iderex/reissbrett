# The interface language, and keeping a translation possible

#63 fixes one vocabulary for the committed path and says it is English. That
settles what this project writes. It leaves the hole underneath, because the
suite this project sits on is translated, reads its own interface language from
a setting, and defaults that setting to whatever the operating system says. So
a beginner can be following a guided path whose steps are named in one language
while every menu the steps point at is in another, which is the incoherence #63
exists to treat arriving from underneath rather than from this project's text.

This document decides the interface language of the first release, what happens
when the suite underneath is set to something else, and whether the text is
externalised from the first line of code. It does not re-open #63.

Nothing in the tree had decided any of that. Three landed documents defer to
this issue by number and none of them answers it. Read at the commit this
branch came off rather than at a moving reference, because this file matches
the same pattern once it lands:

    git grep -n '#118' c69876e99119cdc080a9fce3b0984ac5889214ec -- docs/ README.md .github/
    c69876e99119cdc080a9fce3b0984ac5889214ec:docs/decisions/0004-where-the-layer-attaches.md:97:The translation catalogues, which #118 has to decide against rather than around:
    c69876e99119cdc080a9fce3b0984ac5889214ec:docs/decisions/0009-the-committed-beginner-workflow.md:268:The interface language, which is #118. The path is a sequence of stages and it
    c69876e99119cdc080a9fce3b0984ac5889214ec:docs/decisions/0009-the-committed-beginner-workflow.md:270:is checked against #118's string set rather than against this document.
    c69876e99119cdc080a9fce3b0984ac5889214ec:docs/decisions/0013-the-bar-for-a-coherent-workflow.md:83:The messages half is checked against the externalised string set from #118

The third of those is the one that constrains this decision rather than only
waiting on it: the bar in #13 measures the messages half against an
externalised string set, which is a shape rather than a language, and the
decision below owes it.

## The version everything below is read at

The suite is pinned at 1.1.3 by
`docs/decisions/0005-the-pinned-suite-version.md`, and every upstream command
in this document carries that tag. The commit this was written against is the
one above.

## What the suite underneath already does

The suite offers its interface in forty-five languages, listed in one map in
its translator:

    gh api repos/FreeCAD/FreeCAD/contents/src/Gui/Language/Translator.cpp?ref=1.1.3 --jq '.content' | base64 -d | grep -c 'mapLanguageTopLevelDomain\[QT_TR_NOOP'
    45

and it carries a catalogue per language beside that file:

    gh api repos/FreeCAD/FreeCAD/contents/src/Gui/Language?ref=1.1.3 --jq '[.[].name | select(endswith(".ts"))] | length'
    47

The two numbers are not the same thing and the difference is not a defect. The
map is what the interface offers a person. The catalogues include entries the
map does not name, and nothing here reconciled them, because the decision below
turns on the map rather than on the file count.

Which language is used is read from one string, and the default when nobody has
set it is the operating system's language:

    gh api repos/FreeCAD/FreeCAD/contents/src/Gui/Application.cpp?ref=1.1.3 --jq '.content' | base64 -d | sed -n '479,484p'
        // install the last active language
        ParameterGrp::handle hPGrp = App::GetApplication().GetUserParameter().GetGroup("BaseApp");
        hPGrp = hPGrp->GetGroup("Preferences")->GetGroup("General");
        QString lang = QLocale::languageToString(QLocale().language());
        Translator::instance()->activateLanguage(
            hPGrp->GetASCII("Language", (const char*)lang.toLatin1()).c_str());

That is the fact the whole decision rests on. A machinist in a German workshop
who has never opened a preferences dialog is running the suite in German,
because their machine is German. Nothing was configured wrongly and nobody made
a mistake.

The group that string lives in is `User parameter:BaseApp/Preferences/General`,
which is surface 5 in `docs/extension-surface.md`, the same surface #62 applies
the rest of the known starting state through. So this project can read the
setting and can write it, and that is what makes the choice below a choice
rather than an acceptance.

## The decision

**The first release's interface is English, and there is exactly one
language.** No second language ships, no translation route is set up, and
nothing here promises one to anybody.

**The known starting state sets the suite underneath to English, and the guided
path does not run against a language it did not set.** The value written is the
`Language` key in the group above, and the value is the map's own name for it,
`English`:

    gh api repos/FreeCAD/FreeCAD/contents/src/Gui/Language/Translator.cpp?ref=1.1.3 --jq '.content' | base64 -d | sed -n '197p'
        d->mapLanguageTopLevelDomain[QT_TR_NOOP("English"               )] = "en";

**The person is told that this happened, at the point it happens, and told how
to put it back.** Changing somebody's language without saying so is the silent
option this issue already refuses, and writing the setting instead of reading
it does not make the silence acceptable. What the person is told, in the
vocabulary from #63, is that the guided path is in English, that the suite was
switched to English so the two agree, which setting was changed, and that
changing it back leaves the path naming commands the menus do not use.

**Interface text is externalised from the first line of code that has any.**
Not one string in a source file. This is the part of the decision that costs
nothing now and cannot be bought later at any price worth paying, because text
held in code is a translation nobody can contribute without a rewrite of the
code around it.

## Why the starting state sets it rather than refusing to start

The issue names three answers and rules out one of them by itself. What is left
is refusing to run the guided path when the suite is in another language, and
setting the language as part of the state the path starts from.

Refusing turns a preference into a wall, and it puts the wall in front of
exactly the person this project exists for. A beginner who is running the suite
in their own language has done nothing wrong, and the first thing this software
would say to them is that they may not proceed until they find a preferences
dialog they have never opened and change a setting they did not know existed.
The first stage of the committed path in #9 is where that person is, and #13
measures whether they got through it.

Setting it makes the software work on the machine it was installed on. The cost
lands in one place, it is one setting, and it is reversible by the person in
the dialog they would otherwise have been sent to. The bar in #13 is measured
on the committed path, and a path measured against menus in a second language
would be measuring the mismatch rather than the workflow.

There is a real objection to this and it is not answered by the paragraph
above. The `Language` key is the installation's setting and not the guided
path's, so writing it changes the suite everywhere, including for work that has
nothing to do with this project and including after the person has left the
path by way of #68. This decision accepts that and refuses to hide it. The
disclosure is the third part of the decision rather than a courtesy attached to
it, and an implementation that writes the key without telling the person has
not delivered this decision.

## What it costs the people it excludes

Somebody who does not read English cannot use the committed path. That is the
whole cost, it is not softened by anything below, and it is larger here than it
would be in most projects, because the thing this project sits on does not have
that limit. Forty-five languages upstream and one here is the shape of it.

The loss is concentrated in the population this project is aimed at. A workshop
is not an office, the person at the machine is not required to have worked in
English, and the German-speaking small manufacturer this project has in view is
precisely somebody for whom an English interface is a barrier rather than a
detail.

Two things that are not offered as consolation and are stated because they are
true. The suite underneath stays translated and a person who leaves the guided
path by way of #68 can set it back to their own language, at the price named
above. And nothing in the decision to ship one language makes a second one
harder later, which is what the externalisation part of the decision is for.

What is not claimed: that English is a reasonable requirement, that a machinist
should learn it, or that this will be fixed. The first release ships one
language because this project cannot carry two, and that is a statement about
this project rather than about anybody who is shut out by it.

## The alternatives, and why each was rejected

**Ship German first, or German and English together.** Two languages from the
first release costs a translation route, a review route for translated text,
and a vocabulary that has to be correct in two places at once, before the
workflow the vocabulary describes has been proven to work at all. #63 also
already fixes the vocabulary as English, and this document applies that rather
than reopening it.

**Follow the operator's system locale and translate as far as the suite is
translated.** This sounds like the generous answer and it produces the worst
outcome on this board. The guided path would be partly translated, the suite
underneath fully translated, and the vocabulary in #63 would be a set of
English terms with no relationship to the words on screen. A half-translated
coherent workflow is not a coherent workflow.

**Leave the suite's language alone and say nothing.** Refused by #118 itself,
and correctly. It is the option where the software works for the person who
wrote it and is incoherent for everybody else, with nothing in the product
admitting it.

**Leave the suite's language alone and say so plainly.** The honest version of
the option above, and the closest rejected alternative. It is rejected because
the disclosure it offers is an apology rather than a fix: the person is told
that the menus will not match the instructions, and then has to work through a
path whose every step names a command they cannot find. The measurement in #67
would have to be run in that condition to know what it costs, and running it is
more expensive than setting one string.

**Refuse to start the guided path unless the suite is already in English.** The
argument against it is the section above.

**Translate the guided path and leave the interface language to the suite.**
This is the same defect as the third alternative approached from the other
side, and it adds the cost of a translation route to it.

## What adding a language would require

Listed so that a later decision is made against the cost rather than against
enthusiasm. None of it exists, none of it is committed to, and this list is not
a plan.

A catalogue format and a build step that ships the catalogues with whatever #11
delivers, decided under the means recorded on #14.

A route for the text to arrive. Contributions here are signed off under
`./DCO`, so a translation is a contribution like any other and arrives the same
way, which means a translator needs the same setup a code contributor needs.
Nothing on this board reduces that for text.

A route for the text to be reviewed by somebody who reads the target language
and can refuse a wrong term. This is the expensive one and it is not a tooling
problem. `.github/GOVERNANCE.md` records that one account has write access here
and that there is nobody beside me, so today there is no second reader for a
second language and adding one is adding a person rather than a file.

The vocabulary in #63 would have to hold in two languages at once. Its own
condition checks the documentation and the messages against one string set, and
a second set is a second thing that can drift from the first while both stay
internally consistent.

The measurement in #67 against the bar in #13 is evidence about the language it
was run in. A second language either gets its own run or the bar covers one
language and says so.

The sample project in #69 and anything with text on it that #71 produces.

## What this imposes on other issues

#62 delivers the behaviour. The `Language` key in
`User parameter:BaseApp/Preferences/General` is part of the defined starting
state, set to `English`, applied through surface 5 of
`docs/extension-surface.md`. #62's own conditions already carry the language as
part of that state, and its third and fifth conditions are what make the
setting testable rather than asserted.

One thing #62 has to establish rather than assume, and it is the most likely
way this decision fails in practice. The read quoted above happens while the
suite is starting, so writing the key from a workbench that has already loaded
may not change the interface until the next start. Whether it does was not
observed here, because nothing in this tree runs the suite. If it does not, the
starting state has to be applied before the process that shows it, which is a
different thing from applying it from inside, and the decision above does not
change either way.

#62 or #61 also carries the disclosure. The third part of the decision above is
behaviour and not documentation, and no condition on either issue names it
today. That is a gap in those issues rather than in this document, and it is
written here so it is not discovered when somebody reads the decision and finds
only two of its three parts implemented.

#63 checks its vocabulary against the externalised string set rather than
against this document, which its last condition already says.

#64 writes messages into the same externalised set, since a message is interface
text.

#68 is where a person leaves the guided path, and the setting written by #62 is
still written when they do. Whether leaving offers to put it back is #68's to
decide and this document does not decide it.

#98 is where an architecture rule becomes a test, and the rule that no interface
string is written inline is one, if the check owed by #118's own third
condition lands there rather than beside it.

## What this does not settle

Which language is ever added, and when. Nothing above is a commitment to a
second one and no date is implied by the list of what one would require.

The catalogue format, the extraction tool, the identifier scheme for a string,
and the shape of the check that refuses an inline string. Those are decisions
for code, made under the means recorded on #14, and there is no code:

    gh api "repos/iderex/reissbrett/git/trees/main?recursive=1" --jq '[.tree[] | select(.type=="blob") | .path | split(".") | last] | unique | join(" ")'
    DCO LICENSE gitattributes md txt yml

The language of anything this project writes outside the product.
`CONTRIBUTING.md` already fixes English for tracked files and this document
does not extend or narrow that.

What the suite does with a language it cannot load, and what it falls back to.
That is behaviour rather than a file, nothing here ran the suite, and the
starting state in #62 is where an unset or unusable value is observed rather
than assumed.

It measures nothing about this project. There is no interface, no guided path
and no string, so every sentence about what this software does is a decision
and none of it is an observation.

## Revisiting this

Two things would reopen it, and neither is a change of mind.

A second reader who reads a second language, because the review route is the
part of a translation that cannot be bought with tooling.

A measurement under #67 that shows the English-only path failing the bar in #13
for people who read English, since a second language added to a path that does
not work yet multiplies a defect rather than widening an audience.

## What refuses any of this today

Nothing, and this is the artefact rather than the terminal kind of mark,
because every part of the decision is a property of code that does not exist
yet.

`PROSE, NOT ENFORCEMENT` for the starting state carrying the language. #62's
third and fifth conditions are where it becomes refusable, by a test that fails
when the definition file and the applied state disagree.

`PROSE, NOT ENFORCEMENT` for the disclosure. It is behaviour on the guided path
and no condition on #61 or #62 names it, which the section above records as the
gap it is.

`PROSE, NOT ENFORCEMENT` for externalisation. #118's own third condition owes
the check that refuses a string written inline, and until there is a language
and a file for one to be written in, there is nothing for such a check to read.
