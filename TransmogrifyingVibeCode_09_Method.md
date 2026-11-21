# Chapter 9: The Method

*How to fix it. The extraction methodology. You don't clean up a monolith by fixing the monolith. You clean it up by extracting the good code out of it, and what remains is the mess — concentrated, visible, and small enough to kill.*

[< Previous: The Warning](TransmogrifyingVibeCode_08_Warning.md) | [Next: The Principles >](TransmogrifyingVibeCode_10_Principles.md)

---

## The Fundamental Insight

Your first instinct will be wrong.

You'll look at the god class with its 66 methods and 30 member variables and think: I need to break this up. I need to refactor these methods. I need to separate these concerns. I need to fix this class.

No. Leave the god class alone. Leave the 305-line context menu method. Leave the three redundant CRUD APIs. Leave the memory leak. Leave the copy-paste. Leave all of it.

Instead, look at the code around the god class. The scheduling algorithm that doesn't need the UI. The tag system that doesn't need the sync engine. The org-mode parser that doesn't need the calendar views. The temporal focus manager that doesn't need the editor. These are your future libraries. They're already conceptually independent — they just don't know it yet.

**Extract the good code into libraries. What remains is the mess — concentrated, visible, and small enough to kill.**

This inverts the usual instinct. The usual instinct is to attack the worst code first. To refactor the god class. To break up the 300-line method. This is wrong because:

1. The worst code is the worst code *because* it's entangled. You can't fix it without understanding its dependencies.
2. You can't understand its dependencies because they're implicit and scattered.
3. You'll spend enormous effort fighting a tangle that resists decomposition because everything depends on everything.

Instead:

1. Extract the code that's already good — it just needs to be moved out.
2. Where a library genuinely needs something from the mess, create a narrow interface.
3. Repeat until the mess is alone.
4. Audit the mess. Laugh at it.
5. Fix the mess — now easy, because you can see it clearly and nothing outside depends on its internals.

---

## The PlanStan Case Study

PlanStan was a roughly 140K-line monolith organized as a `PlanStanCore` library and an application shell. In practice, everything included everything, and `CollectionController` wired the entire system together.

The extraction reorganized 140K lines into 92K in libraries plus 48K in the app shell, with 5 interfaces decoupling the two. The pathology that was previously distributed across 582 files concentrated into 8 files.

Here's exactly how it was done.

### Step 1: Identify the Natural Libraries

Before writing any code, survey the codebase. List the conceptual modules. You know them even if the code doesn't reflect them: the scheduling algorithm, the tag system, the org-mode parser, the temporal navigation, the sync engine, the models, the planning subsystem, the editor, the Gantt chart, the dependency graph, the calendar views.

Organize them into tiers by dependency depth:

```
Tier 1 (standalone foundations — depend only on Qt and system libs):
  scheduling, tags, temporal, org-io

Tier 2 (domain libraries — may depend on Tier 1):
  models, sync, planning

Tier 3 (UI libraries — may depend on Tier 1 and 2):
  editor, gantt, graph, calendar-views
```

The tiers are the extraction plan. Extract bottom-up: Tier 1 first, then Tier 2, then Tier 3. This order ensures that each library's dependencies are already extracted when it's time to move it.

No library depends on the app shell. This is the invariant. Every dependency arrow points down (to lower tiers) or sideways (within the same tier), never up.

### Step 2: Extract Tier 1 — The Easy Ones

Tier 1 libraries are pure computation. No god-class dependencies. No UI entanglement. Move the files, update the includes, update CMake, verify build and tests. Each extraction takes an hour or less.

The scheduling library: `DependencyGraph`, `ForwardScheduler`, `EffortParser`, `PriorityFunction`, `DeterminacyDetector`. Pure algorithms. Depends only on Qt::Core. Moved cleanly.

The tags library: `TagSettings`, `CategoryManager`, `TagChipsWidget`, `TagDragData`, `ColorUtils`. A self-contained subsystem. Depends on Qt::Core and Qt::Widgets. Moved cleanly.

The temporal library: `TemporalFocusManager`, `FilterManager`, `RangeScrollBar`, `TemporalScrollStack`, `TemporalRibbon`. Temporal axis management. Depends on Qt::Core and Qt::Widgets. Moved cleanly.

The org-io library: `OrgFileManager`. Pure I/O. Depends on Qt::Core, KF6::CalendarCore, and OrgModeParser. Moved cleanly.

**Key discipline: each library must build independently.** Use CMake's `PROJECT_IS_TOP_LEVEL` to gate in-tree private dependencies. If a library can't build as a standalone project — with only its declared dependencies — the extraction isn't done. This is a bright-line rule. No exceptions. No "it works in-tree so it's fine." Standalone build capability is the proof that the separation is genuine.

### Step 3: Extract Tier 2 — Where Interfaces Begin

Tier 2 was harder. These libraries have real dependencies on the god class.

The sync engine called `CollectionController` for backend access, model queries, and incidence CRUD. The planning subsystem needed the undo stack. The models library needed settings types that lived in the app shell.

These dependencies are real — you can't just move the files and hope. This is where interfaces come in.

### Step 4: Discover Interfaces by Grep

For each library that genuinely needs something from the god class, the process was mechanical:

1. `grep` every file in the library for `CollectionController` method calls
2. List the distinct methods used
3. Create an abstract interface with exactly those methods
4. Change the library code to accept the interface instead of `CollectionController`
5. Have `CollectionController` implement the interface
6. Verify build and tests

No design genius required. Just `grep` and discipline.

The interfaces that emerged:

| Interface | Library | Methods | What It Replaced |
|-----------|---------|---------|-----------------|
| `ICommandDispatcher` | models | 2 | Direct CC dependency for undo |
| `IEditorServices` | editor | 6 subsystem accessors | Direct CC dependency for dock editor |
| `IViewServices` | calendar-views | 6 view-context methods | Direct CC dependency for subwindows |
| `ISyncHost` | sync | 11 backend/CRUD methods | Direct CC dependency for sync engine |
| `ILocaleSource` | models (global) | 5 locale/timezone methods | Static AppSettings calls from 18 files |

`CollectionController` implements all five interfaces. The libraries don't know that. They see only their narrow slice.

The narrowness is the point. `ICommandDispatcher` has 2 methods. Not 10. Not 20. Not "all the methods that seem related to commands." Two methods: `undoStack()` and `pushUpdateCommand()`. Because that's what the grep found. Those are the only methods the consuming code actually calls.

A narrow interface is easy to implement, easy to test, and hard to abuse. A wide interface is just a god class wearing a mask.

### Step 5: Move the Orphans Home

With interfaces in place, many files in the app shell turned out to belong in libraries. Settings data types that five libraries depended on? Move them to `libs/models`. The color utility class? Move it to `libs/tags`. The date button widget? Move it to `libs/editor`. The planning metadata command? Move it to `libs/models` next to the store it operates on.

Each move eliminated a PRIVATE include path leak — a place where a library reached into the app shell's directory structure. In CMake terms: `target_include_directories(PRIVATE ${CMAKE_SOURCE_DIR}/src/...)`. Each such path is a coupling point. Track them. Eliminate them. When they're all gone, the libraries are truly independent.

The orphan moves were satisfying in the way that pulling weeds is satisfying. The garden gets visibly cleaner with each one.

### Step 6: Move the Heavyweight Consumers to the Shell

Some code legitimately needs the full god class surface. `CollectionView` makes 31 distinct method calls on `CollectionController` — 147 invocations total. No narrow interface could serve it. An interface with 31 methods would be a god class wearing a mask.

Solution: move `CollectionView` from `libs/calendar-views` to `src/views/`. It's the app's main UI shell. It legitimately needs the full CC surface. Let it live where its dependencies are, rather than trying to abstract them away.

This move eliminated the last `collectioncontroller.h` include in any library. Zero library files import the god class. The decoupling is complete.

### Step 7: Confront the Residue

After 11 library extractions, 5 interface definitions, and dozens of file moves, the app shell was down to roughly 48K lines across 197 files. The pathology was concentrated:

- **CollectionController**: Still 66 methods, but now implementing 5 narrow interfaces. The dead code, the redundant APIs, the memory leaks — all still there, but *visible*. No library depends on the internals. They talk to interfaces.

- **CollectionView**: Still 84 includes and a 305-line context menu, but now living in the app shell where it belongs. Its dependency on the full CC surface is explicit and acknowledged rather than hidden behind a library boundary.

- **AppSettings**: Still a fake singleton with dead session state, but the 18 files that used to call its static methods now call `ILocaleSource::global()` instead. The dead code is isolated.

**This is the transformation.** Not from bad code to good code. From *distributed* mess to *concentrated* mess. From "I don't know where to start" to "I have a list."

### Step 8: Clean the Residue

With the mess concentrated:

**Delete the dead code.** 500+ lines of methods with zero callers, unused includes, stub implementations, never-emitted signals, and members that are written but never read. This is pure subtraction. The compiler is your proof that nothing breaks.

**Deduplicate the copy-paste.** The two initialization paths collapse into one. The 12 factory invocations get a builder. The color swatch construction gets a helper. The connect/disconnect mirroring gets automated.

**Fix the semantic bugs.** The undo bug in `removeIncidencesKeepChildren()`. The wrong command type in `ExceptionManagerDialog`. The `QTimer::singleShot` race conditions replaced with proper completion signals.

**Rationalize the APIs.** Three CRUD layers become one. Naming conventions become consistent. Implicit contracts become documented or, better, enforced by types.

**Separate the concerns.** The controller stops showing dialogs. The view stops computing business intelligence. The context menu becomes its own class. The 290-line initialization method becomes 5-6 named methods.

None of these cleanups are hard. They're *possible* because the extraction made them visible and isolated. You can't refactor a 300-line method when you're not sure what 50K lines of library code depend on its exact behavior. You *can* refactor it when you know the libraries don't depend on it at all — because they talk to an interface, not to the method.

---

## The Timeline

The full extraction — 11 libraries, 5 interfaces, dozens of file moves — took approximately two days. Not two weeks. Not two months. Two days.

This speed was possible because:

1. **The conceptual modules were already clear.** The developer knew what the scheduling library was, what the sync engine was, what the tag system was. The code was disorganized, but the concepts were crystallized through months of development.

2. **The extraction is mechanical.** Move files. Update includes. Update CMake. Build. Fix errors. Build again. It's work, but it's not creative work. It doesn't require architectural inspiration. It requires `grep` and patience.

3. **LLMs accelerate the mechanical parts.** Updating 50 include paths, rewriting CMake targets, creating interface headers from grep results — this is exactly the kind of work that LLMs handle well. The developer specified what to do; the LLM did the typing.

4. **Each step is independently verifiable.** Extract one library. Build the library standalone. Build the app. Run tests. If anything breaks, the last extraction is the cause. Revert and try again. The feedback loop is tight.

---

## The Emotional Arc

There's a psychological dimension that nobody talks about.

When you start, you feel overwhelmed. 140K lines. 582 files. A god class with 66 methods and 30 member variables. The mess is everywhere. You can't see the edges.

As you extract the first few libraries, you feel relief. The scheduling library comes out cleanly. The tag system separates in an hour. These were *always* independent — they were just imprisoned in a monolith. Freeing them feels like finding clean clothes at the bottom of a messy closet.

The middle extractions are harder. The sync engine has real dependencies on the god class. You create an interface. You audit call sites. You move files. It's work, but it's *mechanical* work — you can see the progress, measure it, verify it.

And then you reach the end, and something unexpected happens: you look at the residue — the concentrated mess — and instead of despair, you feel *amusement*.

A 216-line dead method that nobody calls! A hash map that leaks memory as a party trick! A controller that shows modal dialogs! A selection system with three competing paths, one of which is dead! An entire session-state subsystem that was superseded and never removed!

The mess is funny because it's small. You can see all of it. You can hold it in your head. It's not a formless dread anymore — it's a specific, enumerable list of specific, fixable problems.

That's the transformation. From overwhelming mess to tractable mess. From "I don't know where to start" to "I have a list." The list is long, but it's finite. And every item on it is now a straightforward fix because the extraction removed all the hidden dependencies that made fixes scary.

---

## The Weight Distribution After Extraction

The numbers tell the story:

### Libraries (92K lines, 385 files)

| Library | Lines | Character |
|---------|-------|-----------|
| calendar-views | 28,757 | Rendering engine, view types, geometry |
| sync | 25,402 | Multi-backend sync, three-way merge, transcoding |
| models | 10,910 | Data models, proxy stack, config, interfaces |
| editor | 6,327 | Modular editor, 11 inspector modules |
| temporal | 5,043 | Temporal focus, filter coordination, scrollbars |
| graph | 3,730 | Interactive PERT graph, Sugiyama layout |
| gantt | 3,151 | Custom Gantt chart, bar rendering, arrows |
| planning | 2,328 | ProjectStore, PlanningEngine, dependencies |
| org-io | 2,187 | Org-mode file I/O, headline conversion |
| tags | 1,975 | Tag system, category manager, color utils |
| scheduling | 1,957 | DAG, CPM, forward scheduler, effort parser |

### App Shell (48K lines, 197 files)

| Module | Lines | Character |
|--------|-------|-----------|
| dialogs | 13,746 | Wizards, settings, recovery |
| widgets | 11,790 | Backend config, sidebar, planning docks |
| views | 6,557 | CollectionView (the legitimate heavyweight) |
| controllers | 4,287 | CollectionController + extracted coordinators |
| app | 3,450 | MainWindow, AppController, AppSettings |
| utils | 2,338 | Date math, sky gradients, drag data |
| sync UI | 1,758 | Conflict resolution dialogs and diff widgets |
| docking | 1,538 | DockManager, layout state, view morphing |
| commands | 1,419 | Thin undo/redo wrappers |
| editor | 804 | Dialog wrapper for library editor |

The gravity is in the libraries: 92K lines (66%) vs 48K in the shell (34%). The two largest libraries — calendar-views (28.8K) and sync (25.4K) — together account for 59% of library code. These are the most complex subsystems, and both are independently buildable.

The app shell is dominated by dialogs (13.7K) and widgets (11.8K) — UI that's too tightly coupled to the app's specific workflows to extract into libraries. This is appropriate. The shell should contain app-specific orchestration. The libraries should contain reusable domain logic.

---

## For the Practitioner: What This Method Requires

**You need:** A compiler (or interpreter, or bundler — whatever builds your project). Grep. Whatever build system you use. And the willingness to make many small, verifiable changes.

**You don't need:** A software architecture degree. A whiteboard session. A design document. A consensus meeting. The architecture is already in your code — you're excavating it, not designing it.

**The skill:** Reading code you didn't write (or don't remember writing). Following import chains. Asking "what does this file actually need?" and verifying the answer with grep. The mindset is archaeology, not architecture. You're not designing a new building — you're excavating the one that was always there.

**The discipline:** Each extraction must be independently verifiable. Move one library at a time. Build after each move. Test after each build. Commit after each success. If something breaks, the last extraction is the cause. Revert and try again.

**The patience:** 11 library extractions, 5 interface definitions, dozens of file moves. Two days. But two *focused* days, where each step is mechanical and verifiable and the progress is visible.

---

[< Previous: The Warning](TransmogrifyingVibeCode_08_Warning.md) | [Next: The Principles >](TransmogrifyingVibeCode_10_Principles.md)
