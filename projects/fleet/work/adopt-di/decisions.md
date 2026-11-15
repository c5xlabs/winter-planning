# Adopt dependency injection decisions

## Adoption batch size (decision:adopt-di/adoption-batch-size)

Adoption is cut into bounded adapter, concept, or entrypoint changes, with adoption, consumer narrowing, and newly
available retirement together where their dependencies permit. Deep areas are divided by their dependency chains rather
than left as a phase containing unnamed future landings.

## Rolling retirement (decision:adopt-di/declared-cutover)

Each landing removes the bag fields, construction branches, and bridges whose last consumer it moves; a bag assembler
remains until its last consumer disappears. The closing change verifies the already-retired paths and tightens the final
gates rather than waiting for an owner-declared cleanup phase.

## Cached scopes and thin requests (decision:adopt-di/cached-scopes)

Tenant and runner graphs live in cached child containers; requests hold only their principal and request-local
dependencies, while background callers use the same cached graph without manufacturing a request. This preserves
existing service identity and isolation instead of rebuilding tenant services for every request.

## Synchronous resolution (decision:adopt-di/synchronous-resolution)

Container resolution is synchronous, with locked child containers; async operations remain methods on the resolved
services, and thread- or loop-confined resources retain their existing owners behind injected values or factories. A
small executable integration proof gates process activation, because switching all construction to an async container
would change the existing synchronous and cross-thread access paths.

## Narrow rather than split the bags (decision:adopt-di/narrow-not-split)

`TenantServices` is not split into per-concept sub-bags: a consumer that names exactly what it uses has no need for a
smaller bag, so narrowing supersedes splitting.

## Strangler migration (decision:adopt-di/strangler-migration)

The container grows around the hand-built graphs and takes them over one service at a time, with the old path removed
only when nothing rests on it, so at no point does the hub or the runner stop working. The hub's authenticated pilot and
shared HTTP/background identity proof precede host or CLI activation; after that proof, independent area migrations may
proceed in parallel.

## Container (decision:adopt-di/container)

blizzard adopts dishka as its dependency-injection container, because it was the only one of six candidates that met the
hub's hardest shapes entirely on its public interface, left the domain untouched, and checks the whole graph when the
container is built. Each of its known weaknesses is answered by a guard in this epic's engineering requirements; the
candidates, how each fared, and the weaknesses are in
[container candidates](research/container-candidates/candidates.md).
