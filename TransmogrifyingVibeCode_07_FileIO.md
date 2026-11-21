# Chapter 7: The File I/O Story

*Where vibe code got architecture right — because it had to. Some problems force good architecture. You can't vibe-code a CalDAV sync protocol. You can't hand-wave a three-way merge. File I/O punishes sloppiness immediately.*

[< Previous: The Surface](TransmogrifyingVibeCode_06_Surface.md) | [Next: The Warning >](TransmogrifyingVibeCode_08_Warning.md)

---

## The Forcing Function

Chapter 8 will document the mess — the god classes, the dead code, the redundant APIs, the memory leaks, the timers used as architecture. But first, credit where it's due: not everything in PlanStan's architecture is a disaster. The file I/O subsystem is, by any reasonable measure, genuinely good. Well-abstracted. Properly separated. Clean interfaces. Multiple implementations that plug into the same framework without special-casing.

This isn't because the developer was more careful with I/O code. It's because I/O forces good architecture. Networking punishes sloppiness immediately. The first time a sync corrupts data, you learn to be careful. The first time a file format can't represent your data model, you learn to respect the format's constraints. The first time a round-trip loses information, you learn about transcoding fidelity.

The UI layer can tolerate enormous architectural sins because the user can't see the architecture. The I/O layer cannot, because the data either survives or it doesn't, and you know within seconds which one happened.

---

## The I/O Evolution

PlanStan's I/O story spans all six incarnations and culminates in a system that would be respectable in any professional codebase.

### The .kalb Configuration Format

The `.kalb` format emerged in TimeBuster (incarnation 4) and matured through PlanBuster and PlanStan. It's a JSON configuration format with recursive merging — the collection root has a `.kalb` file, and each calendar subdirectory can have its own `.kalb` that overrides or extends the parent. Like `/etc/apt/sources.list.d/` for calendar configuration.

```json
{
  "collectionId": "remote",
  "backend": {
    "type": "caldav",
    "url": "https://example.com/caldav/",
    "username": "user",
    "manualLoad": false,
    "manualSync": false
  },
  "calendars": {
    "Main": {
      "color": "#FF8000",
      "description": "Primary work calendar",
      "accessMode": "readwrite",
      "allowedTypes": ["VEVENT", "VTODO"]
    }
  }
}
```

The `KalbConfigManager` that reads and writes these files is one of the cleanest classes in the codebase. It has a single responsibility, clear inputs and outputs, and predictable behavior. It was clean from the beginning because the problem it solves — recursive configuration merging — has a natural shape that resists bloat. You merge objects. You resolve conflicts with "most specific wins." There's no room for a god class to form.

### ICS via LocalBackend

The local file-based iCalendar backend. `LocalBackend` reads and writes `.ics` files using `KCalendarCore::ICalFormat`. One file per calendar, stored in a subfolder-per-calendar structure. ETags for change detection. Sequential load with explicit cleanup.

At 1,051 lines, `LocalBackend` is compact for what it does: file discovery, calendar creation, item loading, item saving, change tracking, and error handling. The class has a clear lifecycle — discover, load, modify, save — and each phase maps to explicit methods with documented preconditions.

### CalDAV via RemoteBackend

The CalDAV sync was the crucible that forged the developer's entire approach to I/O. The Kalibrator transcripts (February 28, 2025 — the first night of coding) are dominated by the CalDAV fight: KDAV's blocking behavior, threading crashes, lifecycle management nightmares. By 4 AM, the developer had learned the fundamental lesson:

> "Don't mess with the .ics-handling logic. Do not mess with the CalDAV handling logic. Got it? And if we have to make modifications, make damn sure that you understand what it's doing and why it works!"

This commandment — repeated across sessions, across incarnations, across LLM platforms — is the reason the CalDAV code is good. The developer drew a line around it and defended it. Every subsequent coding session was told, explicitly, that this code was sacred. The LLM couldn't casually refactor it, couldn't "improve" it, couldn't touch it without understanding why it worked.

The result: `RemoteBackend` is a solid CalDAV implementation. ETags for conflict detection. Sequential sync with proper cleanup. Error handling that surfaces problems rather than swallowing them. "There is no stupid 'server wins' logic. Every single hiccup is logged and presented."

### Org-mode via OrgBackend

This is where the I/O story gets architecturally interesting.

Org-mode integration started as a simple feature request: let users store tasks in org files alongside their iCalendar calendars. But org-mode files carry semantics that iCalendar can't represent — hierarchy, effort estimates, progress cookies, custom TODO sequences, property drawers, body content. The naive approach (treating org headlines as flat incidences, discarding what doesn't fit) was the initial implementation. It failed, and its failure triggered the paradigm shift documented in Chapter 3.

The fix was architectural: separate the org file I/O from the backend abstraction.

`OrgFileManager` (2,187 lines in `libs/org-io`) handles pure file I/O: parsing org headlines, converting to and from `KCalendarCore::Incidence` objects, preserving round-trip fidelity (property drawers, body content, headline structure). It returns structured results — `OrgHeadlineResult` structs containing pure incidences (zero custom properties) plus separate `OrgPlanningData` and `OrgRoundtripData` for callers to route.

`OrgBackend` (719 lines) is a thin wrapper that delegates to `OrgFileManager` and implements the `SyncBackend` interface. It's as dumb as `LocalBackend` or `RemoteBackend`. It maps org headlines to VTODOs/VEVENTs using only standard iCalendar properties. Planning properties in the org file (EFFORT, BLOCKER, ORDERED) are ignored by OrgBackend — they're not its concern.

`ProjectStore` (in `libs/planning`) is the other consumer of `OrgFileManager`. It reads project org files, populates `PlanningMetadataStore` with effort, blockers, and ordering, and feeds pure incidences into `GlobalIncidenceModel`. It's not a backend. It doesn't participate in sync. It's a separate subsystem that happens to read the same file format.

The org ecosystem — OrgBackend plus OrgFileManager — is 2.4x larger than LocalBackend. This reflects the reality that org-mode carries more semantic weight and needs correspondingly more parsing and preservation logic. The ratio is honest about the complexity difference.

### Multi-backend: The Same Collection, All Formats

The architectural achievement: a single collection can contain calendars backed by different formats simultaneously. Your events live on CalDAV (accessible from your phone). Your tasks live in local ICS files (fast and private). Your projects live in org files (hierarchical, with effort estimates and dependencies). All visible in the same views, all filterable by the same filters, all editing through the same editor.

This works because all backends produce the same output type: `KCalendarCore::Incidence` objects with zero custom properties. The sync engine, the model layer, and the view layer don't know or care which backend produced an incidence. Backend blindness is the architectural principle, and it's enforced by the incidence purity model.

### DecSync: Serverless Sync

For users who don't want a CalDAV server, DecSync provides serverless synchronization through a shared filesystem (Syncthing, network drive, etc.). Another backend, same interface, same data model. Plug it in and it works because the abstraction is correct.

---

## The Paradigm Shift in I/O Terms

The paradigm shift (Chapter 3) is, at its core, an I/O insight. The question "where does the plan live?" has a different answer than "where does the schedule live?"

**The plan** — structure, dependencies, effort estimates, progress — lives in org files. These are the durable artifacts. They survive rescheduling. They survive sync conflicts. They're authoritative.

**The schedule** — which hours on which days the work happens — lives on the calendar (iCalendar events). These are disposable projections. They can be regenerated from the plan at any time. When reality changes, you regenerate the schedule. The plan persists.

The architectural expression:

```
OrgBackend → pure incidences (schedule items on calendar)
ProjectStore → pure incidences (project tasks in model)
              + planning metadata (effort, blockers, ordering)
```

`OrgBackend` produces incidences for the calendar. `ProjectStore` produces incidences for the model AND planning metadata for the planning subsystem. Both use `OrgFileManager` to read the same file format. The split is clean because the file format supports both use cases, and the two consumers extract different information from the same source.

> "OrgBackend produces pure incidences. ProjectStore produces pure planning metadata. The join on UID is the corpus callosum."

The corpus callosum metaphor is precise: two hemispheres (calendar system and planning system) connected by a thin bridge (UID-based references). Each hemisphere is self-contained. The bridge is narrow and explicit. This is what good architecture looks like — not because someone designed it from first principles, but because the I/O constraints forced it.

---

## Incidence Purity: The Architectural Bright Line

The most important architectural decision in PlanStan is also the simplest to state: **zero custom properties on any KCalendarCore::Incidence, anywhere.**

No `X-PLANSTAN-EFFORT`. No `X-ORG-TODO-KEYWORD`. No `X-PROJECTION`. Zero. Every incidence in the system is a pure iCalendar object that any CalDAV server, any phone calendar app, any standards-compliant tool can consume without modification.

This decision was hard-won. Earlier incarnations used custom properties extensively — effort estimates, ordering flags, blocker references, projection markers, all stored as `X-` properties on incidences. These properties worked within PlanStan but were invisible to every other tool. CalDAV servers preserved them (the spec requires it) but phone calendar apps ignored them. Round-trip fidelity was good for iCalendar-to-iCalendar but terrible for org-mode-to-iCalendar, because org's native properties had no iCalendar equivalents.

The clean solution: planning metadata lives in `PlanningMetadataStore`, a UID-keyed store entirely separate from incidences. The store is populated by `ProjectStore` when it reads org files. Planning consumers (Gantt view, dependency graph, forward scheduler, unscheduled dock) read from the store, not from incidences.

| Property | Previous Location | Current Location |
|---|---|---|
| EFFORT | Incidence custom property | PlanningMetadataStore |
| ORDERED | Incidence custom property | PlanningMetadataStore |
| BLOCKER | Incidence custom property | PlanningMetadataStore |
| SIBLING-ORDER | Incidence custom property | PlanningMetadataStore |
| PROJECTION | Incidence custom property | ProjectionManager internal map |
| ORG-TODO-KEYWORD | Incidence custom property | OrgFileManager internal state |

Result: incidences are pure. The iCalendar model is respected. Every tool in the ecosystem can consume PlanStan's data. And the planning metadata is properly scoped to the subsystem that owns it.

---

## The Sync Engine

The sync subsystem (25,402 lines in `libs/sync`, the second-largest library) is PlanStan's most complex I/O component. It includes:

- **Pluggable backends** — local ICS, CalDAV, org-mode, DecSync, holidays, subscriptions, optional Akonadi
- **Three-way diff engine** — `SyncDiff` computes differences between local state, remote state, and common ancestor
- **Transactional sync** — `SyncTransaction` groups changes atomically
- **SyncCoordinator** — orchestrates multi-backend sync
- **SyncWorker** — runs sync in its own thread
- **ConflictManager** — handles conflict workflows via injected `IConflictResolver`
- **ProjectionManager** — projects org planning data into calendar events
- **IncidenceDiff** — property-level diffing for conflict resolution
- **Transcoding framework** — `PropertyTranscoder`, `RRuleTranscoder` for cross-format conversion
- **CalendarManager** — propagates CRUD to all backend bindings

This is production-quality sync infrastructure. Three-way merge, conflict detection, transactional consistency, cross-format transcoding, multi-backend coordination — these are the problems that calendar companies assign to teams and schedule in quarters. The code exists, it works, and it was built through LLM-assisted development.

The sync engine is also where the developer's defensive instincts were strongest. The CalDAV code — earned through painful overnight debugging sessions in February 2025 — was protected from the first day. The three-way diff engine was built with test coverage. The transactional model ensures that sync either completes fully or rolls back. There is, as the manifesto demands, no "server wins" logic.

The sync engine's decoupling from `CollectionController` (via the `ISyncHost` interface) was one of the five interface extractions documented in Chapter 9. The interface has 11 methods — broader than the other four interfaces, reflecting the sync engine's genuinely broad needs. But 11 methods is still far narrower than the 66 methods that `CollectionController` exposes, and it's a legitimate subset: the methods that sync actually calls, discovered by grep, not designed from theory.

---

## The Lesson for Vibe Coders

If your vibe-coded application touches files or network, look at that code first. It's probably the best code in your codebase, because the I/O forced you to get it right.

The I/O layer is the forcing function. It's where:
- Data loss is immediately visible (the file either contains the right data or it doesn't)
- Round-trip fidelity is testable (save, load, compare)
- Error handling is mandatory (networks fail, disks fill, formats mismatch)
- Abstraction boundaries are natural (each backend is a self-contained unit)

The rest of your codebase may be a mess. The I/O code is probably okay. Start your architectural assessment there — not because it needs the most work, but because it's the foundation you can build on. If your I/O abstraction is clean, your extraction (Chapter 9) has solid ground to stand on.

And if your application reads and writes data correctly — if sync works, if files round-trip, if the data integrity is sound — then the rest is cosmetic. Important cosmetics, worth fixing, but cosmetics nonetheless. The data is the product. The I/O is the data. Get the I/O right and you've got a floor to stand on while you fix everything else.

---

[< Previous: The Surface](TransmogrifyingVibeCode_06_Surface.md) | [Next: The Warning >](TransmogrifyingVibeCode_08_Warning.md)
