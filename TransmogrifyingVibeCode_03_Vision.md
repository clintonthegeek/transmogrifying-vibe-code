# Chapter 3: The Vision

*What was being built, and why. The Palm Pilot nostalgia, the ADHD insight, the plan-first paradigm, and the radical proposition that good time management software doesn't manage time — it manages attention.*

[< Previous: The Developer](TransmogrifyingVibeCode_02_Developer.md) | [Next: The Tumbleweed >](TransmogrifyingVibeCode_04_Tumbleweed.md)

---

## The Palm Pilot Feeling

Before the iPhone, before Google Calendar, before the entire category of "productivity apps" became a multi-billion-dollar industry of beautiful, useless software, there was a handheld device that organized a life.

The Palm Pilot didn't have apps. It had four buttons: Calendar, Contacts, To Do, Memo Pad. You pressed a button and you were there. You wrote with a stylus in a simplified alphabet that took an afternoon to learn and then became invisible. The device fit in a shirt pocket. The battery lasted weeks. And the system worked — not because it was sophisticated, but because it was opinionated. It made decisions for you. Calendar is here. Tasks are here. Notes are here. There is no configuration. There are no themes. There are no plugins. There is a system, and the system works.

The developer behind PlanStan carried the memory of that feeling for two decades. Not the specific technology — stylus input and a 160x160 pixel screen are nobody's nostalgia — but the *experience* of having a single trusted system that organized a life. Everything in one place. Everything accessible. Everything simple enough to use without thinking.

The modern productivity landscape is the opposite of that feeling. We have Notion (infinite flexibility, paralysis of choice, and an industry of YouTube videos explaining how to set up Notion). We have Todoist (pretty, competent, structurally incapable of representing project dependencies). We have Apple Reminders (a grocery list that thinks it's a task manager). We have Google Calendar (a schedule that thinks it's a plan). We have Obsidian (a knowledge base that thinks it's everything). We have a hundred apps, each excellent at one thing, none capable of the Palm Pilot feeling: one place, one system, everything handled.

The vision for PlanStan began in that gap. Not "another productivity app" — a system. An opinionated, complete system for someone who wants to manage their projects, their time, and their attention in one place, on their own machine, without surrendering their data to a cloud service that will pivot, enshittify, or shut down.

---

## The ADHD Insight

But the vision went deeper than nostalgia. It went into neurology.

The developer has ADHD. Not the pop-culture "ooh, squirrel!" version. The real kind: the executive function disorder that makes it physically impossible to start a task you genuinely want to do, that turns every undecided commitment into a psychic tax on an already overtaxed working memory, that produces 60-commit days and 6-month gaps with equal frequency and zero predictability.

David Allen's Getting Things Done framework offers a tantalizing promise to the ADHD brain: externalize everything. Get every commitment out of your head into a trusted system. Make front-end decisions about what each thing means. Then your mind can release it.

Allen writes: *"Your mind is for having ideas, not holding them."*

For neurotypical people, this is a productivity tip. For ADHD brains, it's medicine. The problem is the delivery mechanism.

Pure GTD requires regular weekly reviews (two hours of sustained administrative focus), processing every item in your inbox to zero (an executive function marathon), maintaining multiple context-sorted lists (remembering which list to check), and consistent daily engagement (habit formation). Every one of these is an executive function task. GTD is built by and for people whose executive function works. It offloads *content* memory but still relies on *procedural* executive function for system maintenance.

The ADHD person tries GTD, feels amazing for 72 hours, then the weekly review slips, the inbox piles up, the lists go stale, and they conclude — again — that they're fundamentally broken.

The developer's insight was specific and architectural:

> "They don't need another app that guilt-trips them with red overdue badges. They need a system that thinks the way their brain already wants to think — in bursts, in projects, in 'what's the one thing I can do right now?' — and that quietly handles the bookkeeping their working memory can't."

The ADHD working memory is already overtaxed. Every undecided commitment isn't just occupying RAM — it's consuming the *entire* limited allocation. The result: paralysis, shame spirals, and the peculiar ADHD experience of lying on the couch unable to do the thing you genuinely want to do, knowing you should do it, physically unable to start.

PlanStan was designed, from the ground up, to be the executive function prosthetic that makes GTD's principles accessible to brains that can't maintain the system manually. Not a list app with pretty colors. A system that captures with minimal friction, makes decisions visible rather than requiring recall, structures time without demanding willpower, connects daily grind to long-term vision, and degrades gracefully when the user inevitably falls off the wagon.

---

## The Manifesto

By the third incarnation (TimeBuster, March 2025), the philosophy had crystallized into prose that reads like a founding document:

> "This is an all-purpose IDE for time management... for power-users who aren't afraid to get their hands dirty, and who want full transparency and control over their schedule and tasks."

The metaphor is deliberate and revealing. Not a "calendar app." Not a "task manager." An *IDE* — an Integrated Development Environment. The way a programmer uses an IDE to manage the complexity of a codebase — multiple files, build systems, debuggers, version control, all integrated into a single interface — the developer envisioned PlanStan as the tool for managing the complexity of a *life*.

The principles that emerged were fierce:

**Local-first.** "This is a tool for personal computer owners to own their personal data on their own personal computer that they own." Not a cloud service. Not a SaaS product. Your data lives on your machine, in open formats (iCalendar, org-mode), readable by any text editor. Remote servers are for sync convenience, not authority.

**No "server wins" logic.** "There is no stupid 'server wins' logic. Every single hiccup is logged and presented." When your phone calendar disagrees with your desktop calendar, you don't lose data silently. Every conflict is surfaced. Every resolution requires explicit approval. The user is always in control.

**No autosave, no inference, no telemetry.** Explicit user control over all operations. The application does what you tell it to do, nothing more. It doesn't guess what you meant. It doesn't quietly "improve" your data. It doesn't phone home.

**Detailed diffs.** Not UNIX diffs — scheduling-appropriate comparisons. When a sync conflict arises, you see exactly which properties changed on which items, in terms that make sense for calendar data.

These aren't just preferences. They're a political stance: in an era of cloud services that treat user data as a monetizable resource, PlanStan asserts that your schedule, your projects, your plans, and your time are *yours*. The application is a tool, not a service. It has no interests of its own.

---

## Programming the User

The deeper layer of the vision goes beyond features into pedagogy. PlanStan doesn't just organize data — it teaches organizational thinking.

The developer studied how other software transforms its users:

**Superhuman** teaches Inbox Zero not by explaining it but by making the product's mechanics embody it. Auto-advance after archiving removes the decision pause. The onboarding establishes the clean-inbox state before the user forms an opinion about it. Game *design*, not gamification — intrinsically satisfying interactions, not extrinsic reward systems.

**Bullet Journal** uses handwriting's slowness as a pedagogical tool. Monthly migration — physically rewriting every incomplete task — forces evaluation. If a task isn't worth rewriting, it isn't worth doing. The friction is the feature.

**Sunsama** decomposes "plan your day" into a structured multi-step ritual: gather tasks, estimate durations, observe workload against available time, trim to fit. The repetition teaches the component skills of planning.

**Super Mario Bros** teaches every core mechanic in World 1-1 without a single line of text. Each screen introduces exactly one new element. The level geometry constrains the player into discovering mechanics through interaction.

The developer's insight: PlanStan should teach time management the way Mario teaches platforming — through interaction, not instruction. The software's structure is the curriculum:

- The **Week Agenda** at startup puts the capture surface in front of you immediately. You don't have to remember to check an inbox. The software IS the inbox.
- The **Effort module** in the editor forces estimation: "How long will this actually take?" If you can't answer, the task isn't clarified yet.
- The **Dependency module** forces sequencing: "What has to happen before this?" The Gantt view draws the arrows. You can *see* the structure instead of holding it in your head.
- The **Forward Scheduler** eliminates decision paralysis: "What should I do next?" has a computed answer. The ADHD user opens the app, sees today's tasks, and starts working.
- The **Unscheduled dock** makes review ambient: every time you open the app, you see the state of your world without setting aside two hours for a formal GTD review.

The critical design choice: the ADHD user never has to look at the Gantt view. They can live entirely in Week Agenda and the Unscheduled dock. The project structure exists underneath, supporting the scheduling, but the user interacts only with "what's next?" and "what's today?"

But when they want to — when they have a clear-headed moment — the Gantt is there. The full project structure, the critical path, the timeline. A single continuous line from the five-year vision through the current project through today's task. The ADHD brain's fuel source — the emotional salience of connecting daily drudgery to a compelling future — made visible.

---

## The Paradigm Shift

The most important architectural insight came late, and it changed everything.

PlanStan began as a calendar application. The assumptions were straightforward: flat incidences (events and tasks as independent items), calendar-centric storage (ICS files and CalDAV), and schedule-first thinking (the calendar *is* the plan).

This broke when users tried to plan projects. Three test failures exposed the problem:

- Org-mode date ranges don't map to single DTSTART/DTEND pairs
- Org's progress cookies (`[2/5]`) don't map to iCalendar's 0-100 percent complete
- The EFFORT property — the planning system's most important data point — has no iCalendar equivalent

These weren't parsing bugs. They were symptoms of a fundamental mismatch: the iCalendar data model cannot represent project planning. Effort estimates, dependency links, hierarchical task decomposition, computed progress — all the things that make project management possible — were being discarded because the model had no room for them.

The paradigm shift was a single sentence:

> **"The plan is authoritative. The schedule is disposable."**

The plan — structure, dependencies, effort estimates, progress — lives in org files. The schedule — which hours on which days the work actually happens — lives on the calendar. When reality changes (and it always does), you regenerate the schedule from the plan. The plan evolves deliberately. The schedule adapts automatically.

This inversion — from schedule-first to plan-first — resolved the tension between GTD's "next action" focus and project management's "all steps" view. Traditional project management thinks in terms of all steps: define the work breakdown structure, estimate everything, sequence all dependencies, compute the critical path. GTD thinks in terms of the next step: what's the very next physical action? Don't worry about step 47.

PlanStan maintains both views simultaneously. The Gantt shows all steps. The Unscheduled dock shows next steps. The Week Agenda shows today's steps. The user can operate at whatever level of detail their current cognitive state supports. Structure all, show next.

The architectural expression of this insight was the "split-brain" model:

> "OrgBackend produces pure incidences. ProjectStore produces pure planning metadata. The join on UID is the corpus callosum."

No custom properties on any KCalendarCore::Incidence, anywhere. Zero. The iCalendar data model remains pristine — a pure event/task system that any CalDAV server or phone calendar can consume. The planning metadata — effort, blockers, ordering, dependencies — lives in a separate UID-keyed store that's populated from org files. The two halves communicate through UID references, like a brain's two hemispheres connected by a corpus callosum.

This is real architecture. Not vibe-coded spaghetti — a genuine, thought-through separation of concerns with clean interfaces and explicit data flow. It emerged from a developer who couldn't have drawn a UML diagram but could articulate, in precise prose, exactly how the data should flow and why.

---

## Free Time as a Resource

The scheduling insight that GTD misses — and that PlanStan's forward scheduler captures — is that free time is a finite resource to be allocated, like money in a budget.

Allen explicitly avoids time-based scheduling for most tasks. His calendar is reserved for "hard landscape" — appointments, deadlines, day-specific actions. Everything else goes on context-sorted "as soon as possible" lists. This works for neurotypical people who can trust themselves to scan the list and choose well. ADHD users can't. They need external temporal structure. But pure time-blocking fails too — it creates a rigid schedule that shatters on first contact with reality.

PlanStan's forward scheduler computes the synthesis: available working hours per day (configurable — 120 minutes by default, because this is a personal project tool, not a corporate PM system), which days are working days, what tasks need doing, what order they must happen in, and how to fit them into available time slots. It's Allen's "hard landscape" creation, automated.

The user defines the work — projects, tasks, effort, dependencies. The scheduler fills their available time. The result appears on the Week Agenda as concrete time blocks that were derived from the project structure, not invented by willpower.

The Gantt view makes this visible: solid bars for scheduled work, ghost bars for unscheduled work, gaps for available time. The user can literally see their free time being consumed by their commitments. If there's no free time left and unscheduled tasks remain, they're overcommitted. This is Allen's "complete inventory of commitments" made visual and temporal.

---

## The Deeper Vision

Beneath the features, beneath the paradigm shift, beneath the ADHD framework, there's a proposition so radical that most productivity software can't even frame it:

**Good time management software doesn't manage time. It manages attention.**

The ADHD brain runs on interest and urgency, not importance and obligation. Without a compelling vision pulling them forward, the ground-level work is meaningless drudgery. "Process your inbox" is not motivating. "Build the web app that lets me quit my job" IS motivating.

PlanStan's critical path draws a literal red line from the five-year vision through the current project through today's task. The ADHD user doesn't have to hold the alignment in their head. They can see it. The software provides the emotional salience — the feeling that this matters, that it connects to something real — that the ADHD brain needs to activate.

And when the ADHD user inevitably falls off the wagon — when the weekly review slips, when the inbox piles up, when the 6-month gap opens up — the system degrades gracefully. The data is still there. The projects are still structured. The plans are still authoritative. You can come back, re-run the forward scheduler, and pick up where you left off. The system doesn't punish absence. It doesn't guilt-trip with red overdue badges. It waits.

This is the vision that drove 768 commits, six incarnations, and 13 months of development. Not "a better calendar app." A system that thinks the way the developer's brain actually works, built by that brain, for that brain, using tools that didn't exist two years ago.

The vision was always clear. The architecture took longer to arrive. The next chapter tells the story of the six attempts it took to get there.

---

[< Previous: The Developer](TransmogrifyingVibeCode_02_Developer.md) | [Next: The Tumbleweed >](TransmogrifyingVibeCode_04_Tumbleweed.md)
