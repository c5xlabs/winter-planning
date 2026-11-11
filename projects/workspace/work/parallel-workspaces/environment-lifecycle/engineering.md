# Environment lifecycle engineering requirements

For `epic:parallel-workspaces/environment-lifecycle`.

## Cross-process allocation integrity (engineering:allocation-integrity)

Assignment, collision resolution, and release remain consistent across processes and interruption; independent readers
cannot observe a successful duplicate assignment. Existing corrupt state blocks allocation-changing effects until an
explicit recovery establishes ownership. Serves `product:stable-environment-allocation` and
`product:environment-state-recovery`.

## Verify worktree identity at the effect boundary (engineering:worktree-effect-confinement)

Creation and removal verify the intended workspace ancestry, Git common store, and worktree registration, including path
replacement between planning and execution. Discovery cannot rely solely on the first declared project. Serves
`product:environment-targets` and `product:environment-worktrees`.

## Preserve independent bootstrap outcomes (engineering:lifecycle-step-isolation)

Tracking-wiring failure remains attributable without suppressing independent lightweight bootstrap work; automatic Git
tracking defaults cannot override the env's intended tracking. Repository sequences and allocation lifetime remain
recoverable after process failure. Serves `product:environment-worktrees` and `product:environment-recovery`.

## Safety probes cannot degrade to empty state (engineering:teardown-evidence-completeness)

Destructive preflight distinguishes absent data from failed inspection and retains evidence for every store being
removed. Read-only preview cannot run teardown commands or repair registration implicitly. Serves
`product:destroy-environments` and `product:environment-state-recovery`.
