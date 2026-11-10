# Mission

Blizzard is an **orchestration platform for fleets of coding agents**: work flows in from a backlog, is executed
unattended by agents in isolated environments, and flows out as reviewable deliveries — with a human interrupted only
when an agent is genuinely blocked.

The core of blizzard is **interoperability**. It is deliberately not tied to one workspace framework, one user workflow,
one forge, or one project-management system. Every external dependency is a named seam that a provider plugs into:

| Seam               | What plugs in                                                                                                                  | Reference binding (first implementation)                                                |
| ------------------ | ------------------------------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------- |
| **Workspace**      | Provides isolated execution environments                                                                                       | winter feature environments (polyrepo-native)                                           |
| **Work source**    | The project-management system holding the backlog — specific items ingested by id into the hub, never into runners             | GitHub issues                                                                           |
| **Coding Harness** | The coding agent that does the work                                                                                            | Claude Code, then OpenCode and Codex                                                    |
| **Workflow**       | The definition of how work moves — user-defined graphs of nodes, judgements, and gates, so anyone can shape any agent workflow | Hub-defined YAML workflow graphs; the default build → review → deliver graph            |
| **Delivery**       | Integrates finished work — the workflow graph's deliver node, executed at the hub                                              | Merge to the main branch; a GitHub pull request where a human-review gate is configured |
| **Human channel**  | Reaches people for questions, escalations, visibility                                                                          | the board + a chat bot                                                                  |

The **reference stack** — winter + GitHub + Claude Code — is the proving ground. That is a binding, not an identity: a
design choice that would weld the platform to any single binding is a mission violation, not an implementation detail.

The baseline involves no human at all: agents build, review, and deliver the work, and a human gate is opt-in
configuration that a person dials on node by node as their trust allows ([vision](./vision.md)).

## Scope filter

A proposed feature belongs in blizzard when it carries backlog work through unattended execution to a landed or
reviewable delivery, or brings a human in only where an agent is genuinely blocked or a gate they configured asks for
them, and when it does so against a seam's interface rather than one concrete binding. It does not belong when it would
define work instead of the work source, run an agent loop instead of the harness, manage an environment's internals
instead of the workspace provider, run CI, decompose a feature inside a node, or hard-code a review requirement or its
absence.

When a "should blizzard do X?" question comes up, the answer is here — in this test, the success criteria, or the
non-goals — or this document is wrong and must be amended first.

## Success criteria

Blizzard succeeds when all of the following hold:

1. **Unattended throughput.** The operating engineer (`persona:application-architect`) queues work, walks away
   (overnight, a weekend), and returns to landed work — merged to main, or parked at exactly the human gates they
   configured — or precise escalations. Never to a wedged fleet.
2. **Zero duplicated or clobbered work.** `persona:application-architect` never finds two agents holding the same task,
   and no reaped-but-still-running agent ever overwrites its successor's delivery. These are structural guarantees
   (atomic leases, epochs), not probabilistic ones.
3. **Crash-equivalence.** For `persona:application-architect`, away overnight, `kill -9`, reboot, or power loss at any
   instant loses at most in-flight LLM tokens — never queue state, never delivered work, never truthful status.
4. **Cheap human takeover.** When the fleet escalates, `persona:application-architect` lands in the stuck agent's full
   session context with one pasted command — not in a cold reconstruction of what the agent was doing.
5. **Interoperable by construction.** Every component works against a seam's interface, never its concrete binding — the
   reference stack is the first implementation of the interfaces, not a shortcut around them. Swapping a binding — a
   different forge, harness, or workspace provider — is an adapter's worth of work, never a rewrite, so
   `persona:harness-engineer` can bring the harness they choose; the promise is proven the first time a seam has two
   live bindings (harnesses will be first: Claude Code, then OpenCode/Codex).
6. **Supports the HITL→HOTL transition.** Not every persona can — or should — jump straight to trusting agents fully for
   every task; `persona:senior-se` may never hand some categories over at all. Blizzard meets each person where they
   are: human gates can be dialed on per workflow node — in the graph for the whole fleet, or in a runner's own
   configuration by node name for that operator — so the platform supports any persona's transition throughout their
   agentic-development career. Trust is earned gate by gate, never demanded as an entry fee.
7. **Flexible work shapes — never 1:1:1:1.** Existing orchestrators hard-code one task = one agent = one environment =
   one repo. Blizzard treats every one of those cardinalities as flexible: a chunk wraps one or more PM items; one agent
   works a chunk, optionally fanning out to subagents; a chunk may span one or more feature environments and one or more
   git repositories. For `persona:application-architect`, work touching four repos is one chunk; twenty small bugs may
   be one coordinator with subagents across four environments (how items are grouped into one chunk is the operator's
   call, reached with a model's help or without one — blizzard executes the shape it is handed and never computes it).
   The rigid 1:1 mapping stays available as the always-correct fallback — a floor, never a ceiling.

## Non-goals

Blizzard deliberately is **not**:

- **A task tracker.** The work-source provider (GitHub issues in the reference stack) owns the backlog; blizzard never
  becomes a second place where work is *defined*. The hub's chunks wrap PM items with execution state — a cache and a
  workflow position, never a competing backlog.
- **A coding harness.** Harnesses do the work; blizzard spawns, watches, and harvests them through dumb adapters. It
  never reimplements an agent loop.
- **An isolation layer.** The workspace provider (winter in the reference stack) owns worktrees, ports, and services.
  Blizzard consumes that guarantee; it never manages an environment's internals itself.
- **A CI system.** CI runs where it already runs; blizzard only routes CI *results* back to the owning session.
- **A task-layer workflow engine.** Blizzard's workflow graph governs a *chunk's lifecycle* — which station the work is
  at, who judges it, when it lands. Decomposing one feature into implementation phases *inside* a node is the territory
  of the workflow tooling below blizzard and of the harness itself. Blizzard is the fleet layer above them: which chunks
  run, where they are in their lifecycle, and who is alive.
- **An opinion about process.** Whether code merges straight to main or waits at a human gate is configuration, not
  doctrine. Blizzard ships the autonomous baseline and makes every human gate optional; it never hard-codes a review
  requirement — or its absence.
