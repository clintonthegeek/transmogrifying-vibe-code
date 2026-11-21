# Transmogrifying Vibe Code: A 13-Month Chronicle

*How one developer — who hadn't coded in 15 years — used LLMs to build a 140K-line professional application, discovered the architectural debt that vibe coding always produces, and paid it off. A field report from the future of software development.*

---

## The Invisible Revolution

Something is happening to software, and almost nobody is talking about it honestly.

GitHub Copilot completions are in production at every Fortune 500 company. Junior developers are shipping LLM-generated functions without meaningful review. Startups are building MVPs in weekends with Claude, ChatGPT, and Cursor. The code works. The features ship. The demos are impressive. The investors are satisfied.

Nobody is asking what happens next.

Not "next" as in next quarter. "Next" as in: what happens when that weekend MVP becomes a real product? When the 2,000-line prototype becomes 20,000 lines? When the 20,000 lines become 50,000? When the 50,000 become 140,000, and the feature you need to add next touches five files that nobody — not you, not the LLM — fully understands?

Vibe coding has entered the seams of the software industry with the quiet insistence of water finding cracks in concrete. It works well enough, often enough, that questioning it feels like questioning electricity. But electricity has wiring standards. It has circuit breakers. It has building codes enforced by inspectors who understand that the thing that lights your house can also burn it down.

Vibe-coded software has none of this. Not yet. The standards don't exist because the failure modes haven't been widely experienced. The building inspectors haven't been trained because nobody has documented what the fires look like.

This chronicle is the fire report.

---

## The Specimen

Here is what happened to one person.

A developer who hadn't written code in 15 years picked up LLMs as programming tools in February 2025. Over the next 13 months, through six project incarnations, four different LLM platforms, 175+ documented coding sessions, and 768 git commits, they produced a 140,000-line C++20/Qt6 personal project management application. Not a toy. Not a demo. A professional application with:

- **KDDockWidgets** integration with docking, floating, and tabbed panes
- **A custom Gantt chart** with CPM forward scheduling, dependency arrows, and effort-based rendering
- **An interactive PERT dependency graph** with Sugiyama layout, edge creation, and a minimap
- **A three-way diff sync engine** with transactional conflict resolution across CalDAV, local ICS, org-mode, and DecSync backends
- **A modular incidence editor** with 11 pluggable inspector modules
- **Multiple calendar renderers** including standard, line, and handwritten styles
- **Dark mode**, **semantic zoom**, and a **theming system** with cascading style rules
- **11 standalone libraries** that build independently, each exporting a CMake alias target

This application works. Users can load it, create projects, schedule tasks, sync calendars, manage dependencies, view Gantt charts, and drag items between views. The surface is polished. The features are real. By any reasonable external measure, this is a successful software project.

Now look beneath the surface.

---

## What Lives Underneath

`CollectionController`: 66 methods. 30 member variables. 535 lines of header. Two initialization methods that share 150 lines of copy-pasted code — one of which has zero callers anywhere in the project. It is 216 lines of lovingly maintained dead code, kept in sync across refactors for a function that nobody invokes. It is a plastic plant that someone has been watering for months.

Three separate API layers for modifying incidences, each written at a different stage of understanding, each with different semantics, none documented. One of them is a booby trap: the `ExceptionManagerDialog` still calls the old API, silently creating the wrong command type. Nobody noticed because the old API *mostly* works.

A `QHash<QString, FetchOperation*>` that stores pointers to fetch operations. The pointers are inserted. They are never dereferenced — not for cancellation, not for progress, not for status, not for anything. They exist so the hash can answer "does this key exist?" — which a `QSet` with the same keys already answers. The `FetchOperation*` pointers are also never deleted. They leak. Every single fetch.

A controller that shows modal dialogs during initialization. A class named "View" that is simultaneously a Controller, Mediator, Factory, and Layout Manager, with 84 includes and a 305-line context menu method. An `AppSettings` singleton whose static methods don't use the singleton instance. A geographic coordinate check that considers the Gulf of Guinea — latitude 0.0, longitude 0.0 — an invalid location.

`QTimer::singleShot(500, ...)` as architecture. Not once. Four times. A 500-millisecond prayer that the data will be ready. On a fast machine, wasted time. On a slow machine, a race condition. The software equivalent of closing your eyes, counting to three, and hoping the traffic light changed.

All of this **worked**. The surface was immaculate. Users never saw any of it. That is precisely the danger.

---

## The Thesis

Every vibe-coded application that succeeds will eventually produce this archaeology. Not because LLMs write bad code — they write *adequate* code, often quite good code — but because the organic process of building software through conversation accumulates structural debt in ways that neither the developer nor the LLM can see happening.

The LLM doesn't maintain a mental model of your architecture across sessions. It can't see that this is the third API for the same operation. It doesn't know that `initialize()` stopped being called three months ago. It solves the problem in front of it — accurately, efficiently, locally — without the global awareness that a senior architect would bring to the same decision.

The developer, if they're new to coding or returning after a long absence, can't see the debt accumulating either. They can see that features work. They can see that the UI looks right. They can't see that coupling is increasing, that responsibilities are blurring, that dead code is accreting, that the same logic is being written in three places because nobody remembers the first two.

The debt compounds silently until the day you try to add a feature that touches more than one module, and you discover that everything depends on everything and nobody documented the contracts.

**This is not a failure of vibe coding. It is a predictable, mechanical consequence of the process.** And it has a predictable, mechanical solution.

---

## The Solution, In Brief

You don't clean up a monolith by fixing the monolith. You clean it up by extracting the good code out of it. What remains is the mess — concentrated, visible, and small enough to kill.

This inverts the usual instinct. The usual instinct is to attack the worst code first: refactor the god class, break up the 300-line method, rename the confusing APIs. This is the wrong order. You'll spend enormous effort fighting a tangle that resists decomposition because everything depends on everything.

Instead: extract the code that's already good — the pure algorithms, the data types, the self-contained subsystems — into standalone libraries. Where they genuinely need something from the mess, create a narrow interface. Repeat until the mess is alone. Then audit it, laugh at it, and fix it.

In PlanStan's case: 11 libraries extracted. 5 interfaces discovered by grep, not designed from theory. 140K lines reorganized into 92K in libraries plus 48K in the application shell. The pathology that was previously distributed across 582 files concentrated into 8 files. The god class still has 66 methods, but now you can see every one of them, enumerate the dead code, count the redundant APIs, and fix them one by one — because the libraries don't depend on the internals anymore. They talk to interfaces.

The structure was always there. The extraction revealed it. The mess was always there too. The extraction made it funny instead of terrifying.

---

## Why This Matters Beyond One Project

This chronicle is not about PlanStan. PlanStan is the specimen. The chronicle is about what's coming for everyone.

**Millions of people are about to have this problem.** Every vibe-coded application that succeeds will eventually hit the wall where adding one feature requires understanding five files, and nobody — not the developer, not the LLM — understands how they connect. The hobby project that grew. The startup MVP that got funded. The internal tool that became critical infrastructure. The code works, the users depend on it, and the architecture is a labyrinth.

**Nobody is documenting the full lifecycle.** There are plenty of "I built an app with AI in a weekend" posts. There are zero accounts of what happens at month 6, month 10, month 13 — when the codebase is 140K lines and the god class has 66 methods and you can't remember which of three CRUD APIs is the real one. This is that account.

**The solution exists and it's mechanical.** You don't need to be a senior architect. You need `grep` and discipline. The structure is already there in your codebase — you just have to excavate it. This chronicle is the excavation manual.

**The developer's story obliterates the imposter narrative.** A person who hadn't coded in 15 years, building alone, produced something that would have taken a team of experienced developers months. They didn't need credentials. They needed vision, taste, and the stubbornness to demand better from every LLM they worked with. That's the future of software development, and it's not an imposture — it's a paradigm shift.

---

## The Chronicle

This document is the first chapter of a multi-part chronicle. Each chapter covers a different dimension of the story — the developer, the vision, the incarnations, the art, the surface, the architecture, the mess, and the method.

| Chapter | Title | What It Covers |
|---------|-------|----------------|
| **1** | [Why This Matters](TransmogrifyingVibeCode_01_WhyThisMatters.md) | You are here. The world, the specimen, the thesis. |
| **2** | [The Developer](TransmogrifyingVibeCode_02_Developer.md) | A 15-year gap, a demanding personality, and the discovery that prose is code now. |
| **3** | [The Vision](TransmogrifyingVibeCode_03_Vision.md) | What was being built and why. ADHD, GTD, and the plan-first paradigm. |
| **4** | [The Tumbleweed](TransmogrifyingVibeCode_04_Tumbleweed.md) | Six incarnations in 13 months. Kalibrator to PlanStan, each abandoned not because it failed but because the developer outgrew it. |
| **5** | [The Art of Vibe Coding](TransmogrifyingVibeCode_05_ArtOfVibeCoding.md) | What vibe coding is, what it isn't, and why the bell metaphor matters. |
| **6** | [The Surface](TransmogrifyingVibeCode_06_Surface.md) | The UI-first strategy and why getting the surface right before the architecture is not laziness — it's user-centered development. |
| **7** | [The File I/O Story](TransmogrifyingVibeCode_07_FileIO.md) | Where vibe code got architecture right, because file I/O punishes sloppiness immediately. |
| **8** | [The Warning](TransmogrifyingVibeCode_08_Warning.md) | The forensic analysis. What lives beneath the surface of every successful vibe-coded application. |
| **9** | [The Method](TransmogrifyingVibeCode_09_Method.md) | How to fix it. Extraction, interfaces, concentration, cleanup. |
| **10** | [The Principles](TransmogrifyingVibeCode_10_Principles.md) | Twelve principles for the AI-coding era, distilled from 13 months of practice. |
| **11** | [For the Vibe Coder With a Pile of Spaghetti](TransmogrifyingVibeCode_11_Recipe.md) | The practical recipe. Step by step, with warmth and gentle condescension. |

Read them in order, or skip to the chapter that addresses your situation. If you're a vibe coder who just realized your codebase terrifies you, start with [Chapter 11](TransmogrifyingVibeCode_11_Recipe.md). If you're a skeptic who doubts that LLM-assisted development can produce real software, start with [Chapter 6](TransmogrifyingVibeCode_06_Surface.md). If you're a manager wondering what's going to happen to all the AI-generated code your team is shipping, start right here and don't stop.

The fire is already burning. This is the report.

---

*This chronicle documents the development and architectural reconstruction of PlanStan, a 140K-line C++20/Qt6 personal project management application built entirely through LLM-assisted development over 13 months. The principles are language-agnostic. The story is universal. The mess is probably in your codebase right now.*
