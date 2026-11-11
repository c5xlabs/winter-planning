# Replacement validation engineering requirements

For `epic:rust-adoption/parity-validation`.

## Contract observations, not Python coupling (engineering:behavioral-fixtures)

Executable tests assert final Rust command behavior using recorded legacy observations, source-derived fixtures, and
approved differences; they do not depend on importing Python implementation types or running Python in the routine Rust
verification loop. Serves `product:capability-accounting`.

## Failure and concurrency evidence (engineering:adversarial-scenarios)

The validation suite injects Git failures, missing prerequisites, malformed extension output, occupied resources,
interruption, and competing operations on the same and different scopes. It verifies authoritative state and filesystem
effects as well as stdout, stderr, and exit status. Serves `product:isolated-adoption-proof`.

## Disposable runtime proof (engineering:disposable-runtime-proof)

Runtime scenarios use owned disposable workspace roots, HOME/cache locations, remotes, and provider resources, with
provisioning completed before services are exercised. Test cleanup is limited to resources created by the scenario;
failure retains diagnostics and cannot target a live development environment. Serves `product:isolated-adoption-proof`.
