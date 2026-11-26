# Scope environment decisions

## Inherited environment (decision:running-environments/scope-environment/inherited-environment)

**Decider:** product owner and engineering

A launched workload or lookup does not take Winter identity from the caller's environment: Winter removes the managed
`WINTER_*` names it inherited before applying the scope's values, while `winter env` output stays export lines only, as
in the reference. The reference passes the caller's environment through wholesale, so a shell that sourced one
environment leaked its stale `WINTER_*` values into another scope's launch; this is a registered deviation that fixes
that leak.

## Incomplete launch refusal (decision:running-environments/scope-environment/incomplete-launch-refusal)

**Decider:** product owner and engineering

A template error or refused shell entry stops the affected launch, every scope consumer refuses an unregistered name
instead of deriving an index from it, and a recorded index outside the declared band is reported. The reference warned
and launched with no scope values, gave provisioning, hooks, and nested delegation an unprobed fallback index, and never
checked the recorded index; these are registered deviations because each hid a wrongly configured workload.

## Persistent value exposure (decision:running-environments/scope-environment/persistent-value-exposure)

**Decider:** product owner and engineering

Winter writes no resolved value to disk; resolved values reach only `--resolve` stdout and launched workloads'
environments, as in the reference, and a failing lookup's stderr is shown in its error and disclosed as such. As a
registered deviation, a value substituted into another entry is inserted literally and never re-parsed, and no error
prints a resolved value, because the reference re-parsed a substituted `${` and printed the secret fragment in its
error.

## Lookup execution policy (decision:running-environments/scope-environment/lookup-execution-policy)

**Decider:** product owner and engineering

Lookups follow the reference: computed once per scope per `up` or `start` fan-out with no cache across scopes or runs,
never run by inspection, status, or stop, each with a fixed 30-second deadline, closed stdin, and the workspace root as
working directory. As a registered deviation, a timed-out lookup's whole process group is killed, because the reference
killed only the direct child and left shell-started processes orphaned; an interrupted lookup's group is killed the same
way, and the invocation still ends as the reference's interrupt does, aborted with exit 1.

## Resolution compatibility (decision:running-environments/scope-environment/resolution-compatibility)

**Decider:** product owner and engineering

Resolution follows the reference: plain values may redeclare `WINTER_*` names, colliding keys resolve by band rank then
declaration order, an intermediate read before a later import is allowed, references to masked multi-value imports are
undefined-variable errors, and dotenv imports keep the reference's parsing. Only the reference's Python artifacts are
not copied: an offset applies to a value that is a decimal integer (optional sign, ASCII digits, no whitespace or
underscores, within a 64-bit signed integer), and JSON scalars import in their standard JSON spelling (`true`, `false`,
`null`, and a number as the text the program printed). As registered deviations, a declared key that is not a valid
shell identifier is refused at load, as imported keys already are, because the reference emitted an `export` line that
broke providers' `eval`; and a non-shell command entry is split into words before references are substituted, so a
substituted value fills its own word, because the reference substituted first and let a value add or remove arguments.
