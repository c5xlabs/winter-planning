---
epic: advanced-mocking
refinement: refined
status: planned
---

# Advanced mocking

A standalone epic; its requirements are in `requirements.md`. Research on how the mock fleet is composed, and on what a
second forge vendor would take, is in `research/`.

## The problem

Whoever builds blizzard, `persona:harness-engineer` or a `persona:fleet-agent` working a chunk, needs to see the product
holding a believable fleet before they can judge a change to it. The mock fleet already builds the pieces of one. The
mock-data CLI creates and resets state in domain terms, so an agent asks for a chunk parked on a question rather than
for rows, and every mock is a real service with levers that steer it into a named edge state. What it cannot build is a
whole world.

The two presets that come closest, `scenario board` and `scenario fleet`, are Python. Their shape is fixed in code, so
a new world means a code change, and they compose nothing: a tenant with one team's gardening routines, another's
graphs, and a third's backlog is not something either can be asked for. They write only the hub and runner stores, so a
chunk has no issue behind it and the board shows no title, description, or discussion, and the mock forge, which could
hold all three, starts every env empty. The text they do write is placeholder. Each one runs once against a clean store,
so adding to a world means resetting it and starting over.

That cost lands at a concrete moment. An engineer reworking the runner panel wants a screen that looks like a real
night: a handful of chunks with readable titles, one stuck on a question worth reading, one bounced on a conflict. The
mock fleet gives them six chunks with placeholder text, and the fastest way to anything better is to point a dev server
at the live fleet, which the workspace forbids for good reason.

## The outcome

A fixture is a YAML file, not code, that declares one kind of thing an env should hold: its projects and people, a
codebase, a backlog of issues with their bodies and threads, graphs, gardening routines, runners, or chunks in every
state the fleet can reach, each tied to the issue it came from. The file says which kind it is. Fixtures compose by
being applied one after another to the same env: a tenant's graphs from one file, its gardening from another, its
backlog from a third. The files the test suite, agents, and the demo share are kept beside the mock fleet that applies
them.

A fixture is applied to an env, not run once. Applying it reconciles toward creation: what the fixture declares and the
env lacks is created, what is already there is left alone, and applying the same fixture twice changes nothing. An
engineer adds to a fixture and applies it again to see the addition, without resetting first. Nothing is ever removed by
an apply; removing state is what the product's own routes are for.

The fixtures of one apply describe one world across every place its state lives, and the mock-data CLI keeps the hub
store, the runner store, the mock forge, and the git origins consistent with each other, so a chunk shows the forge
issue it came from and a chunk in delivery has the pull request and branch that go with it.

Fixtures are where a model helps and a script takes over. An agent writes a fixture once from a short brief, with
realistic titles, issue bodies, questions, and review threads. Applying it is deterministic, costs no tokens, and gives
the same world every time. An agent chasing one edge case writes a fixture of a few lines instead of a sequence of
commands. What only needs to look plausible, such as a forty-turn transcript or a week of usage, a fixture asks for by
its shape, and the CLI generates it the same way on every apply.

The `scenario` verbs stop growing. They are kept until fixtures can build what they build, then retired.

## Left for later

- A mock GitLab forge, and a GitLab work source, belong to `epic:advanced-delivery/gitlab`. This epic's fixtures name a
  forge's issues and pull requests without naming its vendor, so a second forge feeds from the same files.
- Worlds that play out over time, such as a night that wedges partway through, belong to `epic:demo`
  (`product:demo-scenarios`). A fixture here is the state a moment starts from.

## What this epic is not

- A replacement for `reset` and the `create` verbs. Fixtures are built on them; `reset` is extended to clear the mock
  forge and what earlier applies recorded, and the `create` verbs stay as they are.
- Real fleet data. Snapshots of the hosted hub's store are already the way to see that, and a fixture never copies from
  one.
- A way to seed a live fleet. Fixtures are applied to a feature env's own stores and mocks, never to a deployment.

## Depends on

- `product:config/sweep/constants-promoted`, which makes the hub's runner offline cutoff, `STALE_AFTER` today, a setting
  a feature env can raise, once `decision:config/sweep/constant-owners` does not keep it fixed.
  `product:runner-presence` waits on both. A feature env that raises the cutoff accepts that its own runner also reads
  online for up to four hours after it stops.
- `product:workspace/running-environments/resource-provisioning/required-service-preconditions`, which lets a
  provisioning step start the services it needs, so `winter provision` can apply the starter world
  (`product:starter-world`).
