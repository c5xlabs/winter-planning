# Container candidates

Six candidates rewired the same miniature of the hub's hardest shapes — read and write ports served by one store, two
clocks of the same type, tenants built on first use and closed on demand, a background pass that runs outside any
request — against a behavioral contract none of them was allowed to change:
[dishka](https://github.com/reagento/dishka), dependency-injector, wireup, svcs, injector, and FastAPI's own `Depends`
with no library at all. Every candidate passed. They differed in what they cost.

dishka was the only container that did everything on its public interface and left the domain untouched. A route names
the one service it uses, `FromDishka[FindingExitService]`, rather than the bag. The whole graph between services is
checked when the container is built, before a request is served. A tenant is a child container that shares the process's
engine and broker and keeps its own stores, and closing it tears down that tenant alone.

The others fell away for reasons specific to blizzard:

- dependency-injector silently ignores its own markers under `from __future__ import annotations`, which all 46 of
  blizzard's routing modules use, and gives the type checker almost nothing to check.
- wireup reaches a long-lived tenant only through internal interface, on a project with one maintainer.
- injector needs more wiring than writing it by hand.
- svcs is a lookup table rather than a container and checks nothing before first use.

dishka's weaknesses, known going in:

- It resolves by type, so two collaborators of the same type can be swapped without a sound: a throttle registered the
  easy way was handed the virtual clock instead of the wall clock, with no error.
- It moves the question "is everything wired?" from the type checker, which answers it today before anything runs, to a
  test that builds the container, and it does not check what only a route asks for.
- Its child containers take no lock unless given one; without it, thirty-two concurrent first requests built sixteen
  stores for one tenant.
- It ignores constructor defaults, which fifteen of the eighty-two classes in the hub's bags carry.
- It is pre-2.0 and largely one maintainer's project, with a deprecation release already open before the major.
