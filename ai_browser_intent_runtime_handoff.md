# AI Browser / Intent Runtime — Complete Architectural Handoff

**Purpose:** Transfer the full conceptual development of the preceding AI-runtime discussion to a new assistant (e.g. Gemini/Claude) without losing architectural context, terminology, or design rationale.

**Audience:** A technically sophisticated AI assistant and the original author.

**Status:** Conceptual architecture / research and prototyping direction. Some claims about existing products, models, browser engines, and ecosystem integrations are contextual rather than requirements.

---

## 0. Executive Thesis

The conversation evolved from a question about **recreated/simulated fruit-fly brains** into a much broader systems thesis:

> **The next useful abstraction for AI may not be a larger language model, but an intelligence runtime that treats models as interchangeable compute engines inside a persistent, event-driven, capability-oriented environment.**

The strongest final formulation was:

> **An intelligence runtime is to intent what the browser is to documents: a persistent execution environment that manages state, context, computation, capabilities, concurrency, storage, security, and lifecycle around a first-class representation of the user's goal.**

The proposed runtime resembles a browser in architectural maturity, but its primary object is not a document/page. It is an **active user intent**.

The proposed stack therefore becomes approximately:

```text
User Intent
    ↓
Intent Object Model (IOM)
    ↓
Intent / State / Event Graph
    ↓
Context Graph + Memory + Knowledge
    ↓
Runtime Scheduler / Orchestrator
    ↓
Agent Workers / Tools / Capabilities
    ↓
Model Runtime(s)
    ↓
Verification / Evidence
    ↓
Events
    ↓
IOM Update
```

A crucial architectural principle emerged repeatedly:

> **The model is not the agent, and the model is not the runtime. The model is a compute resource used by the runtime.**

The most important new platform primitive proposed near the end of the discussion is the **Intent Object Model (IOM)**, analogous conceptually to the DOM:

> **The IOM is the runtime's shared, inspectable object graph representing an active intent, its goals, constraints, agents, dependencies, evidence, decisions, state, and artifacts.**

---

# 1. Conceptual Evolution

## 1.1 Starting Point: Recreated Fruit-Fly Brains

The conversation initially explored work involving recreated/simulated fruit-fly brains.

An important clarification was established:

- A simulated fruit-fly brain is generally a **computational model of neural circuitry**, not a literal biological brain that has simply been copied into software.
- Such a model can serve as a substrate for experiments, control tasks, behavior, or computational neuroscience.
- The interesting systems-level implication is that useful intelligence does not necessarily require one enormous general-purpose computational model.

This led to the first conceptual bridge:

```text
Large General Model
        vs.
Many Smaller Specialized Computational Systems
```

The latter can become powerful when combined with:

- routing
- context
- tools
- memory
- verification
- orchestration
- state
- task decomposition

The important shift was from:

> “How large must the model be?”

to:

> “What runtime can make many different computational capabilities work together?”

---

# 2. Small Specialized Models and the Agent/Model Distinction

The discussion then connected biological specialization to AI architectures.

A small model can be highly useful when it has a narrow role:

```text
                ┌─────────────────────┐
                │ Intelligence Runtime │
                └──────────┬──────────┘
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
   Coding Model       Vision Model       Planning Model
        │                  │                  │
   Retrieval Model    Classification      Verification
```

The important conclusion:

> **An agent should be a stable computational identity/lifecycle, while the model it invokes can be dynamically selected.**

For example:

```text
Coding Agent
    ├── simple task → small local coding model
    ├── difficult task → larger local model
    ├── very difficult task → cloud/frontier model
    └── critical verification → specialist verifier
```

Thus:

```text
Agent ≠ Model
Agent = role + state + lifecycle + capabilities + policies
Model = compute resource
```

This became one of the central architectural assumptions.

---

# 3. Personal / Local AI as an Intelligence Substrate

The next idea was an Apple-like local AI environment in which many specialized agents and models operate together.

The proposed architecture:

```text
                  Personal Intelligence Runtime
┌────────────────────────────────────────────────────────────┐
│                                                            │
│  Global State                                               │
│       │                                                    │
│       ├── Orchestrator / Scheduler                         │
│       │                                                    │
│       ├── Specialized Agents                               │
│       │                                                    │
│       ├── Skills / Tools                                   │
│       │                                                    │
│       ├── Model Pool                                       │
│       │     ├── Small local models                         │
│       │     ├── Larger local models                        │
│       │     └── Cloud/frontier models                      │
│       │                                                    │
│       ├── Model Router                                     │
│       ├── Context Engine                                   │
│       ├── Memory                                            │
│       ├── Retrieval / Knowledge                             │
│       └── Verification                                     │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

The runtime dynamically allocates compute according to task difficulty, latency, privacy, cost, and capability.

This led to a more general principle:

> **AI should be treated as a runtime substrate rather than a single model endpoint.**

---

# 4. Frontend Architecture as an Existing Reference Model

The user recognized that many of these concepts resemble modern frontend architecture.

This produced a detailed analogy:

| Frontend Concept | Intelligence Runtime Concept |
|---|---|
| UI component | Agent / capability |
| Design-system component | Agent archetype / reusable skill |
| Props | Task/context contract |
| Local component state | Agent execution state |
| Global state store | Shared Orchestrator AI state |
| Selector | Context selector |
| Action / command | Runtime command |
| Middleware | Policy / validation / logging |
| Async effect | Tool invocation |
| Router | Task/model routing |
| Memoization/cache | Context/reasoning/result cache |
| Error boundary | Agent failure containment |
| DevTools | Intelligence runtime inspector |

A particularly important insight was:

> **Selectors can construct minimal task-specific context from canonical global state.**

Instead of dumping a huge context window into a model:

```text
Global State
    ↓
Context Selector
    ↓
Minimal Relevant Context
    ↓
Model
```

This is analogous to frontend selectors deriving only the state a component needs.

---

# 5. State, Memory, Knowledge, Context, and Events

A major clarification was the need to keep several concepts separate.

## Definitions

### State
What is true now.

### Memory
What happened previously or what was learned.

### Knowledge
Information that exists externally and can be retrieved.

### Event Log
What happened, when, and in what order.

### Context
The subset of information assembled for a particular computation.

These are not interchangeable.

A useful model:

```text
                    ┌───────────────┐
                    │   Event Log   │
                    │ what happened │
                    └───────┬───────┘
                            │
                         project
                            ↓
                    ┌───────────────┐
                    │  Current State│
                    │ what is true  │
                    └───────┬───────┘
                            │
             ┌──────────────┼──────────────┐
             ↓              ↓              ↓
        semantic         graph         temporal
         memory        knowledge       history
             │              │              │
             └──────────────┼──────────────┘
                            ↓
                    Context Engine
                            ↓
                   Task-specific Context
```

The key architectural statement:

> **RAG should be a context provider, not the center of the architecture.**

For coding in particular, relationship-aware structures may be more useful than pure vector retrieval:

- dependency graph
- symbol graph
- semantic search
- Git history
- architecture decisions
- project state
- event history

---

# 6. CQRS and Event Sourcing

The user has experience with CQRS/event-driven architecture, and this became a strong fit for AI runtime design.

## Commands

Examples:

```text
CreateTask
UpdatePlan
ExecuteSkill
RecordDecision
AddMemory
ModifyPreference
AssignAgent
RequestVerification
EscalateTask
```

## Events

Examples:

```text
TaskCreated
PlanUpdated
SkillExecuted
DecisionRecorded
MemoryAdded
PreferenceChanged
AgentAssigned
VerificationRequested
TaskEscalated
```

The model can propose commands, but the runtime should validate and apply them deterministically.

```text
LLM
 │
 │ proposes command
 ↓
Runtime Validator
 │
 │ accepts/rejects
 ↓
Command Handler
 │
 ↓
Event
 │
 ↓
Projection
 │
 ↓
Current State
```

This creates a useful separation:

```text
Probabilistic layer
    → proposes intent/action

Deterministic runtime
    → validates, authorizes, executes, records
```

This is particularly important for security, provenance, reproducibility, and debugging.

---

# 7. XState / State Machines

The user strongly prefers XState/meta-state-store abstractions over vanilla Redux.

XState became a natural model for agent lifecycle:

```text
idle
  ↓
planning
  ↓
executing
  ↓
verifying
  ↓
complete
```

With recovery:

```text
                ┌──────────────┐
                │     idle     │
                └──────┬───────┘
                       ↓
                ┌──────────────┐
                │   planning   │
                └──────┬───────┘
                       ↓
                ┌──────────────┐
                │  executing   │
                └──────┬───────┘
                       ↓
                ┌──────────────┐
                │  verifying   │
                └───┬──────┬───┘
                    │      │
                 pass      fail
                    │      ↓
                    │  recovery/escalation
                    │      │
                    └──────┘
                       ↓
                  complete
```

A key split:

```text
State Machine / Orchestrator
    → decides what the agent is doing

Model Router
    → decides what compute resource to use

LLM
    → performs probabilistic computation
```

The LLM is therefore not the runtime scheduler.

---

# 8. Declarative Context Configuration

The user proposed an ESLint-like declarative configuration system for context orchestration.

This was considered a particularly strong idea.

Conceptually:

```text
agent.config
├── context
│   ├── identity
│   ├── active_project
│   ├── current_task
│   ├── coding_conventions
│   ├── relevant_files
│   └── recent_decisions
│
├── knowledge
│   ├── repository
│   ├── project_docs
│   └── architecture_decisions
│
├── skills
│   ├── git
│   ├── filesystem
│   └── test_runner
│
├── policy
│   ├── privacy: local_only
│   ├── max_context_tokens
│   └── minimum_confidence
│
└── models
    ├── local_small
    ├── local_large
    └── cloud_frontier
```

The conceptual API is:

> “For this task, construct context using these providers under these policies.”

A context provider hides its implementation:

```text
Context Provider
    ├── SQL
    ├── vector DB
    ├── graph DB
    ├── filesystem
    ├── browser
    ├── API
    └── event store
```

The agent does not need to know which storage technology is underneath.

---

# 9. Plugins, Skills, and Capability Negotiation

The architecture then moved toward a plugin ecosystem.

Conceptual packages:

```text
@context/github
@context/filesystem
@context/calendar
@context/notion
@context/browser
@context/codebase

@skill/git
@skill/playwright
@skill/sql
@skill/docker
@skill/figma

@model/local-small
@model/local-coder
@model/frontier
```

The preferred abstraction is **capability negotiation**.

Instead of:

```text
Agent → hard-coded model
```

the runtime asks:

```text
What capabilities are required?
        ↓
Which models/tools satisfy them?
        ↓
Which candidate best satisfies:
    - reasoning
    - coding
    - vision
    - tool use
    - context length
    - latency
    - cost
    - privacy/locality
        ↓
Select resource
```

This extends classic dependency injection:

> **Agents request capabilities, not concrete implementations.**

---

# 10. Model Routing and Decision Models

OpenRouter was discussed as an example of an external model-routing layer.

The conceptual relevance was:

```text
Task
 ↓
Task classification
 ↓
Candidate models/providers
 ↓
Routing policy
 ↓
Model
```

The important architectural lesson was not to make OpenRouter itself the intelligence runtime. Rather:

> A model router is one subsystem inside the larger runtime.

A newer low-cost decision-oriented model, **Jev**, was also discussed as a possible architectural fit for lightweight decisions such as:

- “Is this task simple enough for a small model?”
- “Should I retry?”
- “Should I escalate?”
- “Do I need another worker?”
- “Is verification necessary?”

These uses were treated as architectural possibilities rather than established product guarantees.

---

# 11. Mapping to the Twelve-Factor App

The 12-factor methodology was then mapped onto the runtime.

| 12-Factor Concept | Intelligence Runtime Interpretation |
|---|---|
| Codebase | Agent/runtime definition |
| Dependencies | Models, skills, tools, context providers |
| Config | Routing, context, policy, privacy, budgets |
| Backing services | Event store, vector DB, model server, browser, filesystem, cloud model |
| Build/release/run | Package/evaluate → release → execute |
| Processes | Disposable inference/agent workers with external durable state |
| Port binding | Capability interfaces |
| Concurrency | Parallel agents/model/tool calls |
| Disposability | Disposable model/worker instances |
| Dev/prod parity | Same runtime/model/context contracts across environments |
| Logs | Agent/model/tool/context events |
| Admin processes | Reindex, replay, rebuild projections, evaluate models |

Important caveat:

- Traditional 12-factor processes emphasize statelessness.
- Loaded models and KV caches can have significant runtime state.
- Therefore durable semantic state should remain external even if compute processes hold ephemeral execution state.

The conclusion was:

> **12-factor is primarily the deployment/operations layer; XState, CQRS, context, and IOM are behavioral/runtime layers.**

---

# 12. Browser as an Intelligence Runtime

The conversation then made its largest conceptual jump.

The browser is already a mature general-purpose runtime that coordinates:

- heterogeneous computation
- persistent state
- ephemeral state
- networking
- caching
- background execution
- security
- permissions
- extensions
- resource management
- user interaction
- process isolation
- storage
- lifecycle
- debugging

Therefore:

> **The browser is a useful reference architecture for an intelligence runtime.**

Not merely a metaphor.

---

# 13. Web Evolution → Intent Evolution

A proposed historical progression:

```text
Web 1
  Documents / resources

        ↓

Web 2
  Applications

        ↓

AI Web
  Capabilities / intent
```

Historically:

```text
User
 ↓
URL
 ↓
Application
 ↓
UI
 ↓
Action
```

Potential AI-native flow:

```text
User
 ↓
Intent
 ↓
Capability discovery
 ↓
Runtime orchestration
 ↓
Tools / services / models / data
 ↓
Outcome
```

This removes a layer of mandatory user interaction.

Instead of:

> “Go to the application and operate it.”

the user can express:

> “Achieve this outcome.”

The browser/runtime becomes the mediator between intent and capabilities.

---

# 14. AI Browser Thesis

The strongest formulation developed was:

> **The browser abstracted away OS/network complexity needed to access applications. An AI browser could abstract away applications themselves, exposing intent as the primary interface and dynamically discovering the capabilities, tools, models, data, and services necessary to fulfill that intent.**

This does **not** mean conventional UI disappears.

Instead:

```text
UI
 ↓
becomes a situational/output modality

Intent
 ↓
becomes the primary computational input
```

Existing websites can initially function as legacy application interfaces.

Future AI-native websites could expose structured capabilities directly.

---

# 15. Browser → Intelligence Runtime Mapping

The mapping became increasingly precise.

| Browser Primitive | Intelligence Runtime |
|---|---|
| Browser | Intelligence runtime |
| Browser tab | Intent workspace / agent session |
| Page | Session/workspace view |
| DOM | Intent Object Model |
| JavaScript | Agent/model computation |
| V8 | LLM/inference engine |
| Web APIs | Capability APIs |
| Web Worker | Agent worker |
| Service Worker | Background/event-driven agent |
| Event loop | Intelligence scheduler |
| Browser storage | Tiered state/memory |
| IndexedDB | Structured persistent state |
| Cache API | Context/result cache |
| localStorage | Lightweight persistent preferences |
| sessionStorage | Working/session memory |
| Cookies | Identity/session/auth metadata |
| HTTP cache | External resource/knowledge cache |
| Browser history | Temporal/task history |
| Extensions | Skills/plugins |
| Permissions | Capability security |
| Browser processes | Isolated runtime workers/services |
| GPU process | Specialized inference compute |
| Browser profile | Personal intelligence environment |
| DevTools | Intelligence Inspector |
| URL | Resource identifier |
| Intent | Desired outcome identifier |

Important differences were also identified:

| Browser | Intelligence Runtime |
|---|---|
| Mostly deterministic | Probabilistic |
| Page is primary unit | Intent/workspace is primary unit |
| URL identifies resource | Intent identifies desired outcome |
| HTTP is core interface | Capability negotiation may be core interface |
| State is comparatively objective | Knowledge has uncertainty/provenance |
| History is navigation | History includes decisions, evidence, actions, outcomes |
| JS executes code | Models perform probabilistic inference |
| Browser scheduler is mostly mechanical | Intelligence scheduler must reason about allocation |

---

# 16. The LLM ↔ V8 Analogy

One of the strongest late-stage conceptual refinements was:

> **The LLM could best be compared to V8: an important computational engine inside the overall harness, not the browser itself.**

This produced:

```text
Browser World                  Intelligence World
────────────────────────────────────────────────────────

Browser                         Intelligence Runtime
DOM                              IOM
JavaScript                       Agent/model computation
V8                               LLM / inference engine
Web APIs                         Capability APIs
Web Workers                      Agent Workers
Service Workers                  Background Agents
Event Loop                       Intelligence Scheduler
IndexedDB                        Structured persistent state
Cache API                        Context/result cache
Cookies                          Identity/session metadata
Browser History                  Event/decision history
DevTools                         Intelligence DevTools
Permissions                      Capability permissions
Extensions                       Skills/plugins
```

The core principle:

> **V8 is powerful because it exists inside a much larger browser runtime. Likewise, an LLM becomes much more useful when it exists inside a larger intelligence runtime.**

---

# 17. Browser Storage as a Memory Topology

The user observed that browsers already contain multiple storage layers:

- localStorage
- sessionStorage
- IndexedDB
- cookies
- cache
- history
- service-worker state

Rather than treating memory as one database, the AI runtime should have a **memory topology**.

Suggested mapping:

| Browser Storage | Intelligence Meaning |
|---|---|
| sessionStorage | Agent working memory |
| localStorage | Lightweight persistent preferences/config |
| IndexedDB | Structured persistent agent/project state |
| Cache API | Reconstructible context/results/artifacts |
| Cookies | Identity/session/auth metadata |
| HTTP cache | Cached external resources/knowledge |
| History | Task/navigation history |
| Service-worker state | Background runtime state |
| Browser profile | Persistent user intelligence environment |
| Event store | Immutable temporal intelligence history |

Important distinction:

> **The event store is not a cache and should not be collapsed into generic browser storage.**

It records:

- what happened
- when it happened
- which command caused it
- which agent acted
- what evidence existed
- what decision was made

This enables:

- replay
- audit
- provenance
- temporal reasoning
- state reconstruction
- debugging

A possible physical layout:

```text
runtime-storage/
├── session/
├── local/
├── indexed/
├── cache/
├── memory/
├── knowledge/
└── events/
```

---

# 18. Multi-Tab = Multi-Intent

A major refinement was that an AI browser tab should represent a **user-facing intelligence workspace / intent session**, rather than necessarily a single agent.

Conceptually:

```text
AI Browser
│
├── Tab: "Launch Product"
│   ├── Main Agent
│   ├── Research Agent
│   ├── Coding Agent
│   ├── Browser Agent
│   └── Verification Agent
│
├── Tab: "Plan Vacation"
│   ├── Research Agent
│   ├── Booking Agent
│   └── Constraint Agent
│
└── Tab: "Debug Production Issue"
    ├── Diagnostic Agent
    ├── Code Agent
    └── Verification Agent
```

Each tab/session owns:

- immediate context
- intent state
- agent graph
- working memory
- artifacts
- browser interactions
- execution history

All tabs can share:

- user profile
- persistent memory
- installed skills
- permissions
- model pool
- runtime
- durable knowledge

Therefore:

> **Tab = intent workspace, not agent.**

---

# 19. Workers and Event-Driven Multi-Agent Execution

The user connected subagents/workers to browser workers and service workers.

The refined analogy:

- Web Worker → direct parallel computation
- Service Worker → background/event-driven runtime associated with an application independent of the current page lifecycle

The proposed agent runtime therefore becomes event-driven:

```text
Main Agent
    │
    │ commands
    ↓
Scheduler
    │
    ├──────────────┬───────────────┬──────────────┐
    ↓              ↓               ↓              ↓
Research       Coding          Retrieval      Verification
Worker         Worker           Worker          Worker
    │              │               │              │
    └──────────────┴───────────────┴──────────────┘
                           │
                         Events
                           ↓
                      Event Store
                           ↓
                     State Projection
                           ↓
                      New Context
                           ↓
                        Decision
```

This avoids the undesirable architecture of a giant autonomous loop.

Instead:

> **Use event-driven orchestration with explicit state transitions, workers, and durable events.**

---

# 20. Browser Process Architecture as Reference

The browser already solves the problem of running many semi-independent activities safely in one environment.

Conceptually:

```text
Browser Host
│
├── Browser/UI Process
├── Renderer Processes
├── GPU Process
├── Utility Processes
├── Network Services
├── Web Workers
├── Service Workers
└── IPC / Message Passing
```

The AI analogue:

```text
Intelligence Runtime
│
├── Runtime / Orchestrator
├── Agent Workers
├── Retrieval Workers
├── Model Workers
├── Verification Workers
├── Tool Workers
└── IPC / Event Bus
```

The key principle:

> **Do not initially modify browser internals to add intelligence. Build the intelligence runtime as a separate host subsystem and integrate it with the browser through explicit APIs.**

Only later, if proven useful, could intelligence become a first-class browser process/service.

---

# 21. Browser Security → Capability Security

Browser permissions provide a natural reference.

Instead of allowing an agent unrestricted access:

```text
Agent
  ├── filesystem.read
  ├── filesystem.write
  ├── network.search
  ├── browser.navigate
  ├── browser.submit
  ├── calendar.write
  └── payment.execute
```

Capabilities should be:

- explicitly declared
- granted by policy
- scoped
- revocable
- auditable

This is analogous to browser permission systems.

---

# 22. Semantic Browser Capability Layer

A particularly important implementation refinement:

> **Do not make AI operate the DOM directly whenever a higher-level semantic capability can be exposed.**

Instead of:

```text
Agent
 ↓
DOM click/type/query
```

prefer:

```text
Agent
 ↓
Semantic Browser API
 ↓
Browser
```

Example capabilities:

```text
SearchWeb
ReadPage
FollowLink
FillForm
SubmitForm
DownloadArtifact
Authenticate
Purchase
Schedule
ExtractData
```

Existing websites can still be driven through their UI as a fallback.

Thus:

```text
AI-native capability
       ↓
semantic API
       ↓
direct structured execution

Legacy website
       ↓
browser automation
       ↓
DOM/UI execution
```

This is analogous to how higher-level browser APIs abstract lower-level platform mechanics.

---

# 23. Browser Core / Prototype Direction

The conversation considered several browser cores.

The preferred practical starting point was:

> **Chromium + CEF (Chromium Embedded Framework)**

because it provides a mature browser runtime while allowing an external host application to own the intelligence layer.

Alternatives discussed:

| Core | Conceptual fit | Practical role |
|---|---:|---|
| Chromium + CEF | Very high | Preferred prototype base |
| WebKit | High | Strong architectural alternative |
| Gecko / Firefox | High | Strong research/architecture alternative |
| Raw V8 / JavaScriptCore | Low | Inference/execution engine, not a browser platform |

The important point:

> **Do not fork an entire browser first.**

A prototype can instead look like:

```text
┌─────────────────────────────────────────────────────────────┐
│                     AI Browser Host                         │
│                                                             │
│  ┌───────────────────────────────────────────────────────┐  │
│  │              Intelligence Runtime                    │  │
│  │                                                       │  │
│  │  IOM / State / Events                                 │  │
│  │  Context Engine                                       │  │
│  │  Agents                                               │  │
│  │  Scheduler                                            │  │
│  │  Model Router                                         │  │
│  │  Memory                                               │  │
│  │  Skills / Plugins                                     │  │
│  │  Verification                                         │  │
│  └───────────────────────┬───────────────────────────────┘  │
│                          │                                  │
│                          │ browser API / IPC                │
│                          ↓                                  │
│  ┌───────────────────────────────────────────────────────┐  │
│  │                        CEF                            │  │
│  │                                                       │  │
│  │  Tabs / Pages / DOM                                   │  │
│  │  Storage                                              │  │
│  │  Networking                                           │  │
│  │  Workers                                              │  │
│  │  Sandbox                                              │  │
│  └───────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

A pragmatic stack discussed:

```text
CEF / Chromium
      +
TypeScript-facing runtime API
      +
SQLite / event store
      +
local model runtime
      +
optional cloud model router
```

The intelligence layer should be implemented in a fast-iterating environment rather than forcing all runtime logic into Chromium's C++ internals.

---

# 24. The Intent Object Model (IOM)

This became the most important late-stage idea.

The user proposed:

> If a browser tab represents one user intent, and a graph of subagents/workers collaborates to fulfill it, what is the equivalent of the DOM?

Answer:

> **Intent Object Model (IOM).**

The IOM is analogous conceptually to the DOM, but its semantic domain is intent rather than UI.

## Core definition

> **The IOM is the runtime's shared, inspectable object graph representing an active intent, its goals, constraints, agents, dependencies, evidence, decisions, state, and artifacts.**

It should not merely be a natural-language prompt.

It is a **living runtime representation**.

---

# 25. DOM vs IOM

| DOM | IOM |
|---|---|
| Represents UI/application structure | Represents intent/work structure |
| Nodes | Goals/subgoals/tasks/artifacts |
| Attributes | Constraints/metadata |
| DOM events | Intent events |
| JavaScript mutates DOM | Agents issue intent operations |
| Browser renders DOM | Runtime executes intent graph |
| DevTools inspect DOM | Intent DevTools inspect IOM |
| Accessibility APIs consume DOM | Agents/tools/context consume IOM |
| Extensions observe/manipulate DOM | Skills/plugins observe/manipulate IOM |
| DOM state reflects application | IOM state reflects intent execution |

This led to a powerful formulation:

> **IOM + Intent Events could be the equivalent of DOM + DOM Events for an intelligence runtime.**

---

# 26. IOM Structure

Initial conceptual structure:

```text
Intent
├── Goal
├── Constraints
├── Subgoals
├── Agents
├── State
├── Evidence
├── Decisions
├── Artifacts
└── Events
```

But the structure should probably be a graph rather than a strict tree because dependencies can cross branches.

Therefore:

> **“Intent Object Graph” may be technically more precise, while “Intent Object Model” is conceptually cleaner because it emphasizes the runtime API/model.**

Example:

```text
Launch Product
│
├── Research Market
│   ├── Competitors
│   └── Pricing
│
├── Build Product
│   ├── Frontend
│   └── Backend
│
├── Prepare Launch
│
└── Validate
     │
     ├── depends on Build Product
     └── depends on Research Market
```

The cross-dependencies motivate a graph.

---

# 27. IOM as the Contract Between Intelligence and Execution

The intended flow:

```text
User Intent
     ↓
IOM
     ↓
LLM / Agent proposes Intent Operation
     ↓
Runtime validates operation
     ↓
Command
     ↓
Event
     ↓
IOM projection updates
     ↓
Scheduler reacts
     ↓
Worker / Tool / Model executes
     ↓
Evidence / Result
     ↓
Event
     ↓
IOM updates again
```

Example:

```text
create_subgoal("Find flights")
        ↓
validate
        ↓
SubgoalCreated
        ↓
IOM updated
        ↓
Scheduler spawns research/booking worker
```

Potential intent events:

```text
IntentCreated
GoalAdded
ConstraintAdded
SubgoalCreated
AgentAssigned
AgentStarted
AgentProducedEvidence
DecisionProposed
DecisionAccepted
TaskBlocked
TaskEscalated
ArtifactCreated
VerificationRequested
VerificationPassed
VerificationFailed
IntentCompleted
```

---

# 28. Multi-Agent Collaboration Through the IOM

Instead of arbitrary agent-to-agent messaging:

```text
Agent A ↔ Agent B
Agent B ↔ Agent C
Agent A ↔ Agent C
```

prefer:

```text
Agent A ─┐
Agent B ─┼──→ IOM / Event Graph ←── Scheduler
Agent C ─┘
```

Agents:

- observe relevant intent state
- publish evidence
- propose decisions
- update subgoals
- request capabilities
- emit events

The IOM becomes the shared coordination substrate.

This provides:

- inspectability
- decoupling
- event history
- replayability
- auditability
- concurrency control
- better debugging

---

# 29. IOM + CQRS + Event Sourcing

The IOM fits naturally as a projection over an event stream.

```text
              Commands
                 ↓
        ┌─────────────────┐
        │ Runtime/Handlers │
        └────────┬────────┘
                 ↓
              Events
                 ↓
        ┌─────────────────┐
        │   Event Store   │
        └────────┬────────┘
                 ↓
            Projection
                 ↓
        ┌─────────────────┐
        │       IOM       │
        └────────┬────────┘
                 ↓
       Context / Scheduler /
       Agents / DevTools
```

This enables:

- replay
- time travel
- audit
- provenance
- state reconstruction
- deterministic debugging
- temporal queries

This is one of the strongest reasons to avoid making the IOM merely an in-memory JSON object.

---

# 30. Context as a Graph Around the IOM

The IOM should not contain all possible knowledge.

Instead:

```text
                    ┌───────────────┐
                    │      IOM      │
                    │ active intent │
                    └───────┬───────┘
                            │
                 ┌──────────┼──────────┐
                 ↓          ↓          ↓
             State       Memory     Knowledge
                 │          │          │
                 └──────────┼──────────┘
                            ↓
                     Context Engine
                            ↓
                    Task-specific Context
                            ↓
                         Model
```

The context engine can select:

- relevant goals
- current state
- relevant events
- semantic memory
- repository files
- project documentation
- user preferences
- external knowledge
- tool outputs
- evidence

according to the active agent and task.

---

# 31. Epistemic State

A future primitive suggested during the state/memory discussion was an explicit epistemic representation.

Instead of storing a fact as merely:

```json
{
  "value": "X"
}
```

represent something closer to:

```text
Proposition
Value
Confidence
Evidence
ObservedAt
VerifiedAt
```

Conceptually:

```text
EpistemicState
├── proposition
├── value
├── confidence
├── evidence[]
├── observed_at
└── verified_at
```

This matters because an intelligence runtime needs to distinguish:

- known
- observed
- inferred
- believed
- verified
- stale
- contradicted

The browser analogy alone does not provide this; it is one of the new primitives required by probabilistic systems.

---

# 32. Agent Graph

An IOM can expose an explicit agent graph:

```text
Intent
│
├── Planner
│
├── Researcher
│    ├── Web Search
│    └── Knowledge Retrieval
│
├── Builder
│    ├── Coding Worker
│    └── Test Worker
│
├── Browser Agent
│
└── Verifier
```

Agents are not necessarily permanent.

They can be:

- created
- suspended
- resumed
- escalated
- replaced
- destroyed

This is similar to browser workers/processes.

---

# 33. Scheduler

The scheduler is the intelligence runtime's equivalent of an event loop/resource scheduler.

Responsibilities:

- react to events
- determine runnable work
- allocate workers
- allocate models
- enforce budgets
- manage concurrency
- respect dependencies
- handle retries
- escalate
- schedule verification
- terminate completed workers

Conceptually:

```text
                 Events
                   ↓
             ┌───────────┐
             │ Scheduler │
             └─────┬─────┘
                   │
        ┌──────────┼───────────┐
        ↓          ↓           ↓
     Agent A    Agent B      Agent C
        │          │           │
        ↓          ↓           ↓
      Model      Model        Tool
        │          │           │
        └──────────┼───────────┘
                   ↓
                 Events
```

---

# 34. Model Router

The model router should be subordinate to the scheduler/runtime.

It can consider:

```text
Task requirements
+
Model capability profile
+
Latency budget
+
Cost budget
+
Privacy policy
+
Context size
+
Availability
+
Quality requirements
```

Example:

```text
if task is trivial:
    local-small

if coding:
    coding-specialist

if complex reasoning:
    local-large or cloud

if critical verification:
    high-quality verifier

if privacy == local_only:
    local candidate set only
```

This is adaptive compute allocation.

---

# 35. Progressive Enhancement for Intelligence

An important analogy to web development was:

> **Progressive enhancement can become progressive intelligence/compute escalation.**

Conceptually:

```text
             User Intent
                  ↓
           Small Local Model
                  │
          sufficient? ── yes → complete
                  │
                  no
                  ↓
          Larger Local Model
                  │
          sufficient? ── yes → complete
                  │
                  no
                  ↓
          Cloud / Frontier
                  │
          sufficient? ── yes → complete
                  │
                  no
                  ↓
             Human review
```

This creates graceful degradation and privacy-aware computation.

---

# 36. Memory and Context Topology

The runtime should distinguish at least:

```text
Working Context
      ↓
Session State
      ↓
Persistent Project State
      ↓
Semantic Memory
      ↓
Durable Knowledge
      ↓
Immutable Event History
```

Each layer has different:

- consistency
- retention
- retrieval
- mutability
- provenance
- cost
- latency

Therefore:

> **“Memory” should be treated as a topology, not a single vector database.**

---

# 37. AI Browser Profile

A browser profile could evolve into a **personal intelligence environment**.

It may contain:

```text
Profile
├── Identity
├── Preferences
├── Permissions
├── Persistent Memory
├── Installed Skills
├── Model Preferences
├── Local Knowledge
├── Event History
├── Projects
└── Intent Sessions
```

Multiple tabs/sessions share this profile while maintaining their own local intent state.

---

# 38. DevTools for Intelligence

The IOM strongly suggests an analogue to browser DevTools.

Proposed panels:

```text
Intelligence DevTools
├── Intent Graph
├── Event Timeline
├── Agent Graph
├── Context Inspector
├── Model Inspector
├── Memory Inspector
├── Capability Inspector
├── Decision Inspector
└── Resource / Cost Inspector
```

## Intent Graph

Shows:

- goals
- subgoals
- dependencies
- blockers
- completion status

## Event Timeline

Example:

```text
10:31 IntentCreated
10:31 GoalAdded
10:32 PlanCreated
10:32 AgentAssigned
10:33 ContextBuilt
10:33 ModelInvoked
10:34 CodeGenerated
10:34 TestExecuted
10:35 TestFailed
10:35 VerificationAgentSpawned
10:36 ContextExpanded
10:36 ModelEscalated
10:37 TestPassed
```

## Context Inspector

Shows exactly what a model received:

```text
Identity
Current Goal
Relevant Files
Recent Decisions
Retrieved Knowledge
Tool Results
Constraints
Token Budget
```

## Model Inspector

Shows:

```text
Selected model
Reason for selection
Capabilities
Latency
Token usage
Cost
Retry count
Escalation history
```

This is essential for debugging probabilistic systems.

---

# 39. “AI Vitals”

A further frontend/browser-style observability analogy was proposed.

Traditional web systems have metrics such as:

- latency
- CPU
- memory
- Web Vitals
- error rate

An intelligence runtime could expose:

```text
Task success rate
Context size
Inference latency
Tool latency
Model cost
Retry rate
Escalation rate
Verification rate
Correction rate
Human intervention rate
Energy consumption
```

These could become runtime SLOs.

---

# 40. Frontend Engineering Relevance

The discussion was partly motivated by concern that AI code generation could make frontend engineering obsolete.

The conclusion was more nuanced:

- Code generation does not eliminate systems architecture.
- Frontend engineering has developed many abstractions relevant to future AI systems.
- The value shifts upward from manually producing UI code toward designing composable runtime abstractions.

Relevant frontend-derived concepts include:

- decomposition
- composition
- state management
- state machines
- events
- routing
- lazy loading
- caching
- performance budgets
- permissions
- accessibility
- design systems
- observability
- declarative configuration
- progressive enhancement

A useful framing:

> **The web/frontend stack may be one of the existing engineering disciplines that has already solved parts of the runtime problem that AI now encounters.**

But avoid the overclaim:

> “Frontend has already solved AI.”

More precise:

> **Frontend/browser architecture provides mature solutions for runtime composition and stateful user-facing computation, which can be extended with probabilistic, epistemic, and intent-specific primitives.**

---

# 41. Design Systems → Intelligence Systems

A design-system component has:

- a stable API
- states
- variants
- behavior
- composition rules
- accessibility expectations
- implementation hidden behind a contract

An intelligence capability could work similarly:

```text
Capability
├── Contract
├── Inputs
├── Outputs
├── Preconditions
├── Permissions
├── State model
├── Failure modes
└── Implementation
       ├── local model
       ├── cloud model
       ├── API
       └── tool
```

The implementation can change without changing the agent's contract.

This is another reason the runtime should abstract models away.

---

# 42. The Full Conceptual Architecture

The consolidated architecture that emerged is:

```text
┌─────────────────────────────────────────────────────────────────────────┐
│                         AI BROWSER / INTENT PLATFORM                    │
│                                                                         │
│  ┌───────────────────────────────────────────────────────────────────┐  │
│  │                           USER PROFILE                             │  │
│  │                                                                   │  │
│  │ Identity · Preferences · Permissions · Persistent Memory          │  │
│  │ Installed Skills · Model Policies · Knowledge · History            │  │
│  └───────────────────────────────────┬───────────────────────────────┘  │
│                                      │                                  │
│                                      ↓                                  │
│  ┌───────────────────────────────────────────────────────────────────┐  │
│  │                     INTENT WORKSPACES / TABS                       │  │
│  │                                                                   │  │
│  │  Tab A                 Tab B                  Tab C                │  │
│  │  Intent A              Intent B               Intent C             │  │
│  │     │                     │                      │                 │  │
│  │     └─────────────────────┼──────────────────────┘                 │  │
│  │                           ↓                                        │  │
│  │                         IOM                                        │  │
│  │            Intent Object Model / Graph                             │  │
│  └───────────────────────────┬───────────────────────────────────────┘  │
│                              │                                          │
│            ┌─────────────────┼──────────────────┐                       │
│            ↓                 ↓                  ↓                       │
│       State / CQRS      Context Graph      Event Store                 │
│            │                 │                  │                       │
│            │                 ↓                  │                       │
│            │         Memory / Knowledge         │                       │
│            │                 │                  │                       │
│            └─────────────────┼──────────────────┘                       │
│                              ↓                                          │
│                     RUNTIME SCHEDULER                                   │
│                              │                                          │
│          ┌───────────────────┼────────────────────┐                     │
│          ↓                   ↓                    ↓                     │
│     Agent Workers       Tool Workers        Verification Workers        │
│          │                   │                    │                     │
│          └───────────────────┼────────────────────┘                     │
│                              ↓                                          │
│                        MODEL ROUTER                                     │
│                              │                                          │
│        ┌─────────────────────┼──────────────────────────┐              │
│        ↓                     ↓                          ↓              │
│   Small Local           Large Local              Cloud / Frontier      │
│        │                     │                          │              │
│        └─────────────────────┼──────────────────────────┘              │
│                              ↓                                          │
│                        MODEL RUNTIMES                                   │
│                              │                                          │
│                    inference / KV cache / GPU                          │
│                                                                         │
│  ┌───────────────────────────────────────────────────────────────────┐  │
│  │                         CAPABILITY LAYER                           │  │
│  │                                                                   │  │
│  │ Search · Browser · Filesystem · Git · APIs · Calendar · SQL       │  │
│  │ Payments · Forms · Downloads · Code Execution · External Apps     │  │
│  └───────────────────────────────────┬───────────────────────────────┘  │
│                                      │                                  │
│                                      ↓                                  │
│  ┌───────────────────────────────────────────────────────────────────┐  │
│  │                         BROWSER RUNTIME                            │  │
│  │                                                                   │  │
│  │ CEF / Chromium · Tabs · Pages · DOM · Networking · Storage         │  │
│  │ Workers · Service Workers · Sandbox · Permissions · Extensions     │  │
│  └───────────────────────────────────────────────────────────────────┘  │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

# 43. The Core Runtime Loop

The most compact representation of the whole architecture is:

```text
                  ┌──────────────────────┐
                  │     User Intent      │
                  └──────────┬───────────┘
                             ↓
                  ┌──────────────────────┐
                  │         IOM          │
                  │ Intent Object Model  │
                  └──────────┬───────────┘
                             ↓
                  ┌──────────────────────┐
                  │ Context Construction │
                  └──────────┬───────────┘
                             ↓
                  ┌──────────────────────┐
                  │   Agent / Planner    │
                  └──────────┬───────────┘
                             ↓
                  ┌──────────────────────┐
                  │     Scheduler        │
                  └──────────┬───────────┘
                             ↓
             ┌───────────────┼────────────────┐
             ↓               ↓                ↓
          Worker A         Worker B         Worker C
             ↓               ↓                ↓
           Model            Tool             Model
             │               │                │
             └───────────────┼────────────────┘
                             ↓
                        Evidence /
                         Results
                             ↓
                          Events
                             ↓
                     Event Projection
                             ↓
                          IOM Update
                             │
                             └──────────────→ repeat
```

This is the central event-driven control loop.

---

# 44. The Most Important Architectural Separation

The architecture can be summarized as five layers:

```text
1. Intent
   What does the user want?

2. Runtime
   How is work coordinated?

3. Capabilities
   What can the system do?

4. Models
   What computational resources can perform inference?

5. State / Memory / Knowledge
   What does the system currently know, remember, and observe?
```

Or:

```text
Intent
  ↓
Orchestration
  ↓
Capabilities
  ↓
Models / Tools
```

with:

```text
State + Events + Memory + Knowledge + Context
```

supporting every layer.

---

# 45. Model as a Compute Resource

The final architecture explicitly rejects:

```text
User
 ↓
LLM
 ↓
Everything
```

in favor of:

```text
User
 ↓
Intent Runtime
 ├── state
 ├── context
 ├── scheduler
 ├── agents
 ├── capabilities
 ├── memory
 ├── events
 ├── verification
 └── model router
       ↓
   model(s)
```

The model is analogous to:

- V8 in a browser
- a GPU kernel
- a database engine
- a compiler backend

It is extremely important, but it is not the whole platform.

---

# 46. The Intent Platform as an “OS for Intelligence”

A useful conceptual label considered throughout the discussion is:

> **Personal Intelligence Runtime**

or:

> **Intent Platform**

or:

> **AI Browser**

The strongest interpretation is that this is effectively an **operating environment for intelligence**, but unlike a traditional OS it begins from user intent rather than processes/files.

Potential hierarchy:

```text
Physical Machine
      ↓
Operating System
      ↓
Browser / Intelligence Runtime
      ↓
Intent Workspace
      ↓
Agents
      ↓
Capabilities
      ↓
Models / Tools
```

The AI browser therefore sits between traditional applications and the user.

---

# 47. Intent as the New Primary Addressing Primitive

A URL historically identifies a resource:

```text
https://example.com/resource
```

An intent identifies an outcome:

```text
"Prepare a launch plan for my product"
```

The runtime can translate this into:

```text
Intent
 ↓
Subgoals
 ↓
Capabilities
 ↓
Resources
 ↓
Execution
```

This does not mean URLs disappear.

Instead:

> **URLs become one resource-addressing mechanism inside a larger intent-oriented runtime.**

---

# 48. Websites as Legacy Application Interfaces

A likely evolutionary path:

```text
Today

Intent
 ↓
Human navigates to website
 ↓
Human operates UI
 ↓
Outcome
```

Near-term AI browser:

```text
Intent
 ↓
AI browser
 ↓
Website via browser automation
 ↓
Outcome
```

Longer-term:

```text
Intent
 ↓
Capability discovery
 ↓
Structured service/API
 ↓
Outcome
```

Thus the browser can initially use today's web and gradually move toward semantic capability protocols.

---

# 49. Capability-Oriented Web

The long-term web could expose:

```text
Capability:
    SearchFlights

Inputs:
    origin
    destination
    date
    passengers

Constraints:
    max_price
    preferred_airlines

Permissions:
    user_approval_required

Outputs:
    flight_options

Evidence:
    source
    timestamp
```

The AI runtime could discover and invoke this capability without requiring a human to navigate a visual website.

The website can remain as a human UI.

Therefore:

```text
Human interface
       +
Machine capability interface
```

could coexist.

---

# 50. Security Model

The intelligence runtime needs stronger controls than ordinary web automation.

Each capability should define:

```text
Capability
├── identity
├── inputs
├── outputs
├── permissions
├── side effects
├── risk level
├── approval requirements
├── audit events
└── implementation
```

Potential policy examples:

```text
filesystem.read       → automatic
filesystem.write      → scoped
network.search        → automatic
email.send            → approval
purchase.execute      → approval
financial.transfer    → strong approval
credential access     → isolated / restricted
```

The browser's security architecture is therefore a useful reference, but AI systems need intent- and risk-aware policy enforcement.

---

# 51. Verification as a First-Class Capability

A recurring architecture principle was:

> **Generation and verification should be separable.**

For example:

```text
Builder Agent
    ↓
Artifact
    ↓
Verifier Agent
    ↓
Evidence
    ↓
Decision
```

Verification can involve:

- tests
- static analysis
- independent model
- browser execution
- source comparison
- schema validation
- policy checks
- human approval

This reduces the assumption that the same model that produced an answer should be trusted to validate it.

---

# 52. Provenance

Every meaningful result should ideally retain:

```text
Who/what produced it
Which model
Which version
Which context
Which tools
Which external sources
Which events
When
Which verification
Which confidence
```

This enables a provenance graph:

```text
Artifact
   │
   ├── produced_by → Agent
   ├── model → Model Version
   ├── context → Context Snapshot
   ├── based_on → Evidence
   ├── verified_by → Verifier
   └── caused_by → Events
```

This is another place where an event-sourced IOM is stronger than a simple chat transcript.

---

# 53. Chat Becomes a View, Not the Runtime

The conversation itself can be understood as one possible UI over the intent runtime.

Instead of:

```text
Chat transcript = system state
```

the runtime becomes:

```text
IOM / State / Events
        ↓
   ┌────┴────┐
   ↓         ↓
  Chat      DevTools
   ↓         ↓
User UI   Runtime UI
```

The chat is therefore a projection/view over the actual intent state.

This is analogous to how a browser UI is not the underlying web application state.

---

# 54. Context Is Also a View

Likewise, model context should not be the canonical state.

```text
Canonical Runtime State
        ↓
Context Policy
        ↓
Context Selector
        ↓
Model Context
```

This makes context:

- reproducible
- inspectable
- configurable
- model-specific
- task-specific

and prevents the context window from becoming the de facto database.

---

# 55. Proposed Minimal IOM Schema

A conceptual TypeScript-style shape:

```ts
interface Intent {
  id: string;

  goal: Goal;

  constraints: Constraint[];

  state: IntentState;

  subgoals: Subgoal[];

  agents: AgentRef[];

  dependencies: Dependency[];

  evidence: EvidenceRef[];

  decisions: Decision[];

  artifacts: ArtifactRef[];

  metadata: IntentMetadata;
}
```

Possible subtypes:

```ts
interface Goal {
  id: string;
  description: string;
  status: GoalStatus;
  priority?: number;
}

interface Constraint {
  id: string;
  type: string;
  value: unknown;
  severity?: "soft" | "hard";
}

interface Subgoal {
  id: string;
  goal: string;
  status: TaskStatus;
  dependsOn: string[];
  assignedAgent?: string;
}

interface Evidence {
  id: string;
  source: string;
  contentRef: string;
  observedAt: string;
  verifiedAt?: string;
  confidence?: number;
}

interface Decision {
  id: string;
  proposal: string;
  rationale?: string;
  evidence: string[];
  status: "proposed" | "accepted" | "rejected";
}

interface ArtifactRef {
  id: string;
  type: string;
  location: string;
  producedBy: string;
}
```

This is conceptual, not a final API specification.

---

# 56. Proposed IOM Operations

The runtime could expose operations such as:

```text
intent.create()
intent.addGoal()
intent.addConstraint()
intent.createSubgoal()
intent.assignAgent()
intent.requestCapability()
intent.attachEvidence()
intent.proposeDecision()
intent.acceptDecision()
intent.createArtifact()
intent.requestVerification()
intent.block()
intent.escalate()
intent.complete()
```

Agents should not directly mutate arbitrary runtime state.

Instead:

```text
Agent
  ↓
Intent Operation
  ↓
Validation
  ↓
Command
  ↓
Event
  ↓
Projection
```

This is analogous to controlled mutation of browser/application state through event mechanisms.

---

# 57. Proposed Event Taxonomy

A more complete event listing:

```text
IntentCreated

GoalAdded
GoalUpdated
GoalCompleted

ConstraintAdded
ConstraintUpdated
ConstraintViolated

SubgoalCreated
SubgoalUpdated
SubgoalBlocked
SubgoalCompleted

AgentAssigned
AgentStarted
AgentPaused
AgentResumed
AgentStopped
AgentFailed

CapabilityRequested
CapabilityGranted
CapabilityDenied
CapabilityExecuted

ContextRequested
ContextBuilt
ContextExpanded
ContextInvalidated

ModelSelected
ModelInvoked
ModelCompleted
ModelFailed
ModelEscalated

EvidenceProduced
EvidenceAttached
EvidenceVerified
EvidenceContradicted

DecisionProposed
DecisionAccepted
DecisionRejected

ArtifactCreated
ArtifactUpdated
ArtifactVerified

VerificationRequested
VerificationPassed
VerificationFailed

IntentEscalated
IntentPaused
IntentResumed
IntentCompleted
IntentFailed
```

The exact taxonomy can evolve.

---

# 58. Agent Lifecycle

A possible generic state machine:

```text
┌──────┐
│ idle │
└──┬───┘
   ↓
┌──────────┐
│ planning │
└────┬─────┘
     ↓
┌───────────┐
│ executing │
└─────┬─────┘
      ↓
┌────────────┐
│ collecting │
│ evidence   │
└─────┬──────┘
      ↓
┌────────────┐
│ verifying  │
└───┬────┬───┘
    │    │
 pass   fail
    │    ↓
    │  recover
    │    │
    │  retry/escalate
    │    │
    └────┘
      ↓
┌──────────┐
│ complete │
└──────────┘
```

---

# 59. Resource Scheduling

The scheduler should manage at least:

```text
CPU
GPU
Model capacity
Network
Tool concurrency
Context budget
Token budget
Cost budget
Latency budget
Permissions
Worker count
```

A task might carry:

```ts
interface ResourcePolicy {
  maxLatencyMs?: number;
  maxCost?: number;
  maxTokens?: number;
  maxParallelWorkers?: number;
  privacy: "local_only" | "trusted_cloud" | "any";
  verification: "none" | "optional" | "required";
}
```

The runtime then allocates resources rather than letting every agent invoke arbitrary models.

---

# 60. Why the IOM May Be the Missing Primitive

The architecture had many pieces before IOM:

- agents
- models
- tools
- state
- context
- memory
- events
- scheduler
- browser

But there was no single semantic object that connected them.

The IOM supplies that center.

```text
Before:

Agents ↔ State
Agents ↔ Tools
Agents ↔ Models
Agents ↔ Memory
Agents ↔ Events
Agents ↔ Browser
```

This can become:

```text
              ┌─────────────┐
              │     IOM     │
              └──────┬──────┘
       ┌─────────────┼───────────────┐
       ↓             ↓               ↓
    Agents        Context         Events
       ↓             ↓               ↓
    Tools          Models          State
       ↓                             ↓
    Browser                       Memory
```

The IOM becomes the semantic coordination layer.

---

# 61. Why the Browser Analogy Is More Than Metaphor

The analogy is structurally strong because browsers already solve:

```text
heterogeneous execution
+
isolation
+
concurrency
+
storage
+
networking
+
permissions
+
background tasks
+
resource scheduling
+
caching
+
extension systems
+
debugging
+
user-facing workspaces
```

The missing AI-specific primitives are:

```text
intent
+
probabilistic execution
+
epistemic state
+
context construction
+
model routing
+
verification
+
provenance
+
capability negotiation
```

Therefore:

> **The intelligence runtime can be viewed as a browser-like execution environment extended with intent, epistemic, probabilistic, and capability-oriented primitives.**

---

# 62. Browser vs AI Runtime: What Must Be Invented

A naïve AI browser would simply add an LLM to a browser.

The proposed architecture is deeper.

## Existing browser primitives can be reused

```text
Processes
Workers
Networking
Storage
Caching
Permissions
Sandboxing
Extensions
Tabs
Profiles
DevTools
```

## New intelligence primitives are required

```text
IOM
Intent Events
Context Engine
Model Router
Epistemic State
Verification
Provenance
Capability Negotiation
Agent Lifecycle
Intent Scheduler
```

Therefore:

```text
AI Browser
=
Browser Runtime
+
Intelligence Runtime
```

not:

```text
AI Browser
=
Browser + Chatbot
```

---

# 63. Prototype Strategy

The recommended prototype should prove the runtime abstraction before attempting to reinvent the browser.

## Phase 1 — IOM + Runtime

Build:

```text
Intent
Goals
Subgoals
Agents
Events
State
Scheduler
```

with no sophisticated browser integration yet.

## Phase 2 — Context Engine

Add:

```text
Context Providers
Selectors
Memory
Knowledge
Event-derived context
```

## Phase 3 — Model Router

Add:

```text
Local model
Larger local model
Cloud model
Capability profiles
Escalation
Budget policies
```

## Phase 4 — Browser Integration

Add CEF/Chromium:

```text
Tabs
Pages
Navigation
Read
Search
Forms
Downloads
Authentication
```

## Phase 5 — Semantic Capability Layer

Expose:

```text
SearchWeb
ReadPage
FillForm
SubmitForm
DownloadArtifact
Schedule
Purchase
```

## Phase 6 — Intelligence DevTools

Build:

```text
Intent Graph
Event Timeline
Agent Graph
Context Inspector
Model Inspector
Memory Inspector
Capability Inspector
```

This produces a much stronger proof of concept than starting by modifying Chromium itself.

---

# 64. Minimal v0 Architecture

A very small implementation could be:

```text
┌──────────────────────────────────────────────┐
│                AI Browser v0                 │
│                                              │
│  ┌────────────────────────────────────────┐  │
│  │             Intent Runtime             │  │
│  │                                        │  │
│  │  IOM                                  │  │
│  │  Event Store                          │  │
│  │  Scheduler                            │  │
│  │  Context Engine                       │  │
│  │  Agent Worker                         │  │
│  │  Model Router                         │  │
│  └────────────────┬───────────────────────┘  │
│                   │                          │
│                   ↓                          │
│             Browser Adapter                 │
│                   │                          │
│                   ↓                          │
│             Chromium / CEF                  │
│                                              │
└──────────────────────────────────────────────┘
```

Possible persistence:

```text
SQLite
├── intents
├── goals
├── agents
├── events
├── artifacts
├── decisions
└── evidence
```

Possible runtime:

```text
TypeScript
  +
XState-like state machine
  +
event-driven scheduler
  +
local model API
  +
CEF
```

This is intentionally simple.

---

# 65. Potential TypeScript Runtime Shape

Conceptually:

```ts
interface IntelligenceRuntime {
  intents: IntentStore;
  events: EventStore;
  context: ContextEngine;
  scheduler: Scheduler;
  agents: AgentRegistry;
  models: ModelRouter;
  capabilities: CapabilityRegistry;
  memory: MemoryStore;
  verifier: VerificationEngine;
}
```

Agent:

```ts
interface Agent {
  id: string;
  capabilities: CapabilityRequirement[];
  state: AgentState;
  execute(task: AgentTask): Promise<AgentResult>;
}
```

Model:

```ts
interface Model {
  id: string;

  capabilities: ModelCapabilities;

  invoke(request: ModelRequest): Promise<ModelResult>;
}
```

Router:

```ts
interface ModelRouter {
  select(
    requirements: ModelRequirements,
    policy: ResourcePolicy
  ): Promise<Model>;
}
```

Context engine:

```ts
interface ContextEngine {
  build(
    intent: Intent,
    agent: Agent,
    policy: ContextPolicy
  ): Promise<ContextSnapshot>;
}
```

This reinforces the separation:

```text
Agent
  ↓ requests capability
Router
  ↓ selects
Model
  ↓ computes
Runtime
  ↓ records
Event
```

---

# 66. Declarative Agent Configuration

A conceptual configuration syntax:

```yaml
agent:
  name: coding-agent

context:
  - identity
  - active_project
  - current_goal
  - relevant_files
  - architecture_decisions
  - recent_events

knowledge:
  - repository
  - project_docs
  - git_history

skills:
  - filesystem
  - git
  - test_runner

models:
  candidates:
    - local-coder-small
    - local-coder-large
    - cloud-coder

policy:
  privacy: local_preferred
  max_context_tokens: 32000
  max_cost: 0.50
  verification: required
```

This is analogous to declarative frontend/runtime configuration and the ESLint idea discussed earlier.

---

# 67. Context Provider Interface

Conceptually:

```ts
interface ContextProvider {
  id: string;

  supports(query: ContextQuery): boolean;

  resolve(
    query: ContextQuery
  ): Promise<ContextFragment>;
}
```

Examples:

```text
UserContextProvider
ProjectContextProvider
RepositoryContextProvider
GitContextProvider
FilesystemContextProvider
BrowserContextProvider
EventContextProvider
SemanticMemoryProvider
KnowledgeGraphProvider
DocumentProvider
CalendarProvider
```

The context engine composes these.

---

# 68. Capability Interface

Conceptually:

```ts
interface Capability {
  id: string;

  description: string;

  inputSchema: unknown;

  outputSchema: unknown;

  permissions: Permission[];

  risk: RiskLevel;

  execute(input: unknown): Promise<unknown>;
}
```

Examples:

```text
browser.search
browser.readPage
browser.fillForm
browser.submitForm

filesystem.read
filesystem.write

git.diff
git.commit

calendar.read
calendar.create

email.read
email.send

sql.query
```

---

# 69. Capability Negotiation

A task can specify:

```ts
interface CapabilityRequirement {
  capability: string;
  constraints?: Record<string, unknown>;
  preferredLocality?: "local" | "cloud" | "any";
  maxLatencyMs?: number;
  maxCost?: number;
}
```

The runtime finds an implementation.

This allows:

```text
Agent
 ↓
"Need code execution"
 ↓
Capability registry
 ↓
available implementations
 ↓
sandbox / local worker / cloud service
```

rather than hard-coding infrastructure.

---

# 70. Event-Sourced IOM and Time Travel

One major advantage of the event model:

```text
Event 1
Event 2
Event 3
Event 4
Event 5
   ↓
Projection
   ↓
Current IOM
```

At any historical point:

```text
Events 1..N
   ↓
Historical projection
   ↓
IOM at time T
```

This enables questions like:

- Why did the agent make this decision?
- What did it know at the time?
- Which model was used?
- Which context was supplied?
- What evidence existed?
- When did the task become blocked?
- What caused escalation?

This is critical for serious AI systems.

---

# 71. Context Snapshots and Reproducibility

Every model invocation should ideally record:

```text
Model
Model version
System policy
Agent state
Intent state
Context snapshot
Tool outputs
Relevant events
Resource policy
Timestamp
```

Thus:

```text
Model Invocation
       │
       ├── model
       ├── context
       ├── intent snapshot
       ├── policy
       ├── tools
       └── result
```

This allows post-hoc analysis and improved reproducibility.

---

# 72. Deterministic vs Probabilistic Boundary

A recurring design principle:

## Probabilistic side

```text
LLMs
Planning
Classification
Inference
Natural language interpretation
Hypothesis generation
Decision proposals
```

## Deterministic side

```text
Authorization
State transitions
Event recording
Schema validation
Capability execution
Resource budgets
Policy enforcement
Dependency management
Artifact persistence
```

Boundary:

```text
Probabilistic Agent
        ↓
   Proposed Action
        ↓
Deterministic Runtime
        ↓
  Validated Command
        ↓
     Event
        ↓
    Execution
```

This is arguably one of the most important architectural principles in the entire concept.

---

# 73. “Agent” as a Runtime Object

The term agent should therefore mean something richer than “LLM loop.”

An agent is:

```text
Agent
├── identity
├── role
├── state machine
├── capabilities
├── context policy
├── resource policy
├── memory access
├── lifecycle
├── model selection policy
├── permissions
└── event participation
```

The LLM is merely one implementation component.

---

# 74. “Session” as a Runtime Object

A browser tab/session becomes:

```text
Session
├── Intent
├── IOM
├── Agent Graph
├── Context
├── Working Memory
├── Browser State
├── Artifacts
├── Event Stream
└── UI View
```

This is richer than a chat conversation.

---

# 75. “Profile” as a Runtime Object

The profile becomes:

```text
Profile
├── Identity
├── Preferences
├── Long-term Memory
├── Permissions
├── Skills
├── Model Pool
├── Knowledge
├── Projects
└── Sessions
```

This is analogous to a browser profile but semantically much richer.

---

# 76. “Tab” as a Runtime Boundary

A tab is useful because it provides:

- user-visible separation
- resource lifecycle
- context boundary
- interruptibility
- concurrency
- mental model

Thus:

```text
Browser Profile
     │
     ├── Intent Tab A
     ├── Intent Tab B
     └── Intent Tab C
```

Each tab can have many workers.

---

# 77. “Page” vs “Intent”

A conventional browser has:

```text
Tab
 └── Page
      └── DOM
```

The AI browser might have:

```text
Tab
 └── Intent
      └── IOM
           ├── Goals
           ├── Agents
           ├── Evidence
           └── Artifacts
```

The browser page is therefore no longer necessarily the primary unit of work.

---

# 78. “DOM” vs “IOM” as Platform Contract

The analogy can be expressed formally:

```text
Browser:

HTML
 ↓
DOM
 ↓
JavaScript / Browser APIs
 ↓
Rendering / Network / Storage / Workers
```

Intelligence runtime:

```text
Intent
 ↓
IOM
 ↓
Agents / Runtime APIs
 ↓
Models / Tools / Memory / Browser / Workers
```

The DOM is a common semantic substrate.

The IOM would serve the same role for intelligence.

---

# 79. IOM as a Standardization Opportunity

If the idea proves useful, the IOM could eventually become a platform-level interface.

Potential standardized concepts:

```text
Intent
Goal
Constraint
Subgoal
Agent
Capability
Evidence
Decision
Artifact
Context
Event
Verification
```

Potential standardized operations:

```text
create
observe
propose
assign
request
execute
verify
complete
escalate
```

Potential standardized events:

```text
created
updated
assigned
started
blocked
verified
completed
failed
```

This could allow independent implementations of:

- agents
- model runtimes
- browsers
- skills
- DevTools
- storage systems
- orchestration frameworks

to interoperate.

---

# 80. Long-Term Architecture

The eventual platform could look like:

```text
                          USER
                            │
                            ↓
                     ┌────────────┐
                     │   INTENT   │
                     └─────┬──────┘
                           ↓
                  ┌─────────────────┐
                  │       IOM       │
                  └───────┬─────────┘
                          ↓
              ┌────────────────────────┐
              │ Intelligence Runtime   │
              │                        │
              │ State / Events         │
              │ Context               │
              │ Memory                │
              │ Scheduler             │
              │ Agents                │
              │ Policies              │
              │ Verification          │
              └───────────┬────────────┘
                          ↓
                Capability Negotiation
                          │
          ┌───────────────┼────────────────┐
          ↓               ↓                ↓
       Browser           Apps            APIs
          │               │                │
          └───────────────┼────────────────┘
                          ↓
                     Tools / Data
                          │
                          ↓
                    Model Router
                          │
             ┌────────────┼─────────────┐
             ↓            ↓             ↓
          Local        Specialist     Frontier
          Models         Models         Models
             │            │             │
             └────────────┼─────────────┘
                          ↓
                      Execution
                          ↓
                       Events
                          ↓
                         IOM
```

---

# 81. The Deep Architectural Thesis

The conversation ultimately converged on several related theses.

## Thesis 1 — Model size is not the whole intelligence problem

Useful intelligence emerges from:

```text
Model
+
Context
+
Tools
+
Memory
+
State
+
Planning
+
Verification
+
Orchestration
```

## Thesis 2 — Agents should not be identified with models

Agents are runtime objects.

Models are compute resources.

## Thesis 3 — Context should be constructed, not dumped

Context is a task-specific projection of a larger state/knowledge environment.

## Thesis 4 — Memory is a topology

Different data requires different persistence and retrieval semantics.

## Thesis 5 — Events are first-class

AI systems need durable temporal history, not merely final outputs.

## Thesis 6 — Browser architecture is a valuable reference

Browsers already solve many of the infrastructure problems required by a general-purpose intelligence environment.

## Thesis 7 — Intent should become a first-class runtime object

This is the purpose of the IOM.

## Thesis 8 — The IOM could be the DOM-equivalent of an intelligence platform

It provides a shared semantic substrate between agents, runtime, tools, models, UI, and DevTools.

## Thesis 9 — The AI browser is not “browser + chatbot”

It is:

```text
Browser Runtime
+
Intent Runtime
+
Capability Layer
+
Model Runtime
```

## Thesis 10 — Frontend engineering remains relevant at the abstraction level

The next layer of software may reuse many concepts developed for:

- state machines
- component systems
- event-driven architecture
- routing
- caching
- design systems
- DevTools
- progressive enhancement
- permissions

---

# 82. Most Important Vocabulary

For continuity with the original discussion, preserve these terms:

| Term | Intended Meaning |
|---|---|
| Intelligence Runtime | Persistent execution environment coordinating AI computation |
| AI Browser | Browser/runtime organized around user intent |
| Intent Platform | General platform for intent-driven execution |
| IOM | Intent Object Model |
| Intent Object Graph | Graph interpretation of IOM |
| Intent Workspace | User-facing tab/session for one active intent |
| Agent | Stateful runtime worker/role, not a model |
| Worker | Isolated unit of concurrent execution |
| Capability | Explicit executable function/resource |
| Skill | Reusable capability package |
| Context Provider | Source/adapter for task context |
| Context Engine | Builds model-specific context |
| Model Router | Selects compute resources |
| Event Store | Durable temporal execution history |
| State Store | Current projected truth |
| Memory | Historical/learned information |
| Knowledge | External information |
| Epistemic State | Explicit representation of confidence/evidence/verification |
| Scheduler | Allocates workers, models, tools, and resources |
| Verification Agent | Independent validation role |
| Intelligence DevTools | Debugging/inspection environment |
| Profile | Persistent personal intelligence environment |

---

# 83. Strongest Final One-Sentence Definition

If the new assistant needs only one sentence to recover the central concept:

> **The proposed AI browser is a browser-like intelligence runtime in which each tab represents a user intent, the Intent Object Model (IOM) is the shared semantic object graph for that intent, agents/workers collaborate through state and events, context is dynamically constructed from memory/knowledge/state, capabilities are negotiated through explicit APIs, models are interchangeable compute resources selected by a scheduler/router, and the entire system is observable and auditable through intent-oriented DevTools.**

---

# 84. Strongest Final Architecture Diagram

```text
                              USER
                               │
                               │ natural-language / multimodal intent
                               ↓
                    ┌─────────────────────┐
                    │   INTENT WORKSPACE  │
                    │      / TAB          │
                    └──────────┬──────────┘
                               │
                               ↓
                    ┌─────────────────────┐
                    │         IOM         │
                    │ Intent Object Model │
                    │                     │
                    │ Goal                │
                    │ Constraints         │
                    │ Subgoals            │
                    │ Dependencies        │
                    │ Agents              │
                    │ Evidence            │
                    │ Decisions           │
                    │ Artifacts           │
                    │ State               │
                    └──────────┬──────────┘
                               │
                ┌──────────────┼──────────────┐
                │              │              │
                ↓              ↓              ↓
          Event Store      State Store    Context Engine
                │              │              │
                │              │       ┌──────┼────────┐
                │              │       ↓      ↓        ↓
                │              │    Memory Knowledge Events
                │              │       │      │        │
                └──────────────┴───────┴──────┴────────┘
                               │
                               ↓
                    ┌─────────────────────┐
                    │      SCHEDULER      │
                    │                     │
                    │ lifecycle           │
                    │ concurrency         │
                    │ budgets             │
                    │ policies            │
                    │ escalation          │
                    └──────────┬──────────┘
                               │
             ┌─────────────────┼─────────────────┐
             │                 │                 │
             ↓                 ↓                 ↓
        Agent Worker      Agent Worker      Agent Worker
             │                 │                 │
             ↓                 ↓                 ↓
        Capability       Capability        Capability
             │                 │                 │
             └─────────────────┼─────────────────┘
                               │
                               ↓
                     ┌──────────────────┐
                     │   MODEL ROUTER   │
                     └────────┬─────────┘
                              │
              ┌───────────────┼────────────────┐
              ↓               ↓                ↓
        Small Local      Specialist Local   Frontier
              │               │                │
              └───────────────┼────────────────┘
                              ↓
                       MODEL RUNTIME
                              │
                              ↓
                     inference / tools
                              │
                              ↓
                        evidence/results
                              │
                              ↓
                           EVENTS
                              │
                              └──────────────→ IOM
                                               │
                                               ↓
                                      DevTools / UI / User


                    ┌─────────────────────────────┐
                    │      BROWSER PLATFORM       │
                    │                             │
                    │ Chromium / CEF              │
                    │ Tabs / Pages / DOM          │
                    │ Networking                  │
                    │ Storage                    │
                    │ Workers                    │
                    │ Sandbox                    │
                    │ Permissions                │
                    │ Extensions                 │
                    └──────────────┬──────────────┘
                                   │
                          semantic browser APIs
                                   │
                     ┌─────────────┼─────────────┐
                     ↓             ↓             ↓
                  Search         Read          Forms
                  Navigate       Extract       Download
                  Authenticate   Schedule      Purchase
```

---

# 85. Final Mental Model for the New Assistant

When continuing this work, do **not** interpret the project as:

> “Build a browser with an LLM chatbot.”

Interpret it as:

> **Design a general-purpose, browser-like runtime for persistent, probabilistic, intent-driven computation.**

The browser contributes mature infrastructure:

```text
process isolation
storage
networking
workers
security
permissions
caching
tabs
profiles
extensions
DevTools
```

The intelligence layer adds:

```text
intent
IOM
agents
context
memory
epistemic state
model routing
capability negotiation
verification
provenance
event-driven orchestration
```

The model layer supplies:

```text
probabilistic computation
```

The resulting hierarchy is:

```text
                 INTENT
                    ↓
                   IOM
                    ↓
             INTELLIGENCE RUNTIME
                    ↓
          ┌─────────┼─────────┐
          ↓         ↓         ↓
        STATE     CONTEXT   EVENTS
          │         │         │
          └─────────┼─────────┘
                    ↓
                SCHEDULER
                    ↓
              AGENT WORKERS
                    ↓
              CAPABILITIES
                    ↓
               MODEL ROUTER
                    ↓
             MODEL RUNTIMES
                    ↓
                RESULTS
                    ↓
                 EVENTS
                    ↓
                   IOM
```

The deepest abstraction is therefore not the LLM, agent, browser page, or chat transcript.

It is:

```text
                         INTENT
                           │
                           ↓
                ┌────────────────────┐
                │        IOM         │
                │                    │
                │ shared semantic    │
                │ runtime object     │
                │ graph              │
                └─────────┬──────────┘
                          │
             ┌────────────┼─────────────┐
             ↓            ↓             ↓
          Context      Agents        Events
             ↓            ↓             ↓
          Models       Tools         State
             │            │             │
             └────────────┼─────────────┘
                          ↓
                     EXECUTION
                          ↓
                       RESULT
                          ↓
                         IOM
```

That is the conceptual endpoint of the discussion.
