# Chapter 5: The Art of Vibe Coding

*What vibe coding is, what it isn't, and why the bell metaphor matters. A year into the practice, with observations for fellow practitioners who think they're done when the features work.*

[< Previous: The Tumbleweed](TransmogrifyingVibeCode_04_Tumbleweed.md) | [Next: The Surface >](TransmogrifyingVibeCode_06_Surface.md)

---

## A Year Old and Already Ancient

February 2026. Vibe coding, as a named practice, is about a year old. Andrej Karpathy coined the term in February 2025, describing a workflow where you "fully give in to the vibes, embrace exponentials, and forget that the code even exists."

The developer behind PlanStan started vibe coding two weeks after the term was coined. That makes them an old-timer in a field where most practitioners are still on their first project. They've been through four LLM platforms (Grok, Claude Sonnet, Claude Opus, and the transitions between them). They've conducted 175+ documented coding sessions through VibeKoder's archived transcripts. They've built a custom tool for managing the process itself. They've produced 140K lines, decomposed them into 11 libraries, and documented the archaeology of what vibe coding actually produces when you're not looking.

This chapter is the practitioner's report. Not a tutorial — the internet has plenty of those, and they're mostly wrong about what matters. A field report from someone who has done this long enough to see the patterns, the failure modes, and the genuine art buried in the practice.

---

## What Vibe Coding Actually Is

Karpathy's original description emphasizes abandonment: give in, forget the code exists, accept every suggestion. This is how most people practice vibe coding, and it's the source of almost all of its problems.

But there's an art underneath the chaos. The VibeKoder founding document captures it:

> "Vibe coding is very organic compared to the structured development of regular code. Traditional coding takes time to be typed and tested. LLMs, by contrast, produce ample code; too much code, a veritable stream of endless code on tap."

> "We might, then, liken vibe coding to gardening: it is the cultivation of blooms of code. The vibe coder plants seeds, and gathers and shapes and prunes the excess of living, quasi-biological material which springs forth."

The gardening metaphor is more honest than the "give in to the vibes" framing. A gardener doesn't control what grows. They don't write the DNA of the tomato plant. But they choose what to plant, where to plant it, how to prune it, what to support with stakes, and what to pull out as a weed. The garden is not the gardener's creation — it's a collaboration between human intention and biological process.

Vibe coding is the same collaboration between human intention and statistical process. The LLM produces code the way a garden produces growth: abundantly, directionally, and without judgment about whether the result is good. The vibe coder's job is not to write the code — it's to evaluate, select, shape, and prune what the LLM produces.

This requires taste. Not technical knowledge (though that helps). Taste: the ability to look at output and say "that's wrong" or "that's ugly" or "that solves the wrong problem" even when you can't articulate precisely why. The developer behind PlanStan had taste in abundance. They could read code well enough to spot garbage, even when they couldn't write the correct version themselves. And they demanded better:

> "Hahaha. No no no. You're still moving a ton of string-writing into the derivative classes which is identical between them. Still redundant!"

That's taste in action. The developer couldn't necessarily refactor the string-writing themselves. But they could see the redundancy, and they refused to accept it. The LLM tried again. And again. And eventually produced something that met the standard.

---

## The Bell Metaphor

The developer's theory of LLM interaction goes deeper than gardening. It goes to the nature of the instrument:

> "An LLM is like a bell; it is a percussion instrument to be struck at a certain place, a certain way, and in response it lets out a single resonant chime in return. Yes, feeding that resultant chime back into the LLM again for a second, third, or fourth strike can continue to lead to useful returns, but eventually the ouroboros shrivels."

This is the most important sentence in the entire VibeKoder corpus, and it contradicts the default mode of LLM interaction that billions of dollars of chatbot UI design has trained people to expect.

The chat interface implies conversation: back and forth, building context, evolving understanding. The bell metaphor says: no. Each interaction is a single strike. The quality of the response depends on the quality of the strike — the precision of the prompt, the completeness of the context, the clarity of the ask. Feeding the response back for another strike (continuing the conversation) can work, but it degrades. The ouroboros — the conversation consuming its own context — shrivels.

The practical implications:

**Short sessions.** The developer learned through painful experience that long LLM conversations degrade. The Grok transcripts from March 15 run to 155,000 words — and the later portions are visibly deteriorated, with the model contradicting itself, losing architectural coherence, and generating code that conflicts with its own earlier output. The "Last Testament of a schized out Grok" document records this degradation with clinical precision.

**Fresh strikes.** Each new task should start with a fresh prompt, not a continuation. The prompt should include the current state of the code (or the relevant portion), the development documentation, and the specific task — not the entire history of the conversation. The developer built a system for this: baton documents that captured the state without the conversation.

**Optimized blows.** The quality of the prompt matters more than the number of iterations. A well-structured prompt with clear context, explicit constraints ("don't touch the sync code"), and a specific ask produces better output on the first strike than a vague prompt produces after five rounds of iteration.

> "By getting away from the ubiquitous chat interface, and offering an interface to re-configure and minutely adjust each 'first' strike as fresh, the resonances from our bell can be played with, heard, tested, recorded, compared, and used freely in parallel."

> "The onus for 'playing' the LLM, then, moves back onto the coder."

This last line is the key. Vibe coding, practiced as art rather than as vibes, puts the responsibility squarely on the human. The LLM is an instrument. The human is the musician. The quality of the music depends on the musician's skill, not the instrument's capabilities.

---

## The Speed

One dimension of vibe coding that deserves honest acknowledgment: the speed is extraordinary.

The developer mined decades of software engineering knowledge — CalDAV protocols, iCalendar specifications, Qt widget frameworks, graph layout algorithms, project management theory, GTD methodology — to produce features in days that would take professional teams weeks.

Consider February 2, 2026. Sixty commits. In a single day, the developer produced:

- A custom Gantt widget (replacing the KGantt dependency entirely, with bar rendering, arrow routing, axis mapping, and a header widget)
- An Akonadi backend (integrating with KDE's personal information management framework)
- A multi-month tiling view
- A temporal scroll stack

Each of these is a substantial feature. Any one of them might be a week's work for an experienced developer working alone. The LLM compressed the implementation time by handling the boilerplate, the API lookup, the pattern application — all the mechanical work that takes time but doesn't require architectural judgment. The developer provided the judgment: what to build, how it should behave, what the constraints are. The LLM provided the labor.

This compression is real, it's dramatic, and it's available to anyone with the taste and vision to direct it. You don't need to know the KGantt API by heart. You need to know what a Gantt chart should look like, how it should behave, and what data it needs to display. The LLM handles the rest. What used to require "five years of Qt experience" now requires "a clear specification and the ability to evaluate output."

But — and this is the "but" that this entire chronicle exists to document — the speed comes at a cost. The code that's produced in 60-commit days is locally correct and globally uncoordinated. Each feature works. The features don't share a consistent architecture. The abstractions don't align. The naming conventions drift. The dead code accumulates because nobody has time, at 60 commits per day, to delete the code that the last commit superseded.

The speed is the achievement. The mess is the tax on the achievement. Both are real.

---

## Four Platforms, One Workflow

The developer worked with Grok, Claude (multiple versions), Sonnet, and Opus over the course of 13 months. Each platform had different strengths:

**Grok** (February-March 2025): The origin platform. Long context windows, high tolerance for the developer's demanding style, good at Qt/KDE code. The transcripts show a model that could sustain detailed architectural discussion for thousands of words. The degradation, when it came, was gradual and documented — the "schized out Grok" phenomenon.

**Claude** (April 2025 onward): The transition platform. The "Team B" prompt — 858 lines of specification — was written as a handoff from Grok to Claude. The developer's observation: Claude was more disciplined about following architectural constraints but less willing to generate large code blocks without being explicitly asked. The demanding boss style worked, but required adjustment.

**Later Claude models** (January 2026 onward): The sprint platform. The January-February burst — hundreds of commits over weeks — happened with models that could handle broader context, longer sessions, and more complex multi-file changes. The library decomposition itself was conducted with Claude, including the forensic analysis and interface extraction.

The workflow remained consistent across all platforms:

1. **Start with context.** Current code (amalgamated or relevant files), development documentation, architectural constraints.
2. **Specify the task.** Clear, specific, bounded. "Add a minimap to the graph view." Not "improve the graph."
3. **Set constraints.** "Don't touch the sync code." "Use the existing ViewInterface pattern." "This must build standalone."
4. **Evaluate output.** Read it. Run it. Look at it with taste. Does it meet the standard?
5. **Iterate or accept.** If the output is wrong, explain why and ask again. If it's right, integrate it.
6. **Document.** Update baton documents, development docs, commit with a clear message. The next session starts clean.

This workflow is platform-agnostic. It works with any LLM that can generate code. The platform doesn't matter. The workflow does.

---

## For Fellow Vibe Coders: The Gentle Warning

If you're reading this as a fellow vibe coder — someone who has built something with LLMs that works, that you're proud of, that does what you wanted it to do — congratulations. You've done something remarkable. A year ago, you probably couldn't have imagined building what you've built. The tools are real, the output is real, and your achievement is real.

Now: have you looked under the hood?

Not at the features. Not at the UI. Under the hood. At the architecture. At the class that started as a controller and became a god. At the method that started at 30 lines and is now 305. At the three different ways you can do the same thing because each one was added in a different coding session and nobody deleted the previous versions. At the dead code that's maintained across refactors because you're not sure it's dead. At the `setTimeout(500)` — or `QTimer::singleShot(500)` — that papers over a timing bug you never fixed.

You see the surface and think you're done. You are not done.

This isn't a criticism. It's a stage in the lifecycle. The surface is the product (Chapter 6 makes the case for why getting it right first is correct). But the architecture is the foundation, and if you want to keep building — if you want to add the next feature, and the one after that, without each addition becoming harder than the last — you need to look under the hood.

The good news: what you'll find there is probably hilarious. Dead methods that nobody calls. Copy-pasted logic that's been maintained in three places. A factory method that repeats the same seven arguments twelve times. A memory leak masquerading as a hash map. An entire dead subsystem that nobody told.

The better news: the fix is mechanical. You don't need to be a senior architect. You need `grep` and the willingness to extract, concentrate, and clean. Chapter 9 is the method. Chapter 11 is the recipe.

The best news: the mess doesn't diminish the achievement. Every professional codebase has this mess. The difference between amateur code and professional code isn't the absence of mess — it's the presence of a cleanup cycle. You're about to start yours.

---

## The Art, Honestly

Vibe coding, practiced well, is a genuine art form. It requires:

- **Vision:** Knowing what you want to build before you start typing
- **Taste:** Evaluating output against a standard that the LLM can't set for itself
- **Discipline:** Setting constraints, enforcing boundaries, refusing to accept "good enough"
- **Architecture awareness:** Understanding, even intuitively, when coupling is increasing and when concerns are blurring
- **Emotional regulation:** Not giving up when the LLM produces garbage for the third time, not accepting garbage out of fatigue, not losing the vision in the details

The developer behind PlanStan demonstrated all of these. They had the vision (an IDE for time management), the taste (refusing redundant code, demanding clean abstractions), the discipline (batons, documentation, constraint-setting), the architectural awareness (recognizing design problems versus implementation problems), and the emotional regulation (continuing to demand better at 3 AM after 1,500 compiler errors).

The art of vibe coding is not in the vibes. It's in the demands. It's in the taste. It's in the refusal to accept the LLM's default temperature when you know what you want is sharper, cleaner, more specific than what the model is inclined to produce.

And it's in the humility to look at what you've built, see the mess beneath the surface, and say: this is good, and it can be better. The code works. Now let's make it clean.

---

[< Previous: The Tumbleweed](TransmogrifyingVibeCode_04_Tumbleweed.md) | [Next: The Surface >](TransmogrifyingVibeCode_06_Surface.md)
