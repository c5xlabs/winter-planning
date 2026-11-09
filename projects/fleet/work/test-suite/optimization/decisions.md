# Optimization decisions

## The UI job runs in the gate (decision:test-suite/optimization/ui-job-in-gate)

The UI job moves from `upper-tiers.yml` to `gate.yml`, beside `frontend`, because `pr.yml` calls the two workflows side
by side and a job in one cannot wait on a job in the other; beside `frontend`, the board is built once and the UI tests
start the moment the build exists.
