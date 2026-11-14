# Replacement validation engineering requirements

For `epic:rust-adoption/parity-validation`.

## Contract observations, not Python coupling (engineering:behavioral-fixtures)

Executable tests assert final Rust command behavior using recorded legacy observations, source-derived fixtures, and
approved differences; they do not depend on importing Python implementation types or running Python in the routine Rust
verification loop. A recorded observation is compared by function, under `decision:functional-parity`. Serves
`product:capability-accounting`.

Acceptance:

- A recorded scenario stores the exit code, stdout, stderr, and the resulting filesystem and git state of one invocation
  against a fixture workspace. Where the invocation hands work to another program, it also stores the argument vector,
  working directory, and contract environment variables each such program received.
- Replay fails on a different exit code, on `--json` output that differs as parsed JSON (documents, and their order
  within a stream), or on different filesystem or git state.
- A JSON string value that the output formats spec classes as message text is compared for presence and type, not
  content.
- Where a command hands work to another program, replay fails on a different argument vector, working directory, or
  contract environment variable given to that program.
- Human output, help, and error and warning texts are stored for reference and never fail a replay.
- A registered deviation replays against an expectation written with its decision, not against the legacy record.

## Failure and concurrency evidence (engineering:adversarial-scenarios)

The validation suite injects Git failures, missing prerequisites, malformed extension output, occupied resources,
interruption, and competing operations on the same and different scopes. It verifies authoritative state and filesystem
effects as well as stdout, stderr, and exit status. Serves `product:isolated-adoption-proof`.

## Disposable runtime proof (engineering:disposable-runtime-proof)

Runtime scenarios use owned disposable workspace roots, HOME/cache locations, remotes, and provider resources, with
provisioning completed before services are exercised. Test cleanup is limited to resources created by the scenario;
failure retains diagnostics and cannot target a live development environment. Serves `product:isolated-adoption-proof`.
