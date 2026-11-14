# Winter component install requirements

For `epic:component-install`.

## Fetch a named release (product:install-component-version)

A person can fetch a specific published release of the Workspace, hub, or runner component for their machine.

Acceptance:

- `winter install workspace|hub|runner <version>` places that release for the current machine's platform in the shared
  version cache, after which `winter <role>` can run it when it is selected.
- Fetching a release already in the cache succeeds without downloading it again.
- Fetching does not change any workspace pin or role default.
- Releases of different components and versions fetched side by side do not overwrite one another.

## Fetch what the current selections need (product:install-selected-versions)

A person can fetch every release their current selections need without first working out which versions those are.

Acceptance:

- `winter install` with no component fetches the Workspace release selected for the current directory and the hub and
  runner defaults, skipping any already cached and any role with no selection.
- The result lists each selected version and whether it was fetched, already present, or failed.
- One failed fetch does not stop the others, and the command reports failure if any fetch failed.

## Trust what was fetched (product:verified-install)

A person can rely on a fetched release being the one that was published, and on the cache never holding a partial
download.

Acceptance:

- A release whose download does not match its published verification data is rejected and never becomes runnable.
- An interrupted or failed fetch leaves no cache entry that `winter <role>` would try to run.
- Two fetches of the same release running at once leave one usable entry.

## Explain fetch failures (product:explain-install-failure)

A person or agent is told why a release could not be fetched and what to change.

Acceptance:

- The error distinguishes an unknown version, a release not published for this machine's platform, a network failure,
  and a failed verification.
- A `winter <role>` invocation that stops because its selected version is not cached names the `winter install` command
  that would fetch it.
- Fetching needs network access only to download; every other `winter` command keeps working offline.
