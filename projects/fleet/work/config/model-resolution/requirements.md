# Model resolution requirements

Requirements of `epic:config/model-resolution`.

## The tier table (product:tier-table)

For each capability tier and each harness this runner has configured, the operator sees the adapter's built-in mapping
where one exists, the runner's own alias where the operator has set one, and the model that results. The
[terminal sample](../assets/model-resolution.txt) shows it on one worked configuration.

Acceptance:

- A tier a harness cannot resolve is shown as exactly that, since it is the reason a runner will pass over work
  demanding it.

## The effort table (product:effort-table)

Effort is shown in the same shape: the value as authored, the runner's alias where one renames it, and what reaches the
harness.

Acceptance:

- A value the harness will drop rather than honor, which today fails silently, is shown as dropped.

## The session pools (product:session-pools)

For the graphs this runner works, each declared session's harness set, model preference, and effort is resolved through
both tables to what each harness in its set would actually run. The pools follow selection all the way through, since a
table that stopped short would hand the operator the last and most error-prone step of the reconstruction this command
exists to spare them.

Acceptance:

- Every harness in a session's acceptable set is listed with what it would run.
- The harness the runner would actually choose is marked, with the reason each earlier harness was passed over.
- The chunk default is shown where it is the layer that supplies a field.

## Provenance on every cell (product:cell-provenance)

Each resolved value names the layer it came from (adapter default, runner alias, graph pin, chunk default, or adapter
fallback), so an operator scanning the table sees at a glance which answers they chose and which were chosen for them.

Acceptance:

- No resolved value appears without the layer that supplied it.

## Two readers, one answer (product:terminal-and-json)

The answer comes as a table a person reads at the terminal and as a JSON form an agent or a script reads, each carrying
every layer's value alongside the effective one rather than the result alone. Both give the answer the runner acts on
when it spawns a session, so the report never drifts from what runs. The [JSON sample](../assets/model-resolution.json)
shows the same answer as the terminal sample.

Acceptance:

- The terminal table and the JSON form name the same layers and the same effective values for the same configuration.

## The runner's own answer without the hub (product:answer-without-hub)

The tier and effort tables belong wholly to the runner, so they answer from the machine alone, with or without a hub.
The graphs and chunk defaults live on the hub, so the pools need it.

Acceptance:

- When the hub is out of reach, the command says so and still prints everything the runner owns.

## A named chunk (product:chunk-view)

Chunk defaults appear only when the operator names a chunk; without one, the pools read as the graphs declare them,
which is the view that holds for every chunk. Naming a chunk also brings in what that chunk is running right now on this
runner: a pool already alive keeps the model it was minted with, so an alias edited this morning changes the next
session, not the current one.

Acceptance:

- With a chunk named, a live pool's model is shown beside what the next session would run, so the operator sees the
  difference rather than discovering it in a transcript.
