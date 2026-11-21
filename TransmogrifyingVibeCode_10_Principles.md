# Chapter 10: The Principles

*Twelve principles for the AI-coding era, distilled from 13 months of practice, 6 incarnations, 768 commits, and one comprehensive reckoning with architectural debt.*

[< Previous: The Method](TransmogrifyingVibeCode_09_Method.md) | [Next: For the Vibe Coder With a Pile of Spaghetti >](TransmogrifyingVibeCode_11_Recipe.md)

---

## 1. The Surface Is the Product

Organize the surface first. Users don't see architecture. They see buttons, charts, transitions, responsiveness. If the surface works, the product works. If the surface doesn't work, nothing else matters.

This is not an argument for sloppy code. It's an argument for sequencing. Build what users touch. Understand what you're building by building the parts that are visible. Then fix the infrastructure — because now you know what the infrastructure needs to support.

PlanStan's Gantt chart, dependency graph, docking system, theming, and calendar views were all built before the library decomposition. The surface drove the architecture, not the reverse. The architecture that emerged from extraction was *better* than any architecture designed in advance would have been, because it was informed by the full reality of what the surface required.

You don't organize a kitchen before you know what meals you'll cook.

## 2. Vision Drives the Machine

The demanding personality is the quality filter. Without it, the LLM produces the path of least resistance — generic, over-abstracted, technically correct, spiritually empty.

> "Do not begin offering me code yet, we are still getting aligned."

The developer who knows what they want — who can specify at the right level of detail, who can reject output that doesn't meet the standard, who can say "still redundant!" and mean it — produces fundamentally different software than the developer who accepts whatever the model generates.

Vision is the scarce resource. LLM output is abundant. The ratio of vision to output determines the quality of the product. More vision, less acceptance of default output, better results. This is leadership, applied to a statistical process instead of a team.

## 3. Prose Is Code Now

Structured, opinionated natural language produces software. This is not a metaphor. It is a literal description of what happens when you type a specification into an LLM prompt and receive working code in response.

The developer learned to program through LLM conversation — not "learned to use LLMs to program," but learned to program, period. The medium of instruction was prose. The output was C++20. The translation happened inside the model, but the specification — the thing that determined what the code would do — was English.

The implications are enormous. Programming is no longer gatekept by syntax knowledge. The gate is now taste, vision, and the ability to specify clearly. These are human skills, and they're distributed more widely than C++ expertise.

> "An LLM is like a bell; it is a percussion instrument to be struck at a certain place, a certain way, and in response it lets out a single resonant chime in return."

The quality of the chime depends on the quality of the strike.

## 4. Extract the Good; Don't Fix the Bad

Your first instinct when confronting a messy codebase will be to attack the worst code. Resist. The worst code is the worst code because it's entangled. You can't fix it without understanding its dependencies, and you can't understand its dependencies because they're implicit and scattered.

Instead, extract the code that's *already* good. The pure algorithms, the data types, the self-contained subsystems. As you extract, the entanglements become explicit (you have to create interfaces for them) and the bad code concentrates (it's what's left).

11 libraries extracted. The god class untouched during extraction. The dead code untouched. The redundant APIs untouched. Only after everything clean was removed did the mess become small enough, visible enough, and funny enough to fix.

## 5. Grep Is Your Architect

You don't need to design interfaces from first principles. You need to know what methods are actually called. `grep` tells you. The interface is the set of methods that appear in your grep results. Nothing more.

Don't anticipate future needs. Don't add methods "for completeness." Don't design a taxonomy. Extract exactly what exists. `ICommandDispatcher` has 2 methods because the grep found 2 methods. `ISyncHost` has 11 methods because the grep found 11 methods. Neither was designed. Both were discovered.

This is liberating. You don't need to be a software architect. You need to be a software *archaeologist*. The architecture is already in the code — you're excavating it, not inventing it.

## 6. Each Library Must Build Alone

The test for whether an extraction is complete: can this library build as a standalone project with only its declared dependencies? If yes, the extraction is real. If no, you have hidden coupling.

This is a bright-line rule. No exceptions. No "it works in-tree so it's fine." Standalone build capability is the proof that the separation is genuine. CMake's `PROJECT_IS_TOP_LEVEL` provides the mechanism. The discipline is yours.

A library that can't build alone is not a library. It's a directory.

## 7. Interfaces Should Be Narrow and Discovered, Not Designed

Every interface in PlanStan has fewer methods than you'd expect. `ICommandDispatcher` has 2. `IEditorServices` has 6. `IViewServices` has 6. `ILocaleSource` has 5. Even `ISyncHost`, the broadest, has 11 — out of a god class with 66.

These interfaces weren't designed. They were discovered by auditing actual usage. A narrow interface is easy to implement, easy to test, and hard to abuse. A wide interface is just a god class wearing a mask.

If your interface has more than 10-12 methods, ask whether you're creating a genuine abstraction or just renaming the coupling. If a single consumer needs 31 methods (as `CollectionView` needed from `CollectionController`), don't create an interface — move the consumer to the shell where the dependency is honest.

## 8. Dead Code Reveals Itself Through Concentration

When code is spread across 582 files, dead code is invisible. It hides in the noise. A dead method in a 2,500-line class is a needle in a haystack. You don't notice it. You maintain it across refactors. You compile it every build. You might even test it.

When the same code is concentrated in 8 files — after the good code has been extracted — dead code is obvious. The 216-line method that nobody calls. The hash map that stores pointers it never dereferences. The session-state subsystem that was superseded and never deleted.

The act of extraction doesn't create dead code. It *exposes* dead code that was always there, hidden by the noise of a large codebase. Extraction is a diagnostic tool as much as an organizational tool.

## 9. File I/O Forces Good Architecture

Even in vibe code, the I/O subsystem is probably okay. Networks punish sloppiness immediately. Corrupt data is visible immediately. Round-trip failures are testable immediately.

The rest of your codebase may tolerate architectural sins indefinitely — the god class works, the copy-paste works, the timers usually win the race. The I/O code can't tolerate them, because the data either survives or it doesn't.

Start your architectural assessment at the I/O layer. Not because it needs the most work — because it's the foundation you can build on.

## 10. Don't Rewrite — Reveal

The application you vibe-coded works. It has features, users, and accumulated domain knowledge encoded in its behavior. A rewrite destroys all of that. An extraction preserves all of that while giving it a structure it can grow into.

The structure was always there. The scheduling algorithm was always independent of the UI. The tag system was always independent of the sync engine. The org-mode parser was always independent of the calendar views. They were just imprisoned in a monolith, unable to express their independence.

Extraction reveals what rewriting destroys.

## 11. Starting Over Is Cheap Now

Six incarnations in 13 months. Each one 3-9 days of work. Each one better than the last.

In the pre-LLM era, starting over was a catastrophic cost: months of work discarded, morale destroyed, stakeholders betrayed. In the LLM era, starting over is a strategic option: carry forward the specification, not the code. Write a baton document. Start a fresh conversation. Build the next version on better foundations in days instead of weeks.

The tumbleweed pattern — grow, break free, scatter seeds, root in better soil — is native to LLM-assisted development. When starting over costs days rather than months, the optimal strategy changes. You can afford to experiment. You can afford to be wrong. You can afford to build something, learn from it, and build something better.

This doesn't mean you should always start over. PlanStan's final incarnation (768 commits, 13 months) was worth extracting rather than rewriting. But the early incarnations (3-9 days each) were worth abandoning. The judgment of when to extract and when to restart is one of the few skills that LLMs cannot provide for you.

## 12. Imposter Syndrome Is the Wrong Frame

If the code works, you're not an imposter. If the Gantt chart renders, if the sync engine syncs, if the scheduler schedules, if the 11 libraries build independently — you built that.

You built it with new tools. The tools don't diminish the achievement. A photographer isn't an imposter because they used a camera instead of painting. An architect isn't an imposter because they used CAD instead of drafting by hand. A developer isn't an imposter because they used LLMs instead of typing every semicolon.

The imposter narrative says: "a real developer would have written clean architecture from the start." This is false. Real developers produce god classes, dead code, and redundant APIs every day. The difference between an amateur and a professional is not the absence of mess — it's the presence of a cleanup cycle.

You built something. Now you're cleaning it up. That's not imposture. That's the full lifecycle of software development.

---

## The Principles as a System

These principles aren't independent. They interlock:

- **Surface first** (1) tells you what to build. **Vision** (2) tells you what to demand.
- **Prose as code** (3) is the mechanism. **The bell metaphor** is the skill.
- **Extract the good** (4) is the strategy. **Grep** (5) is the tactic. **Build alone** (6) is the test.
- **Narrow interfaces** (7) are the coupling discipline. **Dead code concentration** (8) is the diagnostic.
- **I/O forces quality** (9) tells you where to start trusting your code. **Don't rewrite** (10) tells you to preserve what works.
- **Starting over is cheap** (11) gives you courage. **Imposter syndrome is wrong** (12) gives you permission.

Together, they describe a complete lifecycle for LLM-assisted software development: build with vision, build the surface first, build fast, accumulate debt, recognize the debt, extract the good code, discover the interfaces, concentrate the mess, clean the mess, and never apologize for using the tools that exist.

The tools are new. The craft is ancient. The principles bridge them.

---

[< Previous: The Method](TransmogrifyingVibeCode_09_Method.md) | [Next: For the Vibe Coder With a Pile of Spaghetti >](TransmogrifyingVibeCode_11_Recipe.md)
