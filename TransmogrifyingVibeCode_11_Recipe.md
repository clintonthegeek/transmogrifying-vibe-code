# Chapter 11: For the Vibe Coder With a Pile of Spaghetti

*The practical recipe. You built something that works. You should be proud. Now let's talk about what's underneath, because it's probably hilarious.*

[< Previous: The Principles](TransmogrifyingVibeCode_10_Principles.md) | [Back to Chapter 1](TransmogrifyingVibeCode_01_WhyThisMatters.md)

---

## First: Congratulations

You built something. With LLMs, with determination, with vision, with midnight sessions and stubborn refusal to accept "good enough." You built something that works. Something that does what you wanted it to do. Something that, when you demo it, makes people say "wait, you built that?"

You should be proud. The fact that you're reading this chapter — that you've arrived at the "I should probably look under the hood" stage — means you've built something worth looking under the hood of. Most vibe-coded projects never get this far. Most are abandoned at 2,000 lines when the next feature doesn't work and the developer doesn't know why. You kept going. You shipped. You're here.

Now let's talk about the architecture, which is — with the warmest possible condescension — probably a mess.

---

## The Diagnosis

You don't need to read your whole codebase to know what's wrong with it. If you've built something substantial through vibe coding, the following is almost certainly true:

**You have a god class.** One class (or module, or file) that everything depends on. It started as "the controller" or "the app" or "the main thing" and grew every time you added a feature, because the LLM's easiest path was always to add the new code where the existing code already lived. It has too many methods, too many member variables, and too many responsibilities. It might show dialogs. It might compute business logic. It might manage the database AND the UI AND the network AND the undo stack.

**You have dead code.** Methods that nobody calls. Variables that are written but never read. Entire subsystems that were superseded and never deleted. The LLM adds code confidently. Neither you nor the LLM deletes code confidently, because what if something still depends on it? (Usually nothing does.)

**You have copy-paste.** The same logic in two or three places, with small variations. The LLM's natural mode is to solve the problem in front of it, and if the solution is similar to something that already exists, it will write a new version rather than refactoring the old one. Each version works. The duplication is a maintenance burden that hasn't bitten you yet.

**You have implicit contracts.** Which of your three APIs for the same operation should a caller use? Which members are valid during which phase of the lifecycle? What order do things need to be initialized in? None of this is documented. You knew it when you wrote it. You don't know it now. The LLM certainly doesn't know it.

**You have timing hacks.** `setTimeout(500)`, `QTimer::singleShot(100)`, `sleep(1)`, `asyncio.sleep(0.5)` — delays inserted to paper over initialization ordering bugs that nobody wanted to fix properly. They usually work. They occasionally don't. When they don't, the failure is intermittent and baffling.

If you're nodding, welcome to the club. The membership fee is one architectural reckoning. This chapter is the receipt.

---

## The Recipe

### Step 1: Don't Rewrite

Your code works. Preserve that. A rewrite destroys accumulated domain knowledge — the edge cases you've handled, the integrations you've debugged, the user-facing behavior that people depend on. A rewrite also takes longer than you think, because you'll encounter the same problems you encountered the first time, plus new ones.

Instead of rewriting, you're going to *extract*. You'll pull the good code out of the mess. What remains will be the mess — concentrated, visible, fixable.

### Step 2: Survey

List the conceptual modules in your application. Not the files — the concepts. You know them even if the code doesn't reflect them:

- "The authentication system"
- "The payment processing"
- "The notification engine"
- "The data model"
- "The API layer"
- "The rendering/UI"
- "The state management"
- "The file/network I/O"

These are your future libraries (or packages, or modules — whatever your language calls them). Write them down. You'll recognize them because you think about them as units, even if the code treats them as one big blob.

### Step 3: Order by Dependency Depth

Which modules depend on nothing? Those are Tier 1. Which depend on Tier 1? Those are Tier 2. And so on.

```
Tier 1 (standalone): Data types, algorithms, utilities, parsers
Tier 2 (domain):     Models, services, business logic
Tier 3 (UI):         Views, components, pages
```

This ordering is your extraction plan. Extract bottom-up: Tier 1 first. These are the easiest — pure computation, pure data, no god-class dependencies. They'll come out clean because they were always conceptually independent.

### Step 4: Extract One Library at a Time

For each module, starting with Tier 1:

1. **Create the library directory** (or package, or module)
2. **Move the files.** Copy them to the new location. Update the imports/includes.
3. **Verify the build.** Does the library build on its own? (If your language supports this — not all do.)
4. **Verify the app.** Does the full application still build and work?
5. **Run the tests.** Do they pass?
6. **Commit.** One extraction, one commit. Revertible if anything goes wrong.

Each extraction should be a single, reviewable, revertible change. Don't extract five modules at once. Don't refactor while extracting. Just move. The refactoring comes later.

### Step 5: Create Interfaces Only When You Must

When a Tier 2 or Tier 3 module needs something from the god class, you'll discover it during extraction: the build will break because the module can't find the god class anymore.

This is the moment of truth. For each broken dependency, ask: "what does this module actually call on the god class?" Use grep (or your IDE's "find usages") to get the answer. Make a list of the exact methods used.

Create an abstract interface with *exactly those methods*. Nothing more. Don't add methods "for completeness." Don't design a taxonomy. Extract what exists.

Change the module code to accept the interface instead of the god class. Have the god class implement the interface. Verify the build. Commit.

### Step 6: Watch the Monolith Shrink

After each extraction, the remaining monolith is smaller, and the coupling is more visible. Files that obviously belong in a library but are stuck in the monolith — move them when their dependencies are met.

Track the indicators of progress:
- How many files are in the monolith vs. libraries?
- How many import/include paths reach from libraries back into the monolith?
- How many methods does the god class expose?
- How many of those methods are called by library code (through interfaces) vs. only by other monolith code?

These numbers should trend in the right direction: monolith shrinking, libraries growing, coupling decreasing.

### Step 7: Audit the Residue

When the extractions are done, the remaining monolith is the concentrated mess. This is the moment the chronicle has been building to.

Look at it. Really look. Document what you find. Here's what you'll probably discover:

- **Dead methods** — methods with zero callers. Use your IDE's "find usages" or grep. If nothing calls it, delete it.
- **Dead variables** — members that are written but never read. If nothing reads it, delete it.
- **Redundant APIs** — multiple ways to do the same thing. Pick the best one. Redirect callers. Delete the rest.
- **Copy-pasted logic** — the same code in multiple places with small variations. Extract a shared function.
- **Timing hacks** — delays papering over initialization ordering bugs. Replace with proper completion signals or callbacks.
- **Blurred responsibilities** — controllers showing UI, views computing business logic. Separate them.

Write it all down. Make a list. The list will be long. But it will be *finite*. And every item on it is now a straightforward fix, because the extraction removed the hidden dependencies that made fixes scary.

If you can, make the list funny. The amusement is the proof that the problem has become tractable. You don't laugh at things that terrify you.

### Step 8: Clean the Residue

Work through the list. One item at a time. Each fix is small, verifiable, and safe — because the libraries don't depend on the internals. They talk to interfaces.

- Delete dead code. The compiler is your proof.
- Deduplicate copy-paste. Extract shared helpers.
- Fix semantic bugs. The ones that were invisible before are visible now.
- Rationalize APIs. One way to do each thing.
- Separate concerns. Controllers control. Views view. Models model.

Commit after each cleanup. Small changes. Reviewable. Revertible.

---

## The Tools

**What you need:** Your compiler (or interpreter, or bundler). Grep (or "Find Usages" in your IDE). Your build system. Version control. That's it.

**What you don't need:** A software architecture degree. A design patterns book. A whiteboard. A consultant. The architecture is already in your code. You're not designing a new building — you're excavating the one that was always there.

**What helps:** An LLM. Yes, the same tool that created the mess can help clean it up. LLMs are excellent at the mechanical parts of extraction: updating import paths, creating interface headers from usage lists, moving files and fixing build configurations. The human provides the judgment: what to extract, what to keep, where the boundaries should be. The LLM provides the labor.

---

## The Mindset

Archaeology, not architecture.

You're not designing a new system. You're excavating an existing one. The modules are already there — they're just buried under the sediment of organic development. The interfaces are already there — they're implicit in the grep results. The dead code is already there — it's just hidden by the noise of a large codebase.

Your job is to remove the sediment, expose the structure, and clean the residue.

This is less glamorous than "designing a clean architecture from scratch." It's also more useful, because it preserves everything that works — every feature, every edge case, every hard-won integration — while giving it a structure it can grow into.

---

## The Timeline

How long does this take? PlanStan's extraction — 140K lines, 11 libraries, 5 interfaces — took about two days.

Your project is probably smaller. Your language might make extraction easier (Python packages vs. C++ libraries). Your god class might be less entangled (or more — god help you if it's more).

A reasonable estimate: for a 20K-50K line project, expect a few days to a week. For a larger project, maybe two weeks. The time scales linearly with the number of modules to extract, not with the total lines of code.

The cleanup phase (Step 8) takes longer — probably a few weeks of part-time work. But it's the kind of work you can do incrementally, one cleanup per session, interleaved with feature development.

---

## The Emotional Progression

You will experience these feelings in roughly this order:

1. **Dread.** Before you start. The codebase is big and scary and you don't want to look.
2. **Relief.** When the first library extracts cleanly. "Oh — that was always independent. It just didn't know it."
3. **Effort.** When the middle extractions require interfaces. It's work. But it's mechanical work with visible progress.
4. **Amusement.** When you see the concentrated residue and realize the mess is actually kind of funny.
5. **Satisfaction.** When the dead code is deleted, the APIs are rationalized, and the tests still pass.
6. **Pride.** When you look at the result: a working application, organized into clean libraries, with a thin shell. The features didn't change. The users didn't notice. But you can see the structure, and you're no longer afraid of your own code.

---

## Closing: The Milestone

You started with a working application buried in a mess. You end with a working application organized into clean libraries with a thin, comprehensible shell. The features didn't change. The users didn't notice. But you can see the structure now, and you're no longer afraid of your own code.

To the vibe coder who built something ambitious and is now terrified of it: the fact that you're terrified means you've built something worth being terrified of. That's not a failure. That's a milestone. A terrifying codebase means you've built something complex enough to have architectural debt — and that means you've built something real.

The next milestone is making it clean. This chronicle shows you how. The tools are your compiler and grep. The method is extraction, concentration, and cleanup. The mindset is archaeology. The timeline is days, not months.

And to the vibe coder who hasn't looked under the hood yet: look. What you find will be funny. What you do about it will make you a better developer than you were yesterday. And the thing you built — the thing that works, that does what you wanted, that makes people say "wait, you built that?" — will still work when you're done.

The code speaks. The results speak. The imposter syndrome is a ghost. The extraction is the exorcism. Get to work.

---

[< Previous: The Principles](TransmogrifyingVibeCode_10_Principles.md) | [Back to Chapter 1](TransmogrifyingVibeCode_01_WhyThisMatters.md)
