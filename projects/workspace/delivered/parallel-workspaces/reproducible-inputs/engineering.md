# Reproducible input engineering requirements

For `epic:parallel-workspaces/reproducible-inputs`.

## What already holds

The delivered `epic:parallel-workspaces/repository-catalog` built what this epic stands on: the program port and its
child environment
([child-processes.md](../../../delivered/parallel-workspaces/repository-catalog/spec/child-processes.md)), operation
spans through `cx.tracer`, `Diagnostics`, `RepoError` and `Termination::Repo`, `ReadFiles::is_dir`, and `drift::detect`
and `drift::notice_line`
([drift-warning.md](../../../delivered/parallel-workspaces/repository-catalog/spec/drift-warning.md)).

## Resolution record integrity across writers (engineering:resolution-write-integrity)

Lock writes use the delivered configuration write integrity, so concurrent writers updating different inputs cannot lose
each other's records, under `decision:parallel-workspaces/workspace-configuration/write-conflicts`. A checkout is
changed before its record is written, so an interruption leaves a record that the next materialization restores the
checkout to. Materialization, re-pinning, and freezing never treat malformed records as absent; the read-only pin
snapshot alone reads them as an empty lock, as the reference does. Serves
`product:parallel-workspaces/workspace-bootstrap/recorded-input-materialization` and `product:update-standalone-inputs`.

## Restore temporary work on every exit path (engineering:repin-preservation-lifetime)

Temporary preservation created for an update is restored after the checkout whether the resolution or checkout succeeded
or failed; a failed restoration leaves the preserved work in place and names it. An interrupt ends the invocation at
once, as for every command that runs no child it must outlive, and leaves the preserved work in the stash list. Serves
`product:update-standalone-inputs`.

## Content identity cannot depend on stat freshness (engineering:fingerprint-observation)

Fingerprinting detects same-size edits in one timestamp tick, accounts for staged-versus-working state, and preserves
the real Git index byte-for-byte. Object-store-only writes are disclosed in
[content-identity.md](./spec/content-identity.md), Git operations. Serves `product:workspace-content-identity`.
