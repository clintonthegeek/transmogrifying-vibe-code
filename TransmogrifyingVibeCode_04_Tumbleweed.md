# Chapter 4: The Tumbleweed

*Six incarnations in 13 months. Each abandoned not because it failed but because the developer outgrew it. The story of a project that kept starting over — and why that's the most important advantage of the LLM era.*

[< Previous: The Vision](TransmogrifyingVibeCode_03_Vision.md) | [Next: The Art of Vibe Coding >](TransmogrifyingVibeCode_05_ArtOfVibeCoding.md)

---

## Why "Tumbleweed"

In the American West, tumbleweeds start as living plants rooted in soil. When they mature, they snap off at the base and roll across the landscape, scattering seeds as they go. The plant that breaks free isn't dead — it's in its reproductive phase. Each landing site is a potential new colony.

PlanStan tumbled six times. Each incarnation was a living plant that grew until it couldn't grow anymore in its current soil, broke free, and scattered its seeds into the next iteration. Nothing was truly lost. The CalDAV patterns from Kalibrator survived into PlanStan. The philosophy from TimeBuster became the manifesto. The layered architecture from PlanBuster became the blueprint. The code changed; the knowledge accumulated.

In the pre-LLM era, starting over was expensive. Months of work, discarded. In the LLM era, starting over is cheap — a weekend, maybe a week. The speed changes the calculus completely. When starting over costs months, you cling to what you have and try to fix it in place. When starting over costs days, you can abandon an architecture the moment you outgrow it, carry forward the lessons, and build the next version on better foundations.

Six incarnations in 13 months. Each one 3-9 days of focused work. Each one better than the last. This is the tumbleweed advantage, and it's native to LLM-assisted development.

---

## Incarnation 1: Kalibrator (February 28 - March 3, 2025)

**Three days. 20 commits. The overnight marathon.**

It started at 11 PM on February 27 with Grok on X and ended, in a sense, 72 hours later with a working application skeleton and 1,500 compiler errors along the way.

The name was "GrokCal" for the first few hours — the developer and Grok naming the project together like parents naming a child at 2 AM, which is approximately what was happening. By morning it was "Kalibrator," which had the virtue of being unique and the deficit of being impossible to spell.

The architecture was ambitious for a first attempt: custom `CalendarItem` wrappers with polymorphic `EventItem`/`TodoItem` subclasses, a `SyncState` enum (`Clean`, `Dirty`, `New`, `Deleted`) with ETag tracking built into the model, a `SessionController` for UI/data glue, and a SQLite database for sync metadata. The subfolder structure — `collections/default/<calendarName>/` — survived all the way to the final project.

The CalDAV integration was the crucible. KDAV, KDE's WebDAV library, documented its `start()` method as non-blocking. In practice, it blocked. The developer tried everything: `QTimer::singleShot` for async nudging (still froze), `QThreadPool` for offloading (crashed harder), raw KJobs on the main thread (worked, sort of, but lifecycle management was fragile). At 3 AM, after qmake proved intractable, the developer pivoted to cmake — a decision that would persist through every subsequent incarnation.

The lesson that emerged was hard-won and permanent: KDAV's sequential sync with explicit cleanup is more reliable than async parallelism, no matter what the documentation promises. This lesson was encoded as a commandment in every subsequent LLM session:

> "Don't mess with the .ics-handling logic. Do not mess with the CalDAV handling logic. Got it? And if we have to make modifications, make damn sure that you understand what it's doing and why it works!"

Kalibrator wasn't usable. It was barely compilable. But it established the patterns — CalDAV sync, file-per-calendar storage, KDE framework integration — that would persist through every incarnation.

**Seeds scattered:** CalDAV sync patterns, file-based calendar storage, cmake build system, the lesson about sequential sync.

---

## Incarnation 2: KalibratorRestored (March 3-4, 2025)

**One day. A surgical retreat.**

Documented as "Frankenstein's monster, stitched together from exasperation."

Kalibrator had accumulated too many experimental approaches in too short a time. The sync infrastructure was broken in three different ways simultaneously. The CalendarItem wrappers added complexity without value. The SQLite metadata layer was over-engineered for the current state of the application.

KalibratorRestored was a strategic retreat: strip the broken sync infrastructure while keeping the core `CalendarItem`/`Calendar` classes, basic `.ics` file I/O, and the subfolder structure. A revert that preserved the lessons.

This was the developer's first experience with what would become a recurring pattern: the moment when you realize that the code you've written in the last few days has taught you something important about the code you *should* write, and the fastest path forward is to start over with that knowledge rather than to fix what you have.

In the pre-LLM era, this realization leads to despair. Days of work, thrown away. In the LLM era, it leads to a clean directory and a fresh conversation. The cost of starting over had dropped below the cost of fixing in place, and the developer had discovered this empirically.

**Seeds scattered:** The conviction that wrapper classes add complexity. The instinct to retreat and simplify rather than push forward through a tangle.

---

## Incarnation 3: Kalburator (March 4-10, 2025)

**Six days. The architecture explosion.**

If Kalibrator was a cabin, Kalburator was an attempt to build a cathedral.

The file count exploded from approximately 20 to over 60. New architectural concepts proliferated:

- **TransPipeline** and **TransStore** for operation sequencing
- **AbstractCollectionSyncWorker** with `DownloadSyncWorker` and `UploadSyncWorker` subclasses
- **LocalSyncEngine** and **KDAVSyncEngine** abstractions
- **CollectionManager** for multi-collection handling
- `AddEventOperation`, `CreateCalOperation`, `CreateFileOperation` — separate classes for each operation type

Sync became the central architectural concern. The developer, still learning to calibrate how much architecture is appropriate for a given stage of development, let the LLM build out full abstraction layers for problems that didn't yet exist. Background workers, job queuing, proper sync engine abstractions — all technically sound, all premature for an application that couldn't reliably display a calendar yet.

This is a pattern that every vibe coder will encounter: the LLM is happy to build any abstraction you ask for, and it will build it competently. The question of whether you *should* ask for it — whether the abstraction earns its complexity at this stage of development — is a judgment the LLM cannot make and the novice developer hasn't yet learned to make.

The developer was learning. Kalburator taught them what "over-engineered" looks like from the inside: a codebase where every operation has its own class, every class has its own file, and the ceremony of the architecture exceeds the complexity of the actual work.

**Seeds scattered:** The understanding that architecture should follow need, not anticipate it. The experience of over-engineering as a felt quality — not something you read about, but something you lived through and recoiled from.

---

## Incarnation 4: TimeBuster (March 10-16, 2025)

**Six days. Philosophy crystallizes.**

TimeBuster was where the project stopped being a calendar application and became something else.

The manifesto arrived. The developer wrote (or co-wrote with Grok — the transcripts don't always make the authorship clear) the foundational document that would guide every subsequent incarnation:

> "This is an all-purpose IDE for time management... for power-users who aren't afraid to get their hands dirty, and who want full transparency and control over their schedule and tasks."

The proactive/passive view concept emerged: proactive views (MDI subwindows that can set selection focus) versus passive views (panes that react to selection changes). The `.kalb` configuration format — recursive JSON merging from collection root plus calendar subfolders — was designed and implemented. The ViewInterface architecture provided a stable contract between the view layer and the data layer.

But the most important development in TimeBuster was the developer finding their voice. The Grok transcripts from this period are noticeably different from the Kalibrator sessions. The developer stops asking "how should I do this?" and starts saying "this is what I want":

> "This is a tool for personal computer owners to own their personal data on their own personal computer that they own."

> "There is no stupid 'server wins' logic. Every single hiccup is logged and presented."

> "No autosave. No inference. No telemetry."

The transition from "help me build a calendar app" to "I have a specific vision and you're going to help me implement it" happened somewhere in those six days. The developer went from using the LLM as a teacher to using it as a tool. The demanding boss persona wasn't an affectation — it was the natural consequence of having a clear vision and encountering a tool capable of executing it.

TimeBuster also produced the "baton" system. The Grok transcripts from March 15-16 run to 155,000 words in a single session — a record that should be physically impossible given context window limitations. What actually happened was a series of sessions connected by handoff documents: summaries of what had been accomplished, what the current architecture looked like, and what should not be touched. These batons were the developer's solution to the context window problem, invented independently before most AI practitioners had even named the problem.

One document from this period carries a remarkable title: "Last Testament of a schized out Grok Mar 15." It records the moment when the LLM conversation had degraded past the point of usefulness — the model contradicting itself, generating code that conflicted with code it had generated ten messages earlier, losing track of the architecture it had helped build. The developer documented the degradation not with frustration but with clinical interest, as a phenomenon to be understood and worked around.

**Seeds scattered:** The philosophical manifesto. The proactive/passive view concept. The .kalb configuration format. The baton system for session continuity. The understanding that LLM conversations have a shelf life.

---

## Incarnation 5: PlanBuster, Then PlanStan (April 2025)

**The clean break.**

PlanBuster was the transition from "learning to code with an LLM" to "using an LLM to code." The architecture was deliberate:

| Layer | Responsibilities |
|-------|-----------------|
| **IcsIO** | Persistence, sync backends |
| **ModelLib** | In-memory state, undo staging |
| **ControllerLib** | Orchestration, commands |
| **ViewLib** | Qt UI |

QUndoStack integration arrived — proper command pattern for all incidence operations, with per-collection undo stacks. The `.kalb` configuration format matured. Direct KCalendarCore usage replaced the custom wrapper classes that had caused so much friction in Kalibrator and KalibratorRestored.

And then came the document that started PlanStan proper: the "Team B" prompt.

858 lines of specification. A complete architectural blueprint including layer responsibilities, key classes, interfaces, signal flows, and development stages. The developer wrote it as a handoff to a new LLM (moving from Grok to Claude), and it reads like the kind of technical specification that a senior architect produces for a development team:

> "Hello. You are Team B. You are going to help me implement PlanStan, a new version of a program called PlanBuster that I've been working on for a while."

Not "can you help me build?" — "you are going to help me implement." The developer had fully internalized the boss-tool dynamic, and the specification reflected months of accumulated architectural knowledge. The layered separation. The backend abstraction. The undo system. The config format. The MDI workflow. All of it crystallized from four previous incarnations into a document that any professional developer could use as a starting point.

PlanStan — the name that would stick — was born from this prompt. The rename from PlanBuster to PlanStan was a signal: this was the version that would last. No more tumbling. The foundations were correct.

**Seeds scattered:** Clean layered architecture. QUndoStack integration. Direct KCalendarCore usage. The "Team B" prompt as a model for LLM-assisted project specification. The final name.

---

## Incarnation 6: PlanStanLite, Then PlanStan (April 23, 2025 - February 2026)

**Ten months. 768 commits. The final form.**

PlanStanLite was supposed to be a simplified version of PlanStan — hence "Lite." It quickly outgrew the name. By the time it was renamed back to "PlanStan" (the name nobody could kill), it was a 140,000-line application with four backend types, a custom Gantt chart, a dependency graph, a forward scheduler, org-mode integration, and 11 standalone libraries.

The commit history tells the emotional story:

**April-May 2025 (37 commits):** Initial build. The foundations from the Team B prompt realized in code. Backends, models, controllers, views. The core loop: load a collection, display calendars, edit incidences, sync changes.

**May-July 2025 (30 commits):** Stabilization. The first attempt at org-mode integration, which revealed that the iCalendar data model couldn't represent what org-mode offered. The seeds of the paradigm shift, not yet germinated.

**July 16, 2025 - January 4, 2026: The Gap.**

Six months of silence. Zero commits. The ADHD dimension of the story, visible in the git log as a six-month absence between July and January. Not burnout — the application worked, the features were real, the vision was intact. But the executive function that drives daily engagement with a project went offline, and six months passed.

This gap is important for the chronicle because it's important for the audience. Every ADHD developer knows this gap. Every vibe coder with ADHD will hit this gap. It feels like death — like the project is over, like you've failed, like the momentum is gone and can never return.

It's not death. It's dormancy. The tumbleweed was waiting.

**January 4, 2026 (9 commits):** The return. Whatever neurological switch flipped, it flipped hard. Nine commits on the first day back. The project was alive.

**January 5, 2026 (32 commits):** The explosion. Thirty-two commits in a single day. This was the day the paradigm shift arrived — the plan-first architecture, the separation of planning from scheduling, the insight that org files should own the plan while the calendar owns the schedule. The developer's vision, which had been developing for months in their head even while the git log was silent, poured out in a torrent of code.

**January 10-31 (20-55 commits/day):** The sustained sprint. New features arriving daily: the planning subsystem, the dependency graph, the forward scheduler, project metadata stores, companion file management. The codebase growing by thousands of lines per day, each day leaving the application in a working state.

**February 1-9, 2026 (15-60 commits/day):** Peak intensity. February 2 alone: 60 commits. A custom Gantt widget (replacing the KGantt dependency entirely). An Akonadi backend. A multi-month tiling view. A temporal scroll stack. Each feature implemented from specification to working code in hours.

And then the reckoning: the library decomposition. Two days of extraction that reorganized 140K lines into 11 standalone libraries and a thin app shell, creating 5 interfaces along the way. The forensic analysis that documented the mess with comedy rather than shame. The concentrated residue that could finally be seen clearly and cleaned systematically.

**Seeds scattered:** Everything. This is the final form. The seeds it scatters are the libraries themselves — 11 of them, each capable of independent existence, each carrying the accumulated wisdom of six incarnations and 13 months of development.

---

## The Tumbleweed Advantage

Six incarnations in 13 months. To a traditional software manager, this looks like six failures. To a vibe coder, it looks like six progressively better starting points. The difference is the cost of starting over.

In traditional development, starting over means:
- Weeks or months of work discarded
- Team morale damaged
- Stakeholder trust eroded
- Everything rewritten from scratch

In LLM-assisted development, starting over means:
- A week of work composted into lessons
- A specification document (the "baton") that captures everything learned
- A fresh conversation with a clear vision and correct architectural assumptions
- Everything rebuilt in days rather than weeks, because you know exactly what to build this time

The tumbleweed pattern isn't a bug. It's a feature of the LLM era. When starting over is cheap, the optimal strategy changes. Instead of building a monolith and trying to fix it in place, you build a prototype, learn from it, and start again with better foundations. Each iteration is faster because you know more. Each architecture is better because you've lived through the failure modes of the previous one.

Kalibrator taught the developer about CalDAV and the cost of wrapper classes. KalibratorRestored taught them about strategic retreat. Kalburator taught them about over-engineering. TimeBuster taught them about vision and philosophy. PlanBuster taught them about clean layers and undo systems. PlanStan taught them about paradigm shifts and the plan-first architecture.

No single incarnation could have taught all of these lessons. Each was a necessary stage in the developer's education. And the education was cheap — days per incarnation, not months — because LLMs compressed the implementation cycle from "design, code, test, iterate" to "describe, generate, evaluate, start over."

The tumbleweed scatters seeds. Each incarnation that dies fertilizes the next. By the sixth incarnation, the soil was rich enough to sustain 140,000 lines and 768 commits. That soil was made of everything that came before.

---

## A Note for the Vibe Coder on Their Second Project

If you're reading this and your current vibe-coded project is a mess, you have two options:

**Option A:** Extract the good code into libraries (the subject of Chapter 9). This is what PlanStan did, and it works.

**Option B:** Start over. Carry forward the lessons, not the code. Write a specification document that captures everything you've learned about the domain, the architecture, the features that matter and the features that don't. Hand it to your LLM and say: "Here is what I want. Build it right this time."

Option B is not defeat. It's the tumbleweed pattern. It's the natural lifecycle of LLM-assisted development. Kalibrator's code is gone, but Kalibrator's lessons live in PlanStan's sync engine. TimeBuster's code is gone, but TimeBuster's philosophy lives in PlanStan's manifesto.

The question isn't "can I save this code?" The question is: "do I know enough now to write the specification for the code I actually want?" If yes, the tumbleweed is ready to roll.

---

[< Previous: The Vision](TransmogrifyingVibeCode_03_Vision.md) | [Next: The Art of Vibe Coding >](TransmogrifyingVibeCode_05_ArtOfVibeCoding.md)
