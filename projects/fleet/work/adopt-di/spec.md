# Adopt dependency injection spec

## System

| Component                           | Where                          | Holds                                                                                                                                                                                                                                                                              |
| ----------------------------------- | ------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Registry interface                  | `foundation/`                  | Verbs to declare a service and its lifetime, a factory, a resource torn down when its lifetime ends, a value a lifetime is opened with, and an alias from a port to whatever serves it. Grows only when a real registration needs a verb; never passes dishka's own types through. |
| Translation module, one per process | per process                    | Turns registry declarations into dishka providers; offers overrides for tests. With the route edge, the only code importing dishka.                                                                                                                                                |
| Area wiring module                  | beside each domain package     | `register(registry, *, <port>=<adapter>, …)`: keyword arguments typed by the package's own ports.                                                                                                                                                                                  |
| Adapter registration                | store, forge, harness packages | Each concrete adapter registered once, with its lifetime.                                                                                                                                                                                                                          |
| Composition root, one per process   | per process                    | Reads configuration, chooses the concrete adapter for each configured seam, passes adapters to each area's `register`, builds the container with strict validation.                                                                                                                |
| Legacy module (transition)          | per process                    | Answers for today's bags and stores by asking the existing builders for the instances they already make.                                                                                                                                                                           |

The registry seen by a domain area has no way to override anything.

## Domain

| Process     | Scopes                     | Notes                                                                                                                                                                                    |
| ----------- | -------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Hub         | process → tenant → request | The tenant cache lives for the process and holds each tenant's child container; a tenant shares the engine and broker and keeps its own stores. Closing it tears down that tenant alone. |
| Runner host | host → runner              | Each runner is a child container of its host.                                                                                                                                            |
| CLI         | per command                | A command builds a container at the top of its root from the registrations of the areas its command group serves.                                                                        |

A collaborator confined to one thread stays hand-managed and enters the graph as a value or a factory.

dishka builds a service only when something asks for it, so a CLI verb needing a hub client never constructs the store
beneath a daemon; a command group registers only the areas it serves, so a worker verb never imports a daemon's classes.
The carrier a test hands a command becomes an override of the command's container.

```python
# A route names the one service it uses, not the bag:
async def handler(exits: FromDishka[FindingExitService]) -> ...: ...
```

## Flows

During the transition, a bag's field for an adopted service is filled by asking the container, so anything still reading
the bag reads the instance the container holds. Teardown runs in the reverse of build order: each tenant's connections
close before the shared engine is disposed.
