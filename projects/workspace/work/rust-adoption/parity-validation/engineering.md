# Replacement validation engineering requirements

For `epic:rust-adoption/parity-validation`.

## What already holds

`epic:parallel-workspaces/component-runtime` provides what this epic's method stands on, and no more of that epic is a
prerequisite: the Cargo workspace and package edges of its repository layout spec, command registration by file
location, the two gates with the scenario test target, the architecture test, the test harness with its git fixtures and
workspace builder, and the contributor context tree. Its root options, logging, tracing, and the rest of its termination
taxonomy are not needed by anything here.

The contracts of the two proof commands are owned by `epic:parallel-workspaces/environment-lifecycle` and
`epic:parallel-workspaces/repository-catalog`, and the configuration and state rules they read by
`epic:parallel-workspaces/workspace-configuration`.

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

## One scenario, one directory (engineering:scenario-files)

A recorded scenario is one directory that holds its fixture description, its invocations, its comparison rules, and its
record, and nothing outside the directory changes when a scenario is added. The fixture is described as data and built
from that description identically by the record step and by replay; neither the reference tool nor the binary prepares
the state a scenario starts from. Serves `product:capability-accounting` and applies
`engineering:parallel-workspaces/component-runtime/additive-registration`.

Acceptance:

- Adding a scenario directory makes replay run it with no other tracked file changed.
- A scenario whose description is malformed, or names a key the format does not define, fails replay naming the scenario
  and the key.
- Building the same fixture description under two different roots yields equal normalized state, commit ids included.

## Record step apart from every gate (engineering:record-step-separation)

The reference tool runs only in the record step, which a person starts by an explicit command naming the reference
executable. No test, gate, build script, or CI job starts it. The record step runs the reference only inside a fixture
tree under a temporary directory, refuses a location from which a real workspace is reachable, and reads the reference
tool's checkout without writing to it. A scenario cannot direct either tool elsewhere: every working directory and
fixture path it names lies inside its own temporary tree. Applies
`engineering:parallel-workspaces/component-runtime/verification-gates`.

The source check below exists so that no gate and no test can start the reference tool, and it reaches no further than
that. The variable that carries the reference path is known to the record command alone. The reference tool's language
is unnamed in the code that builds fixtures, runs steps, records, and replays. The library and the binary are outside
the second half: the product names that language wherever a contract of its own does, such as an interpreter variable in
a child's environment or an extension's entry point.

Acceptance:

- With no reference executable named, the record step exits non-zero before building any fixture.
- The record step refuses a temporary root that has a workspace configuration file at or above it.
- A scenario that names a working directory or a fixture path that is absolute, or that leads out of its own temporary
  tree directly or through a symbolic link, is malformed; neither the reference tool at record nor the binary at replay
  is started for it.
- A record made at a reference commit other than the pinned one is refused; replay fails a scenario whose record names
  another commit than the pinned one.
- Recording a scenario that already has a record changes nothing unless re-recording is asked for by name.
- After a record, no file of the reference checkout's git directory, its index included, differs in bytes or
  modification time from before.
- A check over the sources fails when the binary, the library, or test support names the variable that carries the
  reference path, and when test support or the replay test target names the reference tool's language.

## Pinned scenario environment (engineering:pinned-scenario-environment)

The reference tool and the binary run a scenario under the same closed environment: an explicit variable list with
nothing inherited, a HOME, configuration, cache, and temporary directory inside the fixture tree, a git configuration
that reaches only fixture remotes, fixed git identities and dates, a fixed locale, timezone, and terminal size, and
closed standard input unless the scenario supplies one. A record states the environment it was made under, and a record
that is not stable under that environment is not stored. Applies `engineering:disposable-runtime-proof`.

Acceptance:

- A scenario recorded twice under two different roots yields records equal under the scenario's comparison rules, or the
  record step fails naming the first difference; text that replay never compares does not make a record unstable.
- Replay fails a scenario whose record carries another environment digest than the harness now produces.
- No recorded file contains the fixture root's real path, or the path of the reference tool's checkout.

## Deferred scenarios are counted (engineering:deferred-scenario-accounting)

A scenario that needs a facility the runner lacks — a terminal, a signal, a controlled clock, a long-lived process, a
provider — declares the facility. Replay lists it as deferred, counts it, and neither runs nor fails it; a scenario is
never skipped silently, and a facility the runner gains makes its scenarios run without editing them. Serves
`product:capability-accounting`.

Acceptance:

- A replay run prints the number of scenarios passed, failed, and deferred, and each deferred scenario with the facility
  it waits for.
- A scenario declaring an unknown facility fails replay.

## The method is proven on real behaviour (engineering:method-proof)

Before a body of commands is built against the scenario method, the method has carried one read-only behaviour and one
state-changing behaviour of the reference tool from record to passing replay, including a failure path, one registered
deviation, and one case in which a deliberately wrong implementation is caught. Under
`decision:rust-adoption/parity-validation/proof-behaviours`. The proof completes the groundwork every command epic
builds on. Capture of the calls a command makes to another program is outside it: no proof behaviour starts a program,
and call capture is built before the first epic whose behaviour does.

Acceptance:

- The full gate replays the proof scenarios and passes.
- For each proof behaviour, a test alters one observable — an exit code, a JSON value, or a byte of resulting state —
  and asserts that replay reports the scenario failed in that dimension.
- The state-changing proof includes a scenario that removes a block lying between two others with comment lines on both
  sides. Its record fixes which of those lines the reference tool removes, and its expectation, written under
  `decision:parallel-workspaces/repository-catalog/removal-comment-extent`, which of them the binary keeps.
- A test replays that scenario with an implementation that reproduces the reference record and asserts replay fails.

## Failure and concurrency evidence (engineering:adversarial-scenarios)

The validation suite injects Git failures, missing prerequisites, malformed extension output, occupied resources,
interruption, and competing operations on the same and different scopes. It verifies authoritative state and filesystem
effects as well as stdout, stderr, and exit status. Serves `product:isolated-adoption-proof`.

## Disposable runtime proof (engineering:disposable-runtime-proof)

Runtime scenarios use owned disposable workspace roots, HOME/cache locations, remotes, and provider resources, with
provisioning completed before services are exercised. Test cleanup is limited to resources created by the scenario;
failure retains diagnostics and cannot target a live development environment. Serves `product:isolated-adoption-proof`.

## Open questions

- The epic's present slices build the scenario method and its proof and stop there. `engineering:adversarial-scenarios`
  and the epic's two product requirements, `product:capability-accounting` and `product:isolated-adoption-proof`, are
  delivered by no slice yet. Engineering states, when the epic is next sliced, which of them a slice of this epic
  delivers and which is met by the scenarios each command epic adds.
- The two proof commands are built by this epic's slices, though their contracts are spec leaves of
  `epic:parallel-workspaces/environment-lifecycle` and `epic:parallel-workspaces/repository-catalog`. Engineering
  confirms, when either of those epics is sliced, that its slices extend what the proof built and do not build it again.
