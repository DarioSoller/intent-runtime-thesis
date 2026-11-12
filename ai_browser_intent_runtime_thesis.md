# The Intent Runtime

## Browser Architecture and Frontend Engineering as Reference Models for AI Harness Design

**Type:** Technical thesis (computer science / AI software engineering), derived from a design thought experiment
**Status:** Conceptual architecture and prototyping direction. Statements about specific products, models, browser engines or ecosystem tooling describe the state of discussion, not verified requirements.
**Classification:** Internal
**Version:** 1.0, October 2026

---

## Abstract

Current AI harnesses tend to collapse several distinct concerns into one object: the language model is treated as the agent, the scheduler, the state store and the user interface at once, and the chat transcript doubles as the system of record. This thesis argues that the next useful abstraction for applied AI is not a larger model but an *intelligence runtime*: a persistent, event-driven, capability-oriented execution environment in which models are interchangeable compute resources and the user's goal is represented as a first-class, inspectable runtime object.

The argument proceeds by reference architecture. The web browser already solves most of the infrastructure problems such a runtime faces (isolation, concurrency, storage tiers, permissions, background execution, extensions, debugging), and frontend engineering has produced mature answers to composition, state management, declarative configuration, dependency injection and observability. We map both onto the AI harness problem, separating the mapping into a build-time and a run-time perspective, and we identify where the analogy breaks and new primitives are required: intent, probabilistic execution, epistemic state, context construction, model routing, verification and provenance.

The central proposal is the Intent Object Model (IOM), a DOM-analogue for intent: a shared, event-sourced object graph through which agents, tools, models, user interfaces and developer tools coordinate. We give a minimal schema, an operation and event taxonomy, a declarative context-composition model modelled on ESLint, a capability-resolution model modelled on hierarchical dependency injection, and a phased prototype strategy based on Chromium/CEF. We close with a maturity gap analysis between the frontend and AI tooling ecosystems, most notably the absence of versioned package registries and lockfiles for agent components.

---

## Contents

1. Introduction
2. Foundations and Terminology
3. Architecture of the Intent Runtime
4. The Intent Object Model
5. Context Engineering
6. Orchestration: Lifecycle, Scheduling, Capability Resolution, Routing
7. The Browser as Reference Architecture
8. Frontend Engineering as Reference Discipline: Build Time and Run Time
9. Observability and Debuggability
10. Prototype Strategy
11. Discussion
12. Conclusion
Appendix A. Glossary

---

## 1. Introduction

### 1.1 From model size to runtime design

The thought experiment behind this thesis began with an unlikely topic: simulated fruit-fly brains. Such a simulation is a computational model of neural circuitry, derived from connectome data, not a biological brain copied into software. Its relevance here is systemic rather than biological. A small, narrowly specialised computational system can be useful when it sits inside an environment that gives it inputs, routes its outputs, and combines it with other specialised systems. Useful behaviour emerges from composition, not from the size of any one component.

Transferred to AI engineering, this shifts the design question from "how large must the model be?" to "what runtime can make many different computational capabilities work together?" A coding agent might use a small local model for trivial edits, a larger local model for harder tasks, a frontier model for the hardest ones, and an independent verifier for anything critical. The agent stays the same; the compute underneath it changes. That observation, generalised, is the thesis.

### 1.2 Problem statement

Harnesses built around large language models today exhibit a recurring set of structural weaknesses. We state them as problems to be solved rather than as criticism of any product:

- **P1. Model/agent conflation.** The model is addressed as if it were the agent, the planner and the scheduler. Lifecycle, permissions and state have no home of their own.
- **P2. Transcript as state.** The chat history is the only persistent representation of what the system knows and has done. It is linear, lossy and unqueryable.
- **P3. Context dumping.** Context is assembled by appending as much as fits. There is no selection discipline, no reproducibility, and the context window becomes the de facto database.
- **P4. Retrieval at the centre.** Vector retrieval is treated as the architecture rather than as one provider among several.
- **P5. Opaque execution.** There is no inspectable record of which model received which context, why a decision was taken, or what evidence existed at the time.
- **P6. Unbounded autonomy loops.** Orchestration is a single autonomous loop in which the model decides what happens next, including resource allocation and authorisation.

### 1.3 Thesis statement

> An intelligence runtime is to intent what the browser is to documents: a persistent execution environment that manages state, context, computation, capabilities, concurrency, storage, security and lifecycle around a first-class representation of the user's goal.

Three corollaries follow and recur throughout the document:

1. The model is not the agent and not the runtime; it is a compute resource used by the runtime.
2. Probabilistic components propose; deterministic components validate, authorise, execute and record.
3. The missing platform primitive is a shared, inspectable object model of the active intent, the IOM, which plays the role the DOM plays in a browser.

### 1.4 Method and scope

This is a design thesis, not an empirical one. Its method is architecture by analogy: we take two mature reference systems, the browser platform and the frontend engineering discipline, extract their structural solutions, and test each against the requirements of an intelligence runtime. Where the mapping holds we adopt the solution; where it breaks we name the new primitive required.

Analogy-driven design has known failure modes, and we return to them in Chapter 11. The claim is never that "frontend has solved AI". The claim is narrower: frontend and browser architecture provide mature solutions for runtime composition and stateful, user-facing, concurrent computation, and those solutions can be extended with probabilistic, epistemic and intent-specific primitives.

### 1.5 Contributions

- **C1.** The Intent Object Model as a DOM-analogue: definition, schema, operations, event taxonomy, and its grounding in CQRS and event sourcing (Chapter 4).
- **C2.** A layered runtime architecture with an explicit deterministic/probabilistic boundary (Chapters 3 and 6).
- **C3.** A systematic browser-to-runtime mapping with an explicit difference analysis (Chapter 7).
- **C4.** A frontend-to-harness mapping split into build-time and run-time perspectives, including ESLint as a reference architecture for declarative context composition, hierarchical dependency injection as the model for capability resolution, and a maturity gap analysis centred on package registries (Chapters 5, 6 and 8).
- **C5.** A phased prototype strategy that proves the runtime abstraction before touching browser internals (Chapter 10).

### 1.6 Outline

Chapter 2 fixes terminology. Chapter 3 gives the architecture at a glance. Chapters 4 to 6 detail the three subsystems that carry most of the novelty: the IOM, the context engine, and orchestration. Chapters 7 and 8 present the two reference architectures in depth. Chapter 9 covers observability, Chapter 10 the prototype path, Chapter 11 discussion and open questions, Chapter 12 the condensed theses.

---

## 2. Foundations and Terminology

### 2.1 State, memory, knowledge, events, context

Five terms are routinely used interchangeably in AI systems and must be separated:

| Term | Meaning | Mutability | Canonical? |
|---|---|---|---|
| **Event log** | What happened, when, in what order, caused by which command | Append-only | Yes (source of truth) |
| **State** | What is true now; a projection over events | Derived | Yes (projected truth) |
| **Memory** | What happened earlier or was learned; historical and semantic | Curated | No |
| **Knowledge** | Information that exists externally and can be retrieved | External | No |
| **Context** | The subset of all of the above assembled for one computation | Ephemeral | No (a view) |

Their relationship:

```text
                    ┌───────────────┐
                    │   Event Log   │  what happened
                    └───────┬───────┘
                            │ project
                            ↓
                    ┌───────────────┐
                    │ Current State │  what is true
                    └───────┬───────┘
                            │
             ┌──────────────┼──────────────┐
             ↓              ↓              ↓
        semantic         graph          temporal
         memory        knowledge        history
             └──────────────┼──────────────┘
                            ↓
                     Context Engine
                            ↓
                   task-specific context  →  model
```

The consequence for retrieval-augmented generation is direct: RAG is a context provider, not the centre of the architecture. For software engineering tasks in particular, relationship-aware structures (dependency graphs, symbol graphs, Git history, architecture decision records, project state, event history) are often more useful than similarity search over chunks.

### 2.2 Agent versus model

An **agent** is a stable computational identity with a lifecycle. A **model** is a compute resource the agent invokes. Formally:

```text
Agent  = role + state machine + capabilities + context policy
       + resource policy + permissions + lifecycle + event participation
Model  = capability profile + invoke(request) → result
```

An agent survives model substitution. A coding agent remains the same runtime object whether the current step runs on a small local model or a frontier model. This separation is what makes routing, escalation, privacy policy and cost control possible at all.

### 2.3 The deterministic/probabilistic boundary

The single most load-bearing principle in the architecture is that probabilistic and deterministic responsibilities are assigned to different components:

| Probabilistic side (models, agents) | Deterministic side (runtime) |
|---|---|
| Natural-language interpretation | Authorisation and permission checks |
| Planning, hypothesis generation | State transitions |
| Classification, inference | Event recording |
| Decision *proposals* | Schema validation |
| Action *proposals* | Capability execution |
| | Resource budgets, policy enforcement |
| | Dependency management, artifact persistence |

The boundary is crossed in one direction only:

```text
probabilistic agent → proposed operation → runtime validation
→ command → event → projection → execution (deterministic)
```

A model may propose `create_subgoal("Find flights")`. Only the runtime decides whether the proposal is valid, authorised and within budget, and only the runtime records it. This matters for security (the model never holds authority), for provenance (every state change has a validated command behind it), and for debugging (the system can be replayed).

### 2.4 Reference disciplines

Four established bodies of engineering practice inform the design:

- **CQRS and event sourcing** supply the command/event/projection discipline and durable history (Chapter 4).
- **Statecharts** (XState-style state machines) supply explicit agent lifecycles with recovery paths (Chapter 6).
- **The twelve-factor methodology** supplies the deployment and operations layer (Chapter 10).
- **Browser architecture and frontend engineering** supply the structural reference for composition, isolation, storage, permissions, declarative configuration, dependency injection and tooling (Chapters 7 and 8).

---

## 3. Architecture of the Intent Runtime

### 3.1 Design principles

- **DP1. Model as compute resource.** The model is comparable to V8 in a browser, a GPU kernel, a database engine or a compiler backend: indispensable, replaceable, not the platform.
- **DP2. Context is constructed, not dumped.** Context is a policy-governed projection of canonical state; it is reproducible and inspectable.
- **DP3. Events are first-class.** Durable temporal history is the source of truth; state is derived.
- **DP4. Memory is a topology.** Different information has different consistency, retention, retrieval, mutability, provenance, cost and latency characteristics; one vector store does not fit all.
- **DP5. Generation and verification are separable.** The component that produced an artifact is not trusted to validate it.
- **DP6. Capabilities are negotiated.** Agents request capabilities, not implementations.
- **DP7. Chat is a view.** The conversation is one projection over runtime state, alongside DevTools and other UIs; it is never the state itself.
- **DP8. Intent is the primary object.** Not the page, not the prompt, not the transcript.

### 3.2 Layer model

| Layer | Question answered | Main components |
|---|---|---|
| 1. Intent | What does the user want? | Intent workspace, IOM |
| 2. Runtime | How is work coordinated? | Scheduler, agent lifecycle, event store, context engine, policies |
| 3. Capabilities | What can the system do? | Capability registry, skills, semantic browser APIs, tools |
| 4. Models | What computes inference? | Model router, local and cloud model runtimes |
| 5. State / memory / knowledge | What does the system know, remember, observe? | Event log, projections, memory tiers, knowledge providers |

Layer 5 is not beneath the others; it supports all of them.

### 3.3 Core runtime loop

Every subsequent chapter elaborates one stage of this loop:

```text
                  User intent (natural language / multimodal)
                              ↓
                  ┌───────────────────────┐
                  │          IOM          │ goals · constraints · subgoals
                  │  Intent Object Model  │ agents · evidence · decisions
                  └───────────┬───────────┘ artifacts · state
                              ↓
                  Context construction (selectors over state, memory, knowledge, events)
                              ↓
                  Agent / planner proposes operations  ──→  runtime validates → commands
                              ↓
                  ┌───────────────────────┐
                  │       Scheduler       │ lifecycle · concurrency · budgets · escalation
                  └───────────┬───────────┘
             ┌────────────────┼────────────────┐
             ↓                ↓                ↓
        agent worker     tool worker    verification worker
             ↓                ↓                ↓
        capability       capability       capability
             └────────────────┼────────────────┘
                              ↓
                  Model router → model runtime(s) (local small / specialist / frontier)
                              ↓
                  evidence / results / artifacts
                              ↓
                  events → event store → projection → IOM update → repeat
                              │
                              └──→ views: chat · DevTools · user UI
```

### 3.4 Runtime objects

The runtime reifies six kinds of object. Each has identity, lifecycle and an inspectable structure.

| Object | Contains | Browser analogue |
|---|---|---|
| **Profile** | Identity, preferences, permissions, long-term memory, installed skills, model pool and policies, local knowledge, projects, event history, sessions | Browser profile |
| **Session (tab)** | One intent, its IOM, agent graph, working context, working memory, browser state, artifacts, event stream, UI view | Tab + page |
| **Agent** | Identity, role, state machine, capability requirements, context policy, resource policy, memory access, model-selection policy, permissions | Web worker / service worker |
| **Worker** | Isolated unit of concurrent execution hosting an agent, tool or model call | Process / worker |
| **Capability** | Identity, description, input/output schema, permissions, side effects, risk level, approval requirements, audit events, implementation | Web API |
| **Skill** | Versioned package bundling capabilities, context providers, configuration presets | Extension |

A tab is an **intent workspace**, not an agent. Several agents collaborate inside one tab; all tabs share the profile.

### 3.5 Full conceptual architecture

```text
┌───────────────────────────────────────────────────────────────────────┐
│                        AI BROWSER / INTENT PLATFORM                   │
│  ┌─────────────────────────────────────────────────────────────────┐  │
│  │ PROFILE  identity · preferences · permissions · memory          │  │
│  │          skills · model policies · knowledge · history          │  │
│  └───────────────────────────────┬─────────────────────────────────┘  │
│  ┌───────────────────────────────┴─────────────────────────────────┐  │
│  │ INTENT WORKSPACES   Tab A        Tab B        Tab C             │  │
│  │                     Intent A     Intent B     Intent C          │  │
│  │                        └────────────┼────────────┘              │  │
│  │                                   IOM                           │  │
│  └───────────────────────────────┬─────────────────────────────────┘  │
│                ┌─────────────────┼─────────────────┐                  │
│           State / CQRS      Context Graph      Event Store            │
│                │           memory · knowledge      │                  │
│                └─────────────────┼─────────────────┘                  │
│                            RUNTIME SCHEDULER                          │
│              agent workers · tool workers · verification workers      │
│                                  │                                    │
│  ┌───────────────────────────────┴─────────────────────────────────┐  │
│  │ CAPABILITY LAYER  search · browser · filesystem · git · APIs    │  │
│  │                   calendar · SQL · payments · code execution    │  │
│  └───────────────────────────────┬─────────────────────────────────┘  │
│                            MODEL ROUTER                               │
│              small local · large local · specialist · frontier        │
│                            MODEL RUNTIMES (inference, KV cache, GPU)  │
│  ┌─────────────────────────────────────────────────────────────────┐  │
│  │ BROWSER RUNTIME  CEF/Chromium · tabs · DOM · network · storage  │  │
│  │                  workers · sandbox · permissions · extensions   │  │
│  └─────────────────────────────────────────────────────────────────┘  │
└───────────────────────────────────────────────────────────────────────┘
```

---

## 4. The Intent Object Model

### 4.1 Motivation: the missing centre

Before the IOM was introduced in the design, the architecture already had agents, models, tools, state, context, memory, events, a scheduler and a browser. What it lacked was a single semantic object connecting them. Every subsystem talked to every other: agents to state, agents to tools, agents to memory, agents to the browser. The IOM replaces that mesh with a hub. It is the object that the context engine reads from, that agents propose changes to, that the scheduler reacts to, that DevTools inspect and that the chat renders.

The guiding question was: if a browser tab represents one user intent and a graph of workers collaborates to fulfil it, what is the equivalent of the DOM?

### 4.2 Definition and comparison with the DOM

> The IOM is the runtime's shared, inspectable object graph representing an active intent: its goals, constraints, subgoals, agents, dependencies, evidence, decisions, state and artifacts.

It is a living runtime representation, not a natural-language prompt.

| DOM | IOM |
|---|---|
| Represents UI/application structure | Represents intent/work structure |
| Nodes | Goals, subgoals, tasks, artifacts |
| Attributes | Constraints, metadata |
| DOM events | Intent events |
| JavaScript mutates the DOM | Agents propose intent operations |
| Browser renders the DOM | Runtime executes the intent graph |
| DevTools inspect the DOM | Intent DevTools inspect the IOM |
| Accessibility APIs consume the DOM | Agents, tools and context engine consume the IOM |
| Extensions observe and manipulate the DOM | Skills observe and manipulate the IOM |
| Platform contract: HTML → DOM → JS/Web APIs → rendering, network, storage | Platform contract: intent → IOM → agents/runtime APIs → models, tools, memory, browser |

IOM plus intent events is to the intelligence runtime what DOM plus DOM events is to the browser.

### 4.3 Structure: graph, not tree

The natural first sketch is a tree (intent → goals → subgoals → tasks). Dependencies cross branches, so the structure must be a directed graph with typed edges:

```text
Launch Product
├── Research Market
│   ├── Competitors
│   └── Pricing
├── Build Product
│   ├── Frontend
│   └── Backend
├── Prepare Launch
└── Validate
     ├── depends on → Build Product
     └── depends on → Research Market
```

"Intent Object Graph" is the more precise term; "Intent Object Model" is retained because, like "DOM", it names the runtime API and model rather than the data structure.

### 4.4 Minimal schema and operations

A conceptual TypeScript shape, deliberately small:

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

interface Goal       { id: string; description: string; status: GoalStatus; priority?: number; }
interface Constraint { id: string; type: string; value: unknown; severity?: "soft" | "hard"; }
interface Subgoal    { id: string; goal: string; status: TaskStatus; dependsOn: string[]; assignedAgent?: string; }
interface Evidence   { id: string; source: string; contentRef: string; observedAt: string;
                       verifiedAt?: string; confidence?: number; }
interface Decision   { id: string; proposal: string; rationale?: string; evidence: string[];
                       status: "proposed" | "accepted" | "rejected"; }
interface ArtifactRef{ id: string; type: string; location: string; producedBy: string; }
```

Agents never mutate the graph directly. They invoke intent operations, which are validated into commands:

```text
intent.create()            intent.attachEvidence()
intent.addGoal()           intent.proposeDecision()
intent.addConstraint()     intent.acceptDecision()
intent.createSubgoal()     intent.createArtifact()
intent.assignAgent()       intent.requestVerification()
intent.requestCapability() intent.block() · intent.escalate() · intent.complete()
```

### 4.5 The IOM as a projection: CQRS and event sourcing

The IOM is not an in-memory JSON object. It is a projection over an event stream:

```text
agent proposes operation
        ↓
runtime validator (schema, authorisation, budget, dependency rules)
        ↓ accept / reject
command handler
        ↓
event  →  event store (append-only)
        ↓
projection
        ↓
IOM  →  context engine · scheduler · agents · DevTools · chat
```

**Event taxonomy.** The following families cover the lifecycle; the exact set will evolve.

```text
Intent:        IntentCreated · IntentPaused · IntentResumed · IntentEscalated
               IntentCompleted · IntentFailed
Goal:          GoalAdded · GoalUpdated · GoalCompleted
Constraint:    ConstraintAdded · ConstraintUpdated · ConstraintViolated
Subgoal:       SubgoalCreated · SubgoalUpdated · SubgoalBlocked · SubgoalCompleted
Agent:         AgentAssigned · AgentStarted · AgentPaused · AgentResumed
               AgentStopped · AgentFailed
Capability:    CapabilityRequested · CapabilityGranted · CapabilityDenied · CapabilityExecuted
Context:       ContextRequested · ContextBuilt · ContextExpanded · ContextInvalidated
Model:         ModelSelected · ModelInvoked · ModelCompleted · ModelFailed · ModelEscalated
Evidence:      EvidenceProduced · EvidenceAttached · EvidenceVerified · EvidenceContradicted
Decision:      DecisionProposed · DecisionAccepted · DecisionRejected
Artifact:      ArtifactCreated · ArtifactUpdated · ArtifactVerified
Verification:  VerificationRequested · VerificationPassed · VerificationFailed
```

**Time travel.** Projecting events `1..N` yields the IOM at time *T*. This answers questions no transcript can: why did the agent decide this, what did it know at the time, which model and context were used, when did the task block, what triggered escalation.

**Invocation snapshots.** Every model call records model and version, system policy, agent state, intent snapshot, context snapshot, tool outputs, relevant events, resource policy and timestamp. Reproducibility of probabilistic systems is never perfect; reproducibility of *inputs* is achievable and is the precondition for post-hoc analysis.

### 4.6 The IOM as coordination substrate

Multi-agent systems built on point-to-point messaging (A talks to B, B to C, A to C) are hard to inspect and impossible to replay. The IOM replaces messaging with shared state:

```text
Agent A ─┐
Agent B ─┼──→ IOM / event graph ←── scheduler
Agent C ─┘
```

Agents observe relevant intent state, publish evidence, propose decisions, update subgoals, request capabilities and emit events. The pattern is a disciplined form of the classical blackboard architecture, with the difference that every write is a validated command and every read can be scoped by the context engine. It yields inspectability, decoupling, replayability, auditability and concurrency control for free.

### 4.7 Epistemic state and provenance

Browser state is comparatively objective: a cookie either exists or it does not. An intelligence runtime deals in beliefs. A fact must therefore not be stored as a bare value but as an epistemic record:

```text
EpistemicState
├── proposition
├── value
├── confidence
├── evidence[]
├── observed_at
└── verified_at
```

This lets the runtime distinguish known, observed, inferred, believed, verified, stale and contradicted information. Every meaningful artifact additionally carries a provenance graph:

```text
Artifact
├── produced_by  → agent
├── model        → model version
├── context      → context snapshot
├── based_on     → evidence
├── verified_by  → verifier
└── caused_by    → events
```

Neither epistemic state nor provenance exists in the browser; both are among the new primitives the analogy does not supply.

### 4.8 Standardisation opportunity

If the IOM proves useful, it could become a platform-level interface in the way the DOM did: a small vocabulary of concepts (intent, goal, constraint, subgoal, agent, capability, evidence, decision, artifact, context, event, verification), operations (create, observe, propose, assign, request, execute, verify, complete, escalate) and events (created, updated, assigned, started, blocked, verified, completed, failed). Independent implementations of agents, model runtimes, browsers, skills, DevTools, storage systems and orchestration frameworks could then interoperate. This is speculative, and Chapter 11 discusses why standards of this kind are hard to establish.

---

## 5. Context Engineering

### 5.1 Context is a view

Model context must never be canonical state. It is derived:

```text
canonical runtime state (IOM, state, memory, knowledge, events)
        ↓ context policy
        ↓ context selectors
model context (reproducible · inspectable · configurable · task- and model-specific)
```

The frontend analogue is exact: a selector derives from a global store only the slice a component needs, memoised, recomputed on change. Context selectors derive from canonical state only what a given agent needs for a given task. The failure mode they prevent is the context window becoming the database.

### 5.2 Context providers and the context engine

A context provider is an adapter that hides its storage technology (SQL, vector store, graph database, filesystem, browser, API, event store) behind a uniform interface:

```ts
interface ContextProvider {
  id: string;
  supports(query: ContextQuery): boolean;
  resolve(query: ContextQuery): Promise<ContextFragment>;
}

interface ContextEngine {
  build(intent: Intent, agent: Agent, policy: ContextPolicy): Promise<ContextSnapshot>;
}
```

Typical providers: user, project, repository, Git, filesystem, browser, events, semantic memory, knowledge graph, documents, calendar. The engine composes fragments under a policy (budget, privacy, priority, freshness) into a snapshot that is recorded with the invocation. The agent does not know which provider delivered which fragment, and does not need to.

### 5.3 Memory topology

Browsers already have a memory topology. They do not persist everything in one store; they have sessionStorage, localStorage, IndexedDB, Cache API, cookies, HTTP cache, history and service-worker state, each with different lifetime, scope and consistency. The runtime adopts the same discipline:

| Tier (runtime) | Browser analogue | Semantics |
|---|---|---|
| Working context | (in-memory) | Current computation only; rebuilt per task |
| Session / working memory | sessionStorage | Lives with the intent session |
| Lightweight preferences and config | localStorage | Small, persistent, low consistency needs |
| Structured persistent project/agent state | IndexedDB | Transactional, queryable |
| Reconstructible context, results, artifacts | Cache API / HTTP cache | Disposable; may be evicted |
| Identity, session, auth metadata | Cookies | Scoped, sensitive |
| Task and navigation history | History | Temporal, navigable |
| Background runtime state | Service-worker state | Survives page lifecycle |
| Semantic memory, durable knowledge | (no analogue) | Curated, retrievable |
| **Event store** | **(no analogue)** | **Immutable, append-only, source of truth** |

The event store is not a cache and must not be collapsed into generic storage. A plausible physical layout: `runtime-storage/{session,local,indexed,cache,memory,knowledge,events}/`.

### 5.4 Declarative context composition: ESLint as reference architecture

The design calls for context to be configured declaratively: "for this task, construct context from these providers under these policies." The frontend ecosystem already contains a mature system that does exactly this kind of work for a different domain. ESLint's configuration model, especially the flat configuration introduced as the default in version 9, is a reference architecture for declarative, composable, overridable, plugin-extended analysis pipelines. The claim here is that its structure transfers almost one-to-one to context composition.

**ESLint's architecture, reduced to its parts.** An `eslint.config.js` exports an array of configuration objects. Each object may declare `files` (glob matchers deciding which sources it applies to), `languageOptions` (the parser, parser options and ambient `globals`), `plugins` (namespaced packages contributing rules, processors and preset configurations), `rules` (rule identifier → severity and options), `processors` (pre-processors that extract analysable code from other file types), and `settings` (shared data available to all rules). Later objects override earlier ones for matching files; rule maps merge, language options deep-merge. Presets such as `js.configs.recommended` are ordinary config objects spread into the array. A rule is a module with `meta` (type, documentation, JSON Schema for its options, message templates, whether it is fixable) and a `create(context)` function that receives an injected context object and returns a visitor over AST node types, optionally using esquery selectors such as `CallExpression[callee.name="fetch"]`. Rules *report* problems with a severity, may offer *fixes* (applied automatically) or *suggestions* (offered, not applied). Different parsers produce ESTree-compatible trees, so rules are parser-independent. `eslint --print-config file.js` shows the fully resolved configuration for one file.

**The mapping.**

| ESLint | Context composition |
|---|---|
| Config array with cascade and override | Layered context config: profile → project → session → task; nearest wins |
| `files` globs | Task matchers: task type, intent domain, agent role, risk tier |
| `languageOptions.parser` | Fragment schema version (IOM/1.x); all providers emit schema-compatible fragments |
| `languageOptions.globals` | Ambient context always present: identity, date, active policies |
| `plugins` (namespaced) | Context-provider and skill packages, namespaced (`@context/repository`) |
| `rules` with severity `off`/`warn`/`error` | Fragment rules with `off`/`optional`/`required`; a required provider that cannot deliver fails context construction |
| Rule options validated by JSON Schema | Provider options validated at build time |
| `processors` | Transformers: chunking, summarisation, redaction of personal data, de-duplication |
| `settings` | Shared provider settings |
| Presets (`recommended`) | Agent archetype presets (`@context-config/coding/recommended`) |
| Rule `create(context)` with injected context object | Provider `resolve(query)` with injected runtime handle; providers never reach into global state |
| esquery AST selectors | IOM selectors: "subgoals blocked on evidence older than one hour" |
| `report` vs `fix` vs `suggest` | Emit `ContextViolation` event vs auto-adjust (truncate, drop optional) vs propose a decision |
| Formatters (stylish, JSON, SARIF) | Prompt renderers per model family; same snapshot, different serialisation |
| `--print-config` | `runtime context explain <task>`: the Context Inspector |
| Inline disable directives | Per-task exceptions, recorded as events for audit |

**What it looks like.** The two configurations below are structurally the same program.

```js
// eslint.config.js
import js from "@eslint/js";
import tseslint from "typescript-eslint";
import importPlugin from "eslint-plugin-import";

export default [
  js.configs.recommended,
  ...tseslint.configs.recommended,
  {
    files: ["src/**/*.ts"],
    plugins: { import: importPlugin },
    languageOptions: { parser: tseslint.parser, globals: { window: "readonly" } },
    rules: {
      "import/no-cycle": "error",
      "@typescript-eslint/no-explicit-any": ["warn", { fixToUnknown: true }],
    },
  },
  { files: ["**/*.test.ts"], rules: { "@typescript-eslint/no-explicit-any": "off" } },
];
```

```js
// context.config.js
import coding from "@context-config/coding";
import repo from "@context/repository";
import git from "@context/git";

export default [
  ...coding.configs.recommended,
  {
    tasks: ["coding/*"],
    providers: { repo, git },
    schema: "iom/1.0",
    ambient: { identity: "always", activeProject: "always" },
    fragments: {
      "repo/relevant-files":  ["required", { maxFiles: 20, ranking: "symbol-graph" }],
      "git/recent-commits":   ["optional", { limit: 10 }],
      "memory/decisions":     "required",
    },
    processors: ["@context/processors/redact-personal-data", "@context/processors/summarise-long"],
    budget: { maxTokens: 32000, onOverflow: "report" },
  },
  {
    tasks: ["coding/hotfix"],
    fragments: { "git/recent-commits": ["required", { limit: 30 }] },
    budget: { maxTokens: 64000 },
  },
];
```

**Pipeline correspondence.** ESLint: sources → parser → AST → rules traverse → messages → formatter. Context engine: sources → providers → IOM-shaped fragments → selectors and fragment rules → context snapshot → prompt renderer. The intermediate representation is the point. ESLint's rules are portable because every parser produces ESTree. Context rules become portable when every provider produces the same fragment schema, which is one more argument for standardising the IOM.

### 5.5 Linting the harness itself

The ESLint analogy has a second application. Agent configurations are programs and can be statically analysed before anything runs. A harness linter would flag, at build time: a `purchase.execute` capability granted without an approval policy; a high-risk task without `verification: required`; required fragments whose minimum size exceeds the token budget; unpinned skill versions; a privacy policy of `local_only` combined with a cloud-only model candidate list; capabilities requested by an agent but provided by no installed skill. None of this requires a model. All of it prevents classes of runtime failure that today are discovered in production transcripts.

---

## 6. Orchestration: Lifecycle, Scheduling, Capability Resolution, Routing

### 6.1 Agent lifecycle as a statechart

An agent is a state machine. The generic lifecycle, with recovery:

```text
idle → planning → executing → collecting evidence → verifying ─┬─ pass → complete
                                      ↑                         │
                                      └── retry / escalate ◄── fail → recover
```

The choice of statecharts over an unstructured reducer is deliberate: hierarchical and parallel states, guarded transitions, entry/exit actions and explicit error states are all needed, and all are available in XState-style libraries. Agents can be created, suspended, resumed, escalated, replaced and destroyed, like browser workers and processes.

The resulting division of labour:

```text
state machine / orchestrator → decides what the agent is doing
model router                 → decides which compute resource to use
model                        → performs probabilistic computation
```

The model is never the scheduler.

### 6.2 Scheduler and resource policy

The scheduler is the runtime's event loop and resource manager. It reacts to events, determines runnable work (respecting dependencies in the IOM graph), allocates workers and models, enforces budgets, manages concurrency, handles retries and escalation, schedules verification and terminates completed workers. Resources under management include CPU, GPU, model capacity, network, tool concurrency, context budget, token budget, cost budget, latency budget, permissions and worker count. Tasks carry a policy:

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

Unlike a browser event loop, which is mechanical, this scheduler must sometimes *reason* about allocation. That reasoning is itself a candidate for a small decision model (Section 6.4), but the decision is always executed deterministically by the scheduler.

### 6.3 Capability resolution as hierarchical dependency injection

The design principle "agents request capabilities, not implementations" is dependency injection. Frontend engineers know DI primarily through Angular, and Angular's injector model is unusually complete, which makes it a useful reference. We walk through its constructs and map each.

**Angular's DI, reduced to its parts.** Injectors form a tree: platform injector → root injector (`providedIn: 'root'`) → injectors created for lazily loaded routes or modules → element injectors declared on components and directives. A dependency is resolved by walking up from the requesting node; the nearest provider wins. Providers come in several shapes: `useClass` (instantiate), `useValue` (constant), `useFactory` with `deps` (compute at injection time), `useExisting` (alias one token to another). Non-class dependencies are addressed through `InjectionToken`s. `multi: true` collects every registered provider for a token into an array, which is how HTTP interceptor chains are assembled. Resolution modifiers (`@Optional()`, `@Self()`, `@SkipSelf()`, `@Host()`) control whether absence is an error and which part of the tree may answer. Lifecycle hooks (`ngOnInit`, `ngOnDestroy`) are called by the framework, not by the component. `TestBed` swaps providers for tests.

**The mapping.**

```text
Angular injector tree                    Intent runtime scope tree
──────────────────────────────           ─────────────────────────────────────────
platform injector                        runtime     (model pool, registries, host)
  └─ root injector                         └─ profile   (identity, privacy policy, memory)
       └─ lazy module injector                  └─ session   (intent, grants, working memory)
            └─ component injector                    └─ agent     (role, context policy, candidates)
                 └─ child component                       └─ task      (overrides, budgets)
```

| Angular DI | Capability resolution |
|---|---|
| `InjectionToken<T>` | Capability identifier (`filesystem.read`, `model.coding`); abstract, never a concrete class |
| Constructor parameters | Agent's declared `CapabilityRequirement[]`; the runtime supplies, the agent never constructs |
| `useClass` | Bind a capability to a local implementation (sandbox executor) |
| `useValue` | Bind to a fixed resource (a pinned model for reproducible evals) |
| `useFactory` + `deps` | **The model router is a factory provider**: `provide: CODING_MODEL, useFactory: (router, policy) => router.select(req, policy)` |
| `useExisting` | Alias `browser.search` to `web.search`; capability indirection |
| `multi: true` | Aggregate many context providers under one token; aggregate policy interceptors into a chain |
| Interceptor chain | Command middleware: authorisation → redaction → budget check → logging, in order |
| `@Optional()` | Optional capability; degrade gracefully if no implementation is installed |
| `@Self()` / `@SkipSelf()` | "Use only session-scoped memory" vs "skip to profile memory" |
| Lazy module with own injector | Skill loaded on demand with its own scoped providers |
| `providedIn: 'root'` tree-shaking | Only capabilities actually requested are bundled into an agent package (Chapter 8) |
| Lifecycle hooks | Agent lifecycle callbacks invoked by the scheduler, not by the agent |
| `TestBed.overrideProvider` | Eval harness: swap live model for a recorded one, swap tools for event-store replays |

A sketch in Angular-flavoured pseudo-code:

```ts
export const CODING_MODEL      = new InjectionToken<Model>("capability:model.coding");
export const CONTEXT_PROVIDERS = new InjectionToken<ContextProvider[]>("context.providers");
export const COMMAND_PIPELINE  = new InjectionToken<CommandInterceptor[]>("command.pipeline");

providers: [
  { provide: CODING_MODEL, useFactory: (r: ModelRouter, p: ResourcePolicy) =>
      r.select({ capability: "coding", minContext: 32000 }, p), deps: [ModelRouter, ResourcePolicy] },
  { provide: CONTEXT_PROVIDERS, useClass: RepositoryContextProvider, multi: true },
  { provide: CONTEXT_PROVIDERS, useClass: GitContextProvider,        multi: true },
  { provide: COMMAND_PIPELINE,  useClass: AuthorizationInterceptor,  multi: true },
  { provide: COMMAND_PIPELINE,  useClass: BudgetInterceptor,         multi: true },
  { provide: COMMAND_PIPELINE,  useClass: AuditInterceptor,          multi: true },
]
```

**Where DI must be tightened.** In Angular a child injector may override a parent's provider arbitrarily. In an intelligence runtime, overrides along the scope tree must be *attenuating*: a session may narrow the profile's permissions, never widen them; a task may restrict the model candidate set, never add a cloud model to a `local_only` session. This is capability-based security grafted onto DI, and it is a point where the analogy must be deliberately broken.

**Negotiation on top of resolution.** Resolution answers "which implementation satisfies this token?" Negotiation adds "which of several satisfying implementations is best under this policy?", scoring candidates on reasoning quality, coding, vision, tool use, context length, latency, cost and locality. DI gives the structure; the scoring function is the new part.

```ts
interface CapabilityRequirement {
  capability: string;
  constraints?: Record<string, unknown>;
  preferredLocality?: "local" | "cloud" | "any";
  maxLatencyMs?: number;
  maxCost?: number;
}
```

### 6.4 Model routing, progressive escalation, decision models

The model router is a subsystem subordinate to the scheduler, never the runtime itself. External routing services can be one backend among several, not the architecture. The router scores candidates against task requirements, capability profile, latency and cost budgets, privacy policy, context size, availability and quality requirements:

```text
trivial task              → small local model
coding                    → coding specialist
complex reasoning         → large local or cloud model
critical verification     → high-quality independent verifier
privacy = local_only      → local candidate set only
```

**Progressive enhancement becomes progressive compute escalation.** A web page is built to work with minimal capability and improve as more is available. The runtime inverts the budget: start with the cheapest sufficient resource and escalate only on insufficiency.

```text
intent → small local model → sufficient? → done
                 ↓ no
         larger local model → sufficient? → done
                 ↓ no
         cloud / frontier   → sufficient? → done
                 ↓ no
         human review
```

This yields graceful degradation (the system still works offline, with lower quality) and privacy by default (data leaves the device only when escalation requires it).

**Decision models.** Many scheduler decisions are small classification problems: is this task simple enough for a small model, should I retry, should I escalate, do I need another worker, is verification necessary. A low-cost decision-oriented model is a plausible fit for these. The source discussion named a candidate; its suitability is unverified and the architecture does not depend on any particular model.

### 6.5 Verification as a first-class capability

Generation and verification are separated by construction:

```text
builder agent → artifact → verifier agent → evidence → decision
```

Verification may be tests, static analysis, an independent model, browser execution, source comparison, schema validation, policy checks or human approval. The architectural effect is that the system no longer assumes the model that produced an answer should be trusted to validate it. Verification results are evidence in the IOM and feed epistemic state.

---

## 7. The Browser as Reference Architecture

### 7.1 Why the analogy is structural

A browser is a mature general-purpose runtime that already coordinates heterogeneous computation, persistent and ephemeral state, networking, caching, background execution, security and permissions, extensions, resource management, user interaction, process isolation, storage, lifecycle and debugging. Every one of these is a requirement of an intelligence runtime. The analogy is therefore not rhetorical; it is a list of solved problems to reuse.

### 7.2 Primitive mapping and differences

| Browser primitive | Intelligence runtime |
|---|---|
| Browser | Intelligence runtime |
| Tab | Intent workspace / session |
| Page | Session view |
| DOM | IOM |
| JavaScript | Agent and model computation |
| V8 | LLM / inference engine |
| Web APIs | Capability APIs |
| Web Worker | Agent worker (direct parallel computation) |
| Service Worker | Background, event-driven agent independent of the current view |
| Event loop | Intelligence scheduler |
| Storage tiers | Memory topology (Section 5.3) |
| Browser history | Event and decision history |
| Extensions | Skills and plugins |
| Permissions | Capability security |
| Processes (browser, renderer, GPU, utility, network) | Orchestrator, agent workers, model workers, verification workers, tool workers |
| GPU process | Specialised inference compute |
| Profile | Personal intelligence environment |
| DevTools | Intelligence DevTools |
| URL | Resource identifier |
| (none) | Intent: desired-outcome identifier |

Where the two differ:

| Browser | Intelligence runtime |
|---|---|
| Mostly deterministic | Probabilistic |
| Page is the primary unit | Intent is the primary unit |
| URL identifies a resource | Intent identifies an outcome |
| HTTP is the core interface | Capability negotiation may be the core interface |
| State is comparatively objective | Knowledge carries uncertainty and provenance |
| History is navigation | History includes decisions, evidence, actions, outcomes |
| JavaScript executes code | Models perform inference |
| Scheduler is mechanical | Scheduler must sometimes reason about allocation |

### 7.3 The LLM is V8, not the browser

V8 is powerful because it exists inside a much larger runtime that supplies it with a DOM to act on, APIs to call, storage to persist into, a security model to live under and tools to debug it. Remove the browser and V8 is a fast interpreter with nothing to do. The LLM is in the same position. Its usefulness is bounded by the harness around it, which is why investment in the harness is not a detour from AI progress but a component of it.

### 7.4 Process architecture and workers

The browser runs many semi-independent activities safely in one environment through process separation and message passing. The runtime mirrors this: orchestrator, agent workers, retrieval workers, model workers, verification workers, tool workers, connected by an event bus rather than ad-hoc IPC. Two worker types carry over with precision. A Web Worker is direct parallel computation bound to a page: an agent worker spawned for a subgoal. A Service Worker is a background, event-driven runtime that outlives any page: a monitoring agent that reacts to events when no session is open.

The event-driven shape:

```text
main agent → commands → scheduler → {research, coding, retrieval, verification} workers
          → events → event store → projection → new context → decision → …
```

This is the alternative to the undesirable "one giant autonomous loop": explicit state transitions, isolated workers, durable events.

### 7.5 Tabs as intents, profiles as environments

A tab provides user-visible separation, a resource lifecycle, a context boundary, interruptibility, concurrency and a mental model users already have. Mapping tab to intent, rather than tab to agent, preserves all of these: "Launch Product", "Plan Vacation" and "Debug Production Issue" are three tabs, each hosting its own agent graph, working memory, artifacts and event stream, all sharing one profile with identity, persistent memory, installed skills, permissions and model pool.

### 7.6 Security: permissions become capability security

Browser permissions are the right starting point and insufficient on their own. Capabilities must be explicitly declared, granted by policy, scoped, revocable and auditable, and additionally risk-tiered and intent-aware:

```text
filesystem.read      → automatic
filesystem.write     → scoped
network.search       → automatic
email.send           → approval
purchase.execute     → approval
financial.transfer   → strong approval
credential access    → isolated / restricted
```

Each capability declares identity, inputs, outputs, permissions, side effects, risk level, approval requirements and audit events. The deterministic boundary of Section 2.3 is what makes enforcement possible: the model cannot bypass a check it never had the authority to perform.

### 7.7 The semantic capability layer and the capability-oriented web

Agents should not operate the DOM when a higher-level semantic capability can be exposed. `SearchWeb`, `ReadPage`, `FollowLink`, `FillForm`, `SubmitForm`, `DownloadArtifact`, `Authenticate`, `Purchase`, `Schedule`, `ExtractData` are the right granularity; click-and-type on DOM nodes is a fallback for legacy sites. The pattern is the one browsers already follow when higher-level Web APIs abstract lower-level platform mechanics.

This generalises into a view of the web's evolution. Web 1 addressed documents; Web 2 addressed applications; an AI-native web addresses capabilities. The historical flow user → URL → application → UI → action becomes user → intent → capability discovery → runtime orchestration → tools, services, models, data → outcome. The browser abstracted away the operating system and the network needed to reach applications. An AI browser abstracts away the applications themselves. UI does not disappear; it becomes a situational output modality, while intent becomes the primary computational input. Existing websites function as legacy application interfaces driven through automation. AI-native services expose structured capabilities directly:

```text
Capability: SearchFlights
Inputs:      origin · destination · date · passengers
Constraints: max_price · preferred_airlines
Permissions: user_approval_required
Outputs:     flight_options
Evidence:    source · timestamp
```

Human interface and machine capability interface coexist. URLs do not vanish; they become one resource-addressing mechanism inside an intent-oriented runtime.

### 7.8 What is reused and what must be invented

| Reused from the browser | Invented for intelligence |
|---|---|
| Processes, workers, IPC | IOM and intent events |
| Networking, storage, caching | Context engine |
| Permissions, sandboxing | Model router |
| Extensions, tabs, profiles | Epistemic state |
| DevTools | Verification and provenance |
| | Capability negotiation |
| | Agent lifecycle and intent scheduler |

Hence: AI browser = browser runtime + intelligence runtime, not browser + chatbot.

---

## 8. Frontend Engineering as Reference Discipline: Build Time and Run Time

### 8.1 Why frontend engineering is relevant

Part of the motivation for this work was the concern that code generation makes frontend engineering obsolete. The conclusion is more specific. Code generation does not eliminate systems architecture; it moves the value of the discipline upward, from producing UI code to designing composable runtime abstractions. And the frontend discipline has spent two decades solving exactly the problems an AI harness now encounters: decomposition and composition, state management, state machines, events, routing, lazy loading, caching, performance budgets, permissions, accessibility, design systems, observability, declarative configuration and progressive enhancement.

The prerequisite research result treated this as one undifferentiated analogy. It is more useful to split it into the two phases every frontend engineer already thinks in: what happens at **build time** (resolving, checking, composing, testing, packaging) and what happens at **run time** (loading, injecting, routing, rendering, reacting, observing). The split matters because the two phases have different maturity on the AI side, and because build-time tooling is where the AI ecosystem is currently weakest.

### 8.2 The two timelines side by side

```text
FRONTEND                                     AI HARNESS
──────── build time ────────                 ──────── build time ────────
source + package.json                        agent manifest + skill / context packages
→ resolve from registry (npm) + lockfile     → resolve from registry + lockfile        [largely missing]
→ type-check (tsc)                           → contract/schema check (capabilities, IOM, policies)
→ lint (eslint)                              → harness lint (policy, budgets, versions)  [Section 5.5]
→ bundle (vite/webpack): tree-shake, split   → compose presets and skill bundles; prune; lazy-split
→ test (jest, playwright, snapshots)         → evals: replay from event store, golden runs, snapshots
→ artifact (dist/)                           → versioned, signed agent release

──────── run time ──────────                 ──────── run time ──────────
load → DI resolve → route → render           load → capability resolve → schedule → execute
→ store / selectors → effects → events       → canonical state / context selectors → tools, models → events
→ reconcile DOM                              → project IOM
→ DevTools / Web Vitals                      → Intelligence DevTools / AI Vitals
```

### 8.3 Build-time analogies

**Package management and registries.** This is the largest gap. Frontend development is unimaginable without npm: scoped and namespaced packages, semantic versioning with declared ranges, lockfiles that pin an exact dependency tree, `peerDependencies` for things the host must provide, `engines` constraints, integrity hashes, public and private registries, provenance attestations linking a published package to its source commit and build, and `npm audit` against a vulnerability database. The AI side has pieces of this: tool interface standards such as the Model Context Protocol, plugin and skill catalogues attached to particular products, prompt and agent files committed to repositories. None of them yet offers the properties that make npm trustworthy at scale. The design here assumes a **skill registry** with:

- A manifest declaring provided capabilities, required permissions, required model capabilities (as *peer* dependencies: the host supplies the model, the skill does not bundle it), context providers contributed, configuration presets, risk tier and audit events.
- Semantic versioning for skills, prompts and context presets, with a lockfile pinning the exact set an agent release was evaluated against. Without a lockfile, two runs of "the same agent" are not the same agent.
- Integrity hashes and provenance attestations, so that a skill's origin and build are verifiable. Prompt injection via dependency is the AI analogue of a compromised npm package, and the defences are the same shape.
- An audit database for skills: known injection vectors, over-broad permission requests, deprecated model dependencies.
- Evaluation results published alongside the package, in the role test results and coverage play today.

The proposed `@context/*`, `@skill/*` and `@model/*` package namespaces in the prerequisite research result are the beginning of this. What turns a naming convention into an ecosystem is the registry and the lockfile.

**Module resolution and bundling.** Bundlers resolve an import graph and emit optimised artifacts. Three bundler techniques transfer. *Tree-shaking* removes code never reached: the analogue is pruning context presets and skill bundles to what an agent's configuration actually requests, which doubles as token-budget control at build time. *Code splitting* defers loading until needed: context providers and skills that are expensive to initialise (indexing a repository, authenticating against a service) load lazily on first request. *Source maps* map emitted code back to its source: provenance (Section 4.7) is a source map from an output back to the context fragments and events that produced it.

**Compilation and type systems.** TypeScript turns runtime failures into build-time errors by checking contracts. Capability schemas, IOM fragment schemas and agent configuration schemas can be checked the same way, and `browserslist`-style target declarations have an analogue in **model capability targets**: an agent package declares the minimum model capabilities it requires (tool use, structured output, vision, a context length), and the build fails or degrades if the configured model pool cannot satisfy them. The frontend answer to a missing platform feature is a *polyfill*; the runtime analogue is a *capability shim*, such as emulating structured output through a parsing tool for a small model that lacks it natively.

**Static analysis.** Covered in Sections 5.4 and 5.5: ESLint as the architecture for declarative context composition, and linting the harness configuration for policy violations before execution.

**Testing and continuous integration.** Unit, integration and end-to-end tests have direct counterparts in evals at different granularity: a capability in isolation, an agent against a fixed context, an intent end to end. Two frontend testing ideas are worth importing specifically. *Snapshot tests* diff a rendered output against a stored baseline; context snapshot tests diff the constructed context for a reference task, catching silent changes in what a model receives. *Record-and-replay* end-to-end tests record network traffic and replay it; the event store makes this native: a recorded intent can be replayed against a new model version with every tool result fixed, isolating the model's contribution. Release gates in CI then become eval gates.

**Design systems and component sandboxes.** A design-system component has a stable API, variants, states, composition rules, accessibility expectations and an implementation hidden behind a contract. A capability has the same anatomy (Section 3.4), and the implementation (local model, cloud model, API, tool) can change without the agent noticing. Storybook, the component sandbox, has an analogue in a **capability and agent playground**: run one capability in isolation, in each of its states, with controlled inputs, before composing it into an intent.

### 8.4 Run-time analogies

**Component model.** UI component → agent or capability; design-system component → agent archetype or reusable skill; props → task and context contract; local component state → agent execution state. These were in the prerequisite research result and hold without modification.

**State management.** Global store → canonical runtime state; selectors → context selectors (Section 5.1); actions and commands → runtime commands; reducers → projections; middleware → policy, validation and logging between proposal and command. The CQRS vocabulary and the Redux vocabulary describe the same shape, which is one reason frontend engineers find event sourcing natural.

**Dependency injection.** Section 6.3. The injector hierarchy maps onto runtime scopes, the model router is a factory provider, and interceptor chains are policy middleware.

**Routing.** Frontend routers are more than URL matching. A route configuration declares *guards* (may this route activate?), *resolvers* (fetch this data before activation) and *lazy loading* (load this module on first navigation). Task routing in the runtime has the same three hooks: policy guards (does this task require approval before any worker starts?), context resolvers (construct this context before the agent activates) and lazy skill loading. Expressing them declaratively in the agent configuration, as route configurations do, is the point.

**Error boundaries and suspense.** An error boundary contains a failure to a subtree and renders a fallback. Agent failure containment is the same: a failed worker is contained to its subgoal, the IOM records `AgentFailed` and `SubgoalBlocked`, and the scheduler decides on retry, escalation or fallback without the whole intent collapsing. Suspense renders a placeholder while asynchronous data loads; the runtime equivalent is a session that can display partial IOM state while expensive context providers resolve.

**Reconciliation and reactivity.** A virtual DOM diffs the desired tree against the current one and applies a minimal patch; fine-grained reactive systems (signals) notify only the subscribers of a changed value. Both transfer to the IOM. Projections compute the minimal change set between successive IOM states; agents subscribe to the IOM slices they care about (a verifier subscribes to `ArtifactCreated` on its subgoal, nothing else) rather than re-reading the whole graph. This is what makes context construction cheap enough to run on every step.

**Hydration and replay.** Server-side rendering ships markup, and the client *hydrates* it into live state. Resuming an intent session is hydration from the event store: replay events, rebuild the IOM, re-attach agents. A paused tab and a crashed tab recover the same way.

**Caching and staleness.** HTTP caching semantics and *stale-while-revalidate* (serve the cached value, refresh in the background) map onto epistemic state with unusual precision. An evidence record has `observed_at` and `verified_at`; a context provider can serve it as stale while a verification worker refreshes it, and the IOM marks it `stale` or `contradicted` on return. Memoisation of selectors is the analogue of memoising context fragments keyed on the IOM slice they depend on.

**Lazy loading, module federation and micro-frontends.** Module federation lets independently deployed applications share modules at run time, and micro-frontend architectures compose independently owned UI fragments into one shell with isolation between them. The intent workspace is the shell; agents from different skill packages, possibly different vendors, compose inside it with isolated scopes, shared only through the IOM. The runtime-shared module is the installed skill, loaded once and resolved into every scope that requests it.

**The accessibility tree.** This is the least obvious and perhaps the most instructive analogy. Browsers maintain, alongside the DOM, an accessibility tree: a semantic representation of the page consumed not by rendering but by machines (screen readers, automation). ARIA attributes let authors enrich it where the visual DOM is insufficient. The semantic capability layer of Section 7.7 is an accessibility tree for agents, and the capability declarations an AI-native website would publish are its ARIA. Two decades of accessibility engineering, including the hard lesson that semantics must be authored and cannot be reliably inferred from visuals, apply directly.

**Progressive enhancement** → progressive compute escalation (Section 6.4). **Observability** (Web Vitals, real-user monitoring, DevTools) → AI Vitals and Intelligence DevTools (Chapter 9). **Feature flags** → policy toggles and model canaries: route a percentage of tasks to a new model version and compare evidence quality before promotion.

### 8.5 Maturity gap analysis

| Concern | Frontend ecosystem | AI harness ecosystem (as observed) | Needed |
|---|---|---|---|
| Package distribution | Registries, scoped packages, semver, lockfiles, provenance, audit | Product-specific catalogues, repository-committed files, interface standards without packaging | Skill registry with manifest, peer model dependencies, lockfile, attestations |
| Static checking | Type systems, linters, schema validation | Mostly absent; errors surface at run time | Contract checking, harness linter |
| Declarative composition | ESLint/bundler configs, route configs | Prompt files, ad-hoc YAML | Layered context configuration (Section 5.4) |
| Dependency resolution | Hierarchical DI, factory providers | Hard-coded model and tool references | Scoped capability resolution with attenuation (Section 6.3) |
| Testing | Unit, integration, e2e, snapshot, record/replay | Evals, often unversioned and non-reproducible | Event-store replay, context snapshot tests, eval gates |
| Failure isolation | Error boundaries, process isolation | Whole-loop failure | Per-subgoal containment via IOM |
| Debuggability | DevTools, source maps, time-travel debugging | Transcript reading | Intelligence DevTools over an event-sourced IOM |
| Performance budgets | Bundle budgets, Web Vitals | Token counting | AI Vitals as SLOs |

The pattern in the right-hand column is consistent: the AI ecosystem has run-time behaviour without build-time discipline. The frontend ecosystem acquired that discipline in response to scale and breakage; the AI ecosystem will need it for the same reasons.

### 8.6 Where the analogy fails

Three things have no frontend precedent and must be designed from first principles. *Probabilistic execution*: a component renders the same output for the same props; a model does not, so verification, confidence and retry are structural rather than exceptional. *Epistemic state*: frontend state is true by construction; runtime knowledge is believed with evidence and may be contradicted. *Intent as object*: no frontend abstraction represents what the user is trying to achieve; the IOM is new. Importing frontend patterns without adding these three produces a well-engineered harness that is still wrong about the nature of its workload.

---

## 9. Observability and Debuggability

### 9.1 Intelligence DevTools

An event-sourced IOM makes a DevTools analogue not just possible but almost free. Proposed panels:

| Panel | Shows |
|---|---|
| Intent Graph | Goals, subgoals, dependencies, blockers, completion status |
| Event Timeline | Ordered events with causality; scrubbing reconstructs the IOM at any point |
| Agent Graph | Live agents, their states, workers, escalations |
| Context Inspector | Exactly what a model received: identity, goal, files, decisions, retrieved knowledge, tool results, constraints, token budget; the resolved context configuration (`--print-config` analogue) |
| Model Inspector | Selected model, selection reason, capabilities, latency, tokens, cost, retries, escalation history |
| Memory Inspector | Contents and tier of each memory layer |
| Capability Inspector | Granted, requested, denied capabilities and their policies |
| Decision Inspector | Proposals, evidence, rationale, acceptance |
| Resource / Cost Inspector | Budgets consumed per intent, agent, model |

### 9.2 A timeline

```text
10:31 IntentCreated         10:34 TestExecuted
10:31 GoalAdded             10:35 TestFailed
10:32 PlanCreated           10:35 VerificationAgentSpawned
10:32 AgentAssigned         10:36 ContextExpanded
10:33 ContextBuilt          10:36 ModelEscalated
10:33 ModelInvoked          10:37 TestPassed
10:34 CodeGenerated
```

Every line is a replayable point. Clicking `ModelEscalated` shows which decision model recommended escalation, on which evidence, under which budget.

### 9.3 AI Vitals

Web Vitals turned user-perceived performance into a small set of measurable, budgetable metrics. The runtime equivalent, candidates for service-level objectives:

```text
task success rate · context size · inference latency · tool latency · model cost
retry rate · escalation rate · verification rate · correction rate
human intervention rate · energy consumption
```

These are only measurable because the runtime emits events for every stage. A transcript-based system cannot compute a retry rate.

---

## 10. Prototype Strategy

### 10.1 Browser core selection

| Core | Conceptual fit | Role |
|---|---|---|
| Chromium + CEF | Very high | Preferred prototype base |
| WebKit | High | Architectural alternative |
| Gecko / Firefox | High | Research alternative |
| Raw V8 / JavaScriptCore | Low | An engine, not a platform |

Chromium with the Chromium Embedded Framework is preferred because it provides a mature browser runtime while letting an external host application own the intelligence layer. The governing rule: do not fork a browser first, and do not put runtime logic into Chromium's C++ internals. Build the intelligence runtime as a separate host subsystem in a fast-iterating environment (TypeScript), integrate through explicit APIs and IPC, and only later, if proven, promote intelligence to a first-class browser process.

### 10.2 Host architecture

```text
┌──────────────────────────────────────────────────────────────┐
│                       AI Browser Host                        │
│  ┌────────────────────────────────────────────────────────┐  │
│  │  Intelligence Runtime (TypeScript)                     │  │
│  │  IOM · event store · scheduler · context engine        │  │
│  │  agents · model router · memory · skills · verifier    │  │
│  └───────────────────────────┬────────────────────────────┘  │
│                              │ browser adapter / IPC         │
│  ┌───────────────────────────┴────────────────────────────┐  │
│  │  CEF / Chromium                                        │  │
│  │  tabs · pages · DOM · storage · networking             │  │
│  │  workers · sandbox · permissions                       │  │
│  └────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────┘
Persistence: SQLite (intents, goals, agents, events, artifacts, decisions, evidence)
Models:      local model runtime + optional cloud router
```

### 10.3 Phases

1. **IOM and runtime.** Intent, goals, subgoals, agents, events, state, scheduler. No browser integration. Proves the abstraction.
2. **Context engine.** Providers, selectors, memory tiers, knowledge, event-derived context, declarative context configuration.
3. **Model router.** Local small, local large, cloud; capability profiles; escalation; budgets.
4. **Browser integration.** CEF: tabs, pages, navigation, read, search, forms, downloads, authentication.
5. **Semantic capability layer.** `SearchWeb`, `ReadPage`, `FillForm`, `SubmitForm`, `DownloadArtifact`, `Schedule`, `Purchase`.
6. **Intelligence DevTools.** Intent graph, event timeline, agent graph, context, model, memory and capability inspectors.

### 10.4 Runtime shape

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

interface Agent      { id: string; capabilities: CapabilityRequirement[]; state: AgentState;
                       execute(task: AgentTask): Promise<AgentResult>; }
interface Model      { id: string; capabilities: ModelCapabilities;
                       invoke(request: ModelRequest): Promise<ModelResult>; }
interface ModelRouter{ select(req: ModelRequirements, policy: ResourcePolicy): Promise<Model>; }
interface Capability { id: string; description: string; inputSchema: unknown; outputSchema: unknown;
                       permissions: Permission[]; risk: RiskLevel;
                       execute(input: unknown): Promise<unknown>; }
```

The flow these interfaces enforce: agent requests capability → router selects → model computes → runtime records → event.

### 10.5 Declarative agent manifest

The prerequisite research result's YAML configuration, extended with the build-time concerns of Chapter 8 (versioned dependencies, model capability targets, lockfile reference):

```yaml
agent:
  name: coding-agent
  version: 1.4.0

dependencies:                      # resolved from a skill registry, pinned by lockfile
  "@skill/filesystem": ^2.1.0
  "@skill/git": ^3.0.0
  "@skill/test-runner": ~1.2.0
  "@context/repository": ^1.0.0
  "@context-config/coding": ^0.9.0

peerModelCapabilities:             # supplied by the host's model pool, not bundled
  toolUse: required
  structuredOutput: required
  minContextTokens: 32000

context: ./context.config.js       # Section 5.4

models:
  candidates: [local-coder-small, local-coder-large, cloud-coder]

policy:
  privacy: local_preferred
  max_context_tokens: 32000
  max_cost: 0.50
  verification: required
```

### 10.6 Twelve-factor mapping (operations layer)

| Factor | Runtime interpretation |
|---|---|
| Codebase | Agent and runtime definition |
| Dependencies | Models, skills, tools, context providers (declared, pinned) |
| Config | Routing, context, policy, privacy, budgets |
| Backing services | Event store, vector store, model server, browser, filesystem, cloud models |
| Build / release / run | Package and evaluate → release → execute |
| Processes | Disposable inference and agent workers with external durable state |
| Port binding | Capability interfaces |
| Concurrency | Parallel agents, model and tool calls |
| Disposability | Disposable worker and model instances |
| Dev/prod parity | Same runtime, model and context contracts across environments |
| Logs | Agent, model, tool and context events |
| Admin processes | Reindex, replay, rebuild projections, evaluate models |

One caveat: loaded models and KV caches hold substantial run-time state, so "stateless process" must be read as "durable semantic state is external", not "no state in memory". Twelve-factor governs deployment; statecharts, CQRS, context and the IOM govern behaviour.

---

## 11. Discussion

### 11.1 Related concepts

The architecture is not built from nothing, and naming its ancestors sharpens what is new. The IOM as shared coordination substrate is a blackboard architecture with validated writes. Isolated workers communicating through events are an actor model. The agent as role plus state plus policies has affinities with belief-desire-intention agents, and the epistemic state record is a pragmatic version of belief maintenance. Durable, replayable orchestration resembles workflow engines. Tool interface standards such as the Model Context Protocol address the capability interface and are complementary; they do not address packaging, resolution, scheduling or the IOM. External model-routing services address one subsystem and should remain one.

### 11.2 Risks and open questions

- **Over-engineering.** A full event-sourced runtime is heavy for a single-agent task. The phased strategy mitigates this, and a minimal v0 should be measured against a plain agent loop on real tasks before the architecture is extended.
- **The IOM as bureaucracy.** If every trivial step requires a validated command, latency and cost rise. The remedy is granularity: not every token is an event, but every state change is.
- **Scheduling intelligence.** A scheduler that reasons about allocation is itself probabilistic at the margin. Keeping the decision proposal and the deterministic enforcement separate is the only defensible structure, but the quality of allocation decisions is an empirical question.
- **Standardisation.** The DOM succeeded because browser vendors converged under competitive and standards pressure over many years. An IOM standard would need the same, and the incentives for runtime vendors are not obviously aligned.
- **Capability-oriented web security.** Machine-readable capability declarations are an attack surface. The attenuation rule of Section 6.3 and the risk-tiered approval policy of Section 7.6 are necessary, not sufficient.
- **Registry governance.** A skill registry inherits every supply-chain problem npm has, plus prompt injection. Provenance and audit tooling must exist from the start, not be retrofitted.

### 11.3 Governance relevance

The properties this architecture produces as engineering by-products are the properties regulated environments require: a complete, replayable record of inputs and decisions; separation between a probabilistic proposer and a deterministic authoriser; explicit, risk-tiered approval for consequential actions; provenance for every artifact; and verification independent of generation. For organisations operating under AI, data-protection and operational-resilience regulation, an event-sourced intent runtime is closer to auditable by construction than any transcript-based harness can be. This is an argument for the design, not a claim of compliance, which depends on far more than architecture.

### 11.4 Threats to validity

Analogy-driven design risks importing solutions to problems one does not have and missing problems the reference system never had. Section 8.6 names the second risk explicitly. The first is addressed by the phased prototype: every imported mechanism must earn its place against a measured baseline. The thesis is also the product of a single design conversation; its claims about ecosystem maturity (Section 8.5) reflect the state of that discussion and should be re-examined as tooling evolves.

---

## 12. Conclusion

The argument converges on ten theses:

1. **Model size is not the whole intelligence problem.** Useful intelligence is model plus context plus tools plus memory plus state plus planning plus verification plus orchestration.
2. **Agents are not models.** Agents are runtime objects with lifecycle; models are compute resources.
3. **Context is constructed, not dumped.** It is a task-specific, policy-governed projection of a larger state and knowledge environment.
4. **Memory is a topology.** Different information needs different persistence and retrieval semantics.
5. **Events are first-class.** Durable temporal history, not final outputs, is the source of truth.
6. **Browser architecture is a valid reference.** It has already solved most of the infrastructure problems of a general-purpose intelligence environment.
7. **Intent should be a first-class runtime object.** That is the purpose of the IOM.
8. **The IOM can be the DOM of an intelligence platform.** A shared semantic substrate between agents, runtime, tools, models, UI and DevTools.
9. **An AI browser is not browser plus chatbot.** It is browser runtime plus intent runtime plus capability layer plus model runtime.
10. **Frontend engineering remains relevant at the abstraction level, in both phases.** Its run-time patterns (components, stores, selectors, state machines, DI, routing, error boundaries, reconciliation, caching) transfer with modification; its build-time discipline (registries, lockfiles, type checking, linting, bundling, testing) is what the AI ecosystem most lacks.

In one sentence:

> The proposed AI browser is a browser-like intelligence runtime in which each tab represents a user intent, the Intent Object Model is the shared, event-sourced semantic object graph for that intent, agents and workers collaborate through state and events, context is constructed declaratively from memory, knowledge and state, capabilities are resolved and negotiated through explicit, scoped interfaces, models are interchangeable compute resources selected by a scheduler and router, and the entire system is observable and auditable through intent-oriented DevTools.

The deepest abstraction is therefore not the LLM, the agent, the page or the chat transcript. It is the intent, and the runtime that holds it.

---

## Appendix A. Glossary

| Term | Meaning |
|---|---|
| Intelligence runtime | Persistent execution environment coordinating AI computation |
| AI browser | A browser-like runtime organised around user intent |
| Intent platform | General platform for intent-driven execution |
| IOM (Intent Object Model) | Shared, inspectable, event-sourced object graph of an active intent |
| Intent object graph | The graph interpretation of the IOM |
| Intent workspace | User-facing tab or session for one active intent |
| Agent | Stateful runtime role with lifecycle, policies and capability requirements; not a model |
| Worker | Isolated unit of concurrent execution |
| Capability | Explicitly declared, schema-bound, permissioned executable function or resource |
| Skill | Versioned package bundling capabilities, providers and presets |
| Capability negotiation | Scored resolution of a capability requirement against candidate implementations |
| Context provider | Adapter supplying context fragments from a source |
| Context engine | Composes fragments into a model-specific context snapshot under policy |
| Context configuration | Layered declarative specification of providers, fragment rules, processors and budgets |
| Harness linter | Static checker for agent configurations and policies |
| Model router | Selects a compute resource under requirements and policy; a factory provider |
| Model capability target | Minimum model features an agent package requires from the host |
| Event store | Durable, append-only temporal execution history |
| State store | Current truth projected from events |
| Memory | Historical and learned information, tiered |
| Knowledge | External, retrievable information |
| Epistemic state | Explicit record of proposition, value, confidence, evidence and verification |
| Provenance graph | Links from an artifact to its producer, model, context, evidence and events |
| Scheduler | Allocates workers, models, tools and budgets; reacts to events |
| Verification agent | Independent validation role |
| Intelligence DevTools | Inspection and debugging environment over the IOM and event store |
| AI Vitals | Runtime metrics usable as service-level objectives |
| Profile | Persistent personal intelligence environment shared by all sessions |
| Semantic capability layer | High-level browser and service capabilities exposed to agents instead of raw DOM operations |
| Skill registry | Versioned, attested package distribution for skills, providers and presets |