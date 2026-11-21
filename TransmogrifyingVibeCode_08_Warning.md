# Chapter 8: The Warning

*What lives beneath the surface. The forensic analysis. Not self-flagellation — celebration of honesty. Every vibe-coded application has this beneath the surface. Here's what ours looked like.*

[< Previous: The File I/O Story](TransmogrifyingVibeCode_07_FileIO.md) | [Next: The Method >](TransmogrifyingVibeCode_09_Method.md)

---

## The Dark Comedy

Let us now turn our attention from the things that work to the things that are hilarious.

After 11 library extractions and 5 interface definitions (Chapter 9), the app shell was down to roughly 48K lines across 197 files. It contained three classes of particular archaeological interest: `CollectionController`, `CollectionView`, and `AppSettings`. Each carries the full sedimentary record of every midnight hack, every "I'll fix it later," every copy-paste that became load-bearing, and every architectural assumption from a stage of development that no longer exists.

This chapter is not an exercise in shame. It's a field guide. Because every vibe-coded application that succeeds has a version of this beneath its polished surface, and the first step toward fixing it is being able to look at it without flinching. Ideally, with amusement.

---

## I. CollectionController: The God That Forgot What It Created

**535 lines of header. ~2,578 lines of implementation. 66 methods. 30 member variables. 17 nullable pointers. Zero lifecycle documentation.**

### The Plastic Plant

CollectionController has two initialization methods: `loadCollectionFromFile()` (290 lines) and `initialize()` (216 lines). They do almost exactly the same thing. They share approximately 150 lines of verbatim copy-pasted code — identical lambdas, identical object creation sequences, identical signal wiring.

The differences are subtle and, in the way of horror movies, *almost worse* for being subtle. One sets up sync infrastructure. The other doesn't. One batches discovery tracking. The other starts discovery immediately. One processes deferred calendar creations. The other just... doesn't.

`initialize()` has zero callers. Not one. Anywhere. In the entire project. It is 216 lines of lovingly maintained dead code — maintained because someone clearly kept it in sync with `loadCollectionFromFile()` over time, updating both methods whenever features changed, all for a function that nobody calls.

It's like watering a plastic plant. Every week. For months.

### Three Ways to Do the Same Thing

There are three separate API layers for modifying incidences:

| Layer | Character | Problem |
|-------|-----------|---------|
| **Old API** (`addIncidence`, `updateIncidence`) | "I was here first" | `addIncidence` has zero external callers. Dead. |
| **Unified dispatch** (`pushAddCommand`, `pushUpdateCommand`) | "I'm the correct one" | Routes to project-aware commands. This is what callers should use. |
| **Low-level mutation** (`applyIncidenceAddition`, etc.) | "I don't believe in undo" | Direct model mutation with a "safety net" comment that says "I know this is wrong but it hasn't crashed yet." |

The punchline: `ExceptionManagerDialog` still calls the Old API's `updateIncidence()`, which means it creates the wrong command type for project tasks. It's been doing this silently. Nobody noticed. The old API is a booby trap disguised as a deprecated method.

### The Memory Leak That Thinks It's a HashMap

`m_pendingFetches` is a `QHash<QString, FetchOperation*>` that stores pointers to fetch operations. These pointers are inserted. They are removed. They are *never dereferenced* — not for cancellation, not for progress, not for status, not for anything. They are stored purely so the hash can answer "does this key exist?" — which is exactly what `m_calendarsLoading` (a `QSet<QString>` with the same keys) already does.

The `FetchOperation*` pointers are also never deleted. They leak. Every. Single. Fetch.

Three containers. Same keys. One of them leaks memory as a party trick.

### A Controller That Shows Modal Dialogs

`initializeSyncInfrastructure()` — a method on a *controller* — directly calls `dialog.exec()` on a `TransactionRecoveryDialog`. A modal, blocking, GUI dialog. In a controller. During initialization.

"But it's for recovery!" Sure. And the dentist's office is technically a restaurant because they have a waiting room with magazines.

### The Signal Ventriloquist

Three times in the codebase, CollectionController does this:

```cpp
emit m_kalbConfigManager->settingsChanged();
```

That's the controller emitting a signal *on behalf of another object*. Anyone listening to `settingsChanged()` on the config manager will receive emissions that the config manager didn't send. It's like someone else tweeting from your account. This is a Qt anti-pattern so severe it should come with a warning label.

### The 500-Millisecond Prayer

```cpp
QTimer::singleShot(500, this, [this]() { runSync(); });
```

A 500ms delay to "allow the calendar to be set up." This is not engineering. This is *hope*. On a fast machine, you're wasting half a second. On a slow machine, you're racing the universe. The software equivalent of closing your eyes, counting to three, and assuming the traffic light changed.

### The Undo Bug That Undoes Wrong

`removeIncidencesKeepChildren()` reparents children to their grandparent, then deletes the selected items. These are separate undo stack entries. If you undo the deletion, the items come back — but the children stay reparented. The "keep children" operation is partially irreversible. It's Ctrl+Z with an asterisk.

### The Dead Code Graveyard

| Deceased | Cause of Death | Lines |
|----------|---------------|-------|
| `initialize()` | Zero callers. Nobody knows it exists. | ~216 |
| `stageIncidenceModification()` | Zero callers. Born dead. | ~48 |
| `addIncidence()` | Superseded by `pushAddCommand()`. | ~10 |
| `onBackendSyncCompleted()` | Body is literally `Q_UNUSED` and a comment: "implement behavior here." | 4 |
| `m_useOrgBackend` + setter | Flag never read, setter never called. | — |
| `calendarsForBackend()` | Private helper with zero callers. | ~17 |

Conservative estimate: 400+ lines that do absolutely nothing, maintained across refactors, compiled every build.

---

## II. CollectionView: Five Jobs in a Trenchcoat

**238 lines of header. 2,428 lines of implementation. 84 includes. Named "View." Is not a view.**

CollectionView is simultaneously:
1. A View (renders nothing itself, but sure, the name's right there)
2. A Controller (makes business decisions, orchestrates deletion, manages undo)
3. A Mediator (routes signals between 10+ widgets)
4. A Factory (constructs 12+ subwindow types)
5. A Layout Manager (creates and wires 8 dock widgets)

It's the Swiss Army knife of classes, except every tool is a slightly different knife and they're all a little rusty.

### Three Selection Paths Walk Into a Bar

The `m_currentSelection` member is mutated from three different places:

1. **`onItemsSelected`** — Sets selection. Doesn't emit `currentSelectionChanged`. Is never connected to anything. *Is dead code.* The bartender asks why it's even here. It shrugs.

2. **`onProactiveViewItemsSelected`** — Sets selection. Emits `currentSelectionChanged`. Calls `updateSelectionDependentActions()`. This is the correct one.

3. **A lambda in `setupToolViews`** — Sets selection. Emits `currentSelectionChanged`. Does NOT call `updateSelectionDependentActions()`. It got the signal right but forgot the side effect.

Your selection-dependent toolbar buttons may or may not update depending on *where* you clicked. Latent bug factory, open 24/7.

### The 305-Line Context Menu

`onItemContextMenuRequested` is 305 lines long. One method. It builds menus, queries domain state, checks incidence types, constructs color swatch pixmaps, creates undo commands, manages undo macros, and creates submenus with per-project liquidation options.

If this method were a restaurant, it would seat you, take your order, cook your food, serve it, wash the dishes, do the taxes, and then ask if you'd like dessert.

### The Copy-Paste Ouroboros

`openViewType()` is 202 lines of factory method with an `if/else if` chain for known view types, followed by a `switch` statement on `viewType.implementation`. Both halves construct subwindows with the exact same 7-argument parameter tuple, repeated verbatim 12+ times:

```cpp
m_collectionController->globalModel(),
m_collectionController,
collection,
m_temporalFocusManager,
m_filterManager,
m_collectionController->templateManager(),
nullptr
```

Twelve times. The same seven arguments. Including `nullptr`. If the constructor signature ever changes, you get to update it in 12+ places.

### 84 Includes

The `.cpp` file has eighty-four `#include` directives. A well-factored Qt class has 10-20. If `#include` directives were friends on social media, this class would be a small-town mayor who accepts every request.

### Live Test Code in Production

`runSyncBackendTests()` will actually push test data to a CalDAV server. The remote path is live. It creates dummy events in every calendar of the collection. There are no safeguards. This is a loaded gun in a drawer labeled "Utilities."

---

## III. AppSettings: The Singleton That Can't Let Go

### Dead Session State

`SessionManager` was extracted to handle session persistence. Good. Clean. Modern.

Meanwhile, in `AppSettings`, the following methods continue to exist: `windowGeometry()`, `setWindowGeometry()`, `windowMaximized()`, `setWindowMaximized()`, `dockLayout()`, `setDockLayout()`, `loadSession()`, `saveSession()`.

They are all dead. Zero external callers. They read and write to a config file called `planstanlite_session` that nothing else reads from. It's a tree falling in a forest with no one around, except the tree is also writing to disk.

The `loadSession()` method contains this gem:

```cpp
if (windowGroup.hasKey("Geometry")) {
    QByteArray geomData = QByteArray::fromBase64(
        windowGroup.readEntry("Geometry", QByteArray()));
    // Decode geometry from base64 if it was stored as such
    // For now, store as separate x/y/w/h values
}
```

It reads data. Assigns it to a local variable. Does nothing with it. The comment says "for now." The "now" in question was approximately never.

### Static Methods That Don't Need the Instance

The static methods `effectiveLocale()`, `effectiveFirstDayOfWeek()`, etc. all check if the singleton exists, then access `PlanStanLiteSettings` — which is a *different singleton that doesn't need AppSettings at all*. The null check is protecting against the app not being started yet. Which is when nobody would be calling these anyway. It's a safety rail on a bridge over solid ground.

### Every Setter Writes to Disk

Every. Single. One.

```cpp
void AppSettings::setCalendarLocale(const QString &locale) {
    PlanStanLiteSettings::setCalendarLocale(locale);
    PlanStanLiteSettings::self()->save();
}
```

Change 5 settings in a row? Five disk writes. Open the settings dialog and adjust preferences? Every slider movement, every checkbox toggle — straight to disk. No batching. No dirty flag. Just raw, unfiltered I/O enthusiasm.

### The Gulf of Guinea Bug

```cpp
bool valid = (lat != 0.0 || lon != 0.0);
```

A location at (0.0, 0.0) — the Gulf of Guinea, where the equator meets the prime meridian — is reported as invalid. If you live on a buoy at the intersection of the world's two most important imaginary lines, PlanStan will not acknowledge your existence.

---

## IV. The Unified Theory of Vibe Code Decay

Across all three classes, the same patterns recur like a chorus. These are the universal failure modes of vibe-coded software, and if you're a vibe coder reading this, you will recognize them in your own codebase.

### 1. Accretion Without Deletion

New features are added alongside old ones. Nobody deletes the old ones. `initialize()` was never removed when `loadCollectionFromFile()` took over. Session state code was never removed when `SessionManager` arrived. `onItemsSelected` was never removed when `onProactiveViewItemsSelected` replaced it.

The codebase grows like a coral reef — always adding, never subtracting. LLMs are particularly prone to this because they don't carry context about what's been superseded. Each session adds new code. No session removes old code, because the LLM doesn't know the old code exists unless you tell it.

### 2. Copy-Paste as Architecture

Instead of extracting shared logic, methods are duplicated with small variations. Two initialization paths sharing 150 lines. Two collection-creation flows sharing 40 lines. Twelve factory invocations with the same 7 arguments.

LLMs default to copy-paste because it's the path of least resistance. Extracting a shared method requires understanding the full context of both call sites. Copying requires understanding only the immediate need. When the developer is moving fast — 60 commits per day — the copy is always faster than the extraction.

### 3. Implicit Contracts

Which of the three CRUD layers should a caller use? Which of the 17 nullable members are valid at any given lifecycle stage? When is it safe to call `calendarsForBackend()` versus `backendForCalendar()`?

Nothing is documented. The contracts exist only in the developer's head, and that developer is you six months ago, and that person is dead to you now.

### 4. QTimer::singleShot as Load-Bearing Infrastructure

Four instances across the codebase, all papering over initialization ordering bugs. The code doesn't have a timing model. It has a *prayer* model.

This pattern is particularly dangerous because it usually works — the timing is usually right, the data is usually ready, the race condition is usually won. It fails rarely and intermittently, making the bugs nearly impossible to reproduce. The 500ms timer works on your machine. It fails on the CI server. It fails when the network is slow. It fails when the system is under load. And when it fails, nothing in the error output tells you that a timer-based assumption was violated.

### 5. UI and Business Logic in a Blender

Controllers show dialogs. Views compute business intelligence. Context menus create undo commands. The architectural layers aren't so much violated as *ignored*, like speed limit signs on a highway where everyone does 90.

---

## The Warning

This is the warning to every vibe coder who has built something that works:

**All of this worked.** Users never saw any of it. The surface — the Gantt chart, the dependency graph, the calendar views, the docking, the theming — was impeccable. The features were real. The application was useful. If you judged the software by its surface, it was professional-grade.

That is the danger. The mess is invisible until you try to change something deep. Until you try to add a feature that touches the god class. Until you try to refactor the 305-line method. Until you try to understand why the selection sometimes doesn't update. Until the 500ms timer loses its race on someone else's machine.

The mess is not unusual. Professional codebases develop the same patterns. The difference is that professional teams expect this and schedule cleanup cycles. The solo vibe coder encounters it for the first time and reads it as personal failure.

It's not personal failure. It's organic growth. It's the normal, predictable consequence of building software feature by feature, session by session, without a global architectural view. And it has a normal, predictable fix.

That fix is the subject of the next chapter.

---

[< Previous: The File I/O Story](TransmogrifyingVibeCode_07_FileIO.md) | [Next: The Method >](TransmogrifyingVibeCode_09_Method.md)
