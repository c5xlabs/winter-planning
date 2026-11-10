# Adopt dependency injection decisions

## Adoption batch size (decision:adopt-di/adoption-batch-size)

**Open.** How large each adoption landing is, decided when adoption begins: one landing per concept package is the
starting assumption, and a package whose services form a deep chain may want splitting.

## Cutover is declared (decision:adopt-di/declared-cutover)

**Decider:** the epic's owner

Cutover is called by the owner when adoption has emptied the builders and narrowing is near its end, rather than
discovered, so the old path is taken apart as fast as nothing rests on each piece of it.

## Narrow rather than split the bags (decision:adopt-di/narrow-not-split)

`TenantServices` is not split into per-concept sub-bags: a consumer that names exactly what it uses has no need for a
smaller bag, so narrowing supersedes splitting.

## Strangler migration (decision:adopt-di/strangler-migration)

The container grows around the hand-built graphs and takes them over one service at a time, with the old path removed
only when nothing rests on it, so at no point does the hub or the runner stop working. The hub goes first, because its
bags are the larger debt and its graph proves the shape; the runner host and the CLI follow.

## Container (decision:adopt-di/container)

blizzard adopts dishka as its dependency-injection container, because it was the only one of six candidates that met the
hub's hardest shapes entirely on its public interface, left the domain untouched, and checks the whole graph when the
container is built. Each of its known weaknesses is answered by a guard in this epic's engineering requirements; the
candidates, how each fared, and the weaknesses are in
[container candidates](research/container-candidates/candidates.md).
