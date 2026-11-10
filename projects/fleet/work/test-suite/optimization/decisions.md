# Optimization decisions

## Reaping asserted by pid (decision:test-suite/optimization/reaping-asserted-by-pid)

The reaping tests assert by pid that the descendant is gone, rather than that its marker is still absent after 0.7 s,
because an absent marker proves reaping only on a machine fast enough to have written it by then.

## The client timeout as a keyword (decision:test-suite/optimization/client-timeout-keyword)

`daemon_holding` takes its client timeout as a keyword, and the silent-daemon test passes 0.25 s rather than waiting out
`LOCAL_CLIENT_TIMEOUT`, because that test's subject is that silence fails closed, not the constant's value, which is
asserted where it is defined if anywhere.

## A committed durations table (decision:test-suite/optimization/committed-durations-table)

The split reads a committed `tests/durations.json` rather than the latest run's artifacts, because every shard has to
compute the same partition and so cannot read times a shard has not seen.

## The UI job runs in the gate (decision:test-suite/optimization/ui-job-in-gate)

The UI job moves from `upper-tiers.yml` to `gate.yml`, beside `frontend`, because `pr.yml` calls the two workflows side
by side and a job in one cannot wait on a job in the other; beside `frontend`, the board is built once and the UI tests
start the moment the build exists.
