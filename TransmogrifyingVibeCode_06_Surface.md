# Chapter 6: The Surface

*The UI-first strategy and why it's correct. Humans interact with I/O devices. If it seems to work, nobody worries about architecture. So make it seem to work first.*

[< Previous: The Art of Vibe Coding](TransmogrifyingVibeCode_05_ArtOfVibeCoding.md) | [Next: The File I/O Story >](TransmogrifyingVibeCode_07_FileIO.md)

---

## The Thesis of the Surface

Here is a proposition that will offend every software architect who has ever drawn a box diagram: **the surface is the product.**

Not the architecture. Not the database schema. Not the API contracts. Not the test coverage. The surface — what the user sees, touches, interacts with. The window they open. The buttons they click. The charts they read. The colors, the transitions, the responsiveness, the feel.

If the surface works, the product works. If the surface doesn't work, nothing else matters. No user has ever said "the architecture is clean, so I'll tolerate a terrible UI." Every user has said, implicitly, "this looks right and does what I expect, so I'll use it" — regardless of what the code beneath it looks like.

This is not an argument for sloppy code. It's an argument for *sequencing.* Get the surface right first. Understand what you're building by building the parts that users touch. Then fix the architecture — because now you know what the architecture needs to support.

You don't organize a kitchen before you know what meals you'll cook. You don't architect a codebase before you know what features it needs to deliver. The surface is the discovery mechanism. The architecture follows.

---

## The Surface Timeline

The story of PlanStan's surface is the story of a single extraordinary week: February 2-8, 2026. During that week, the developer — directing LLMs with the demanding precision documented in Chapter 2 — produced a user experience that looks like it was built by a professional design team.

### Day 1-2: Docking (February 2)

The application had been using a basic MDI window layout. The developer wanted docking — panels that float, dock to edges, tab together, resize, and maintain state across sessions. KDDockWidgets, a mature docking framework, was integrated in a single day.

But integration wasn't the whole story. The developer specified seven milestones for the docking implementation:

1. Framework integration and basic dock creation
2. Dock persistence (save/restore layout across sessions)
3. DockManager class to encapsulate all docking logic
4. LayoutStateMachine for managing layout transitions
5. ViewMorphController for semantic zoom — the concept that view panels transform their content based on available space
6. ViewDockWidget as the standard container for all view types
7. SessionManager integration for layout persistence

Seven milestones. One day. Each producing a working, demonstrable state of the application. This is what "demanding boss meets capable LLM" looks like in practice: not a single monolithic feature request, but a structured progression of specific, verifiable steps.

### Day 2-3: The Gantt Chart

The application had been using KGantt, an external Gantt chart library from the KDE Calligra suite. It was functional but limited — the developer wanted custom bar rendering, dependency arrow routing, CPM overlay display, and effort-based rendering modes that KGantt's API couldn't support.

The entire KGantt dependency was replaced with a custom Gantt chart in a single day.

The replacement included:

- **GanttChartWidget** — a QAbstractScrollArea that renders bars, dependency arrows, and CPM overlays
- **GanttView** — combines the chart with a task tree (using the existing HierarchyProxyModel)
- **GanttAxisMapper** — transforms between time coordinates and pixel positions
- **GanttHeaderWidget** — draws the day/week/month ruler
- **GanttArrowRouter** — routes dependency arrows between bars with proper avoidance
- **GanttItemDelegate** — custom rendering with solid/dashed task bars, progress fill, summary brackets, milestone diamonds, and deadline flag markers

This is the kind of custom widget work that causes experienced Qt developers to estimate "two to three weeks." The LLM handled the boilerplate — the scroll area mechanics, the paint event structure, the coordinate transformation math. The developer provided the specification: what a Gantt chart should look like, what interactions it should support, what data it should display.

### Day 3-4: The Dependency Graph

An interactive PERT-style dependency graph with Sugiyama layout. Not a static diagram — a draggable, editable, interactive graph view with:

- **DependencyGraphView** — a QGraphicsScene wrapper
- **GraphNodeItem** — draggable node representations
- **DependencyEdgeItem** — directed edges between nodes
- **ConnectorItem** — handles for creating new edges by drag
- **GraphMinimap** — an overview widget showing the full graph at reduced scale
- **GraphLayout** — automatic Sugiyama layout (layer assignment, crossing minimization, coordinate assignment)
- **Tool system** — SelectMoveTool for dragging, CreateEdgeTool for edge creation
- **GraphCommands** — undo support for all graph operations

Sugiyama layout is a non-trivial graph algorithm. It involves cycle removal, topological layering, node ordering within layers to minimize edge crossings, and coordinate assignment to minimize edge lengths. The implementation is correct — the graph renders properly, nodes are positioned sensibly, and edges route cleanly.

### Day 4-5: Calendar Views

The calendar view system expanded to include:

- **Year view** — a compact, 12-month overview with event indicators
- **Range agenda** — an adaptive day-by-day agenda that tiles days based on available width
- **Project dashboard** — card-based project summaries with progress indicators
- **Multi-month tiling** — multiple months visible simultaneously, with semantic zoom controlling density

Each view type plugged into the existing ViewInterface architecture. Each supported the same data model, the same filter system, the same selection protocol. The factory pattern that would later be identified as the "12 copies of the same 7 arguments" anti-pattern (Chapter 8) was born here — and it worked perfectly, even if the code was ugly.

### Day 5-6: Renderers and Theming

The ViewPalette theming system arrived:

- **StandardIncidenceRenderer** — the default block-style calendar item rendering
- **BackgroundShadingRenderer** — subtle background coloring instead of block rendering
- **HandwrittenRenderer** — a hand-drawn aesthetic for calendar items
- **LineRenderer** — minimal line-based rendering
- **RendererRegistry** — pluggable renderer selection
- **StyleResolver** — cascading style resolution from global defaults through collection, calendar, and per-item overrides
- **TextLayoutEngine** — consistent text layout across all renderers
- **PlanningColorScheme** — semantic coloring for planning states

Dark mode support. Multiple rendering styles selectable per view. Style cascading — global settings overridden by collection settings, overridden by calendar settings, overridden by per-item settings. The full CSS-like cascade, applied to calendar rendering.

### Day 6-7: Spreadsheet View

A spreadsheet-style task view with:

- **TaskTreeView** — tree/table hybrid with sortable columns
- **SpreadsheetItemDelegate** — base delegate for spreadsheet cells
- Six specialized delegate types: Duration, DateTime, Predecessor, Progress, Enum, CheckState
- Inline editing, column resizing, column visibility toggles

### The Result

By February 8, the application's surface was comprehensive. A user opening PlanStan saw:

A docked interface with floating panels, a Gantt chart with dependency arrows and CPM overlays, an interactive dependency graph with a minimap, multiple calendar view types (month, week, year, agenda, multi-month, range agenda, dashboard), a spreadsheet task view, a modular editor with 11 inspector modules, a theming system with multiple renderers, and dark mode.

Nobody would guess this was built by one person who hadn't coded in 15 years. And that's the point.

---

## Why Surface-First is Correct

The traditional software engineering sequence is: design the architecture, implement the data layer, build the business logic, and then add the UI on top. This is correct for teams building systems with well-understood requirements. It is wrong for a single developer building a new kind of application through discovery.

The surface-first approach works because:

**The surface is the specification.** When you build the Gantt chart, you discover what data the Gantt chart needs. When you build the dependency graph, you discover what graph operations need undo support. When you build the spreadsheet view, you discover that six different cell types need six different delegates. The UI tells you what the architecture needs to support. You can't know this in advance — you discover it by building.

**The surface provides feedback.** A completed surface can be evaluated immediately: does it look right? Does it feel right? Does it do what the user needs? Architecture can't be evaluated this way — you can only evaluate architecture by trying to change it, which doesn't happen until development is well underway. The surface gives you feedback on day 1.

**The surface motivates.** For a solo developer — especially one with ADHD — visible progress is fuel. A Gantt chart on screen is motivating. A well-refactored abstract base class is not. The surface keeps the developer engaged during the long slog of building 140K lines.

**The surface reveals architectural needs.** PlanStan's five interfaces (`ICommandDispatcher`, `IEditorServices`, `IViewServices`, `ISyncHost`, `ILocaleSource`) were not designed in advance. They were discovered by building the surface and then asking: what does this view need from the system? What's the narrowest contract that supports this UI? The surface drove the architecture, not the reverse.

**The surface tolerates architectural mess.** This is the uncomfortable truth: the god class, the dead code, the redundant APIs, the `QTimer::singleShot` prayers — none of them affected the user experience. The surface was immaculate even when the architecture was a disaster. This means the developer could build features at maximum speed, producing a working product, and defer architectural cleanup to a time when they understood the full scope of what they'd built.

The deferral wasn't laziness. It was pragmatism. You don't organize a kitchen before you know what meals you'll cook. The developer cooked every meal — Gantt charts, dependency graphs, sync engines, scheduling algorithms — and then organized the kitchen. The organization was better for having been deferred, because it was informed by the full reality of the codebase rather than by advance speculation about what the codebase might become.

---

## The Demanding Boss and the Surface

The developer's demanding personality (Chapter 2) was most visible in surface work. This is where taste operates at maximum leverage: a Gantt bar that's the wrong shade, a context menu that's organized illogically, a scrollbar that doesn't feel right — these are the details that separate a polished product from a competent one.

Quotes from the development sessions reveal the granularity of the demands:

"I want a floating navigator with watermark months." Not "add a navigator" — a specific visual treatment with a specific design element.

"The temporal scrollbar needs cursors and handle drag." Not "add a scrollbar" — a scrollbar with specific interaction behaviors.

"Add a minimap to the graph view." A specific feature, specified concisely, producing a specific result.

Each demand was precise enough to generate a specific implementation and evaluable enough to be accepted or rejected immediately. The developer didn't say "make it better" — they said exactly what they wanted. This is the skill that makes vibe coding produce results rather than mush: the ability to specify at the right level of detail. Too vague and the LLM produces something generic. Too detailed and you might as well write it yourself. The developer found the sweet spot: specific enough to guide, general enough to let the LLM's implementation strength shine.

---

## The Surface as Evidence

For skeptics who doubt that LLM-assisted development can produce real software: look at the surface.

A custom Gantt chart with CPM overlay, dependency arrows, effort-based rendering, summary brackets, milestone diamonds, and deadline flags. An interactive PERT graph with Sugiyama layout, edge creation, and a minimap. A docking system with semantic zoom. Multiple calendar renderers. A spreadsheet view with six delegate types. A theming system with cascading style resolution.

This is not a demo. This is not a proof of concept. This is a professional-grade application surface built in a week by one person directing LLMs. The code beneath it needed cleanup (the next three chapters document this in loving detail). But the surface — the thing that users interact with, the thing that makes the application useful — was solid from the moment it was built.

The surface speaks. Let it.

---

## A Note on Sequence

This chapter comes before the architecture chapters (7-9) for a reason. The chronicle follows the actual sequence of development: surface first, architecture later. This is not how the story is usually told in software engineering. The usual narrative starts with architecture and treats the UI as decoration.

PlanStan's narrative starts with the surface and treats the architecture as infrastructure that makes the surface sustainable. The surface is why the application exists. The architecture is how the application survives.

Both matter. But the surface came first, and the surface is what the user takes home.

---

[< Previous: The Art of Vibe Coding](TransmogrifyingVibeCode_05_ArtOfVibeCoding.md) | [Next: The File I/O Story >](TransmogrifyingVibeCode_07_FileIO.md)
