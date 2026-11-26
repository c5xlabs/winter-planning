# Child state

What a nested workspace holds, read from its own `ws status --json`, and how the outer `ws status` reports it. The read
is the status read of [invocation.md](./invocation.md); the loss audit of [teardown.md](./teardown.md) uses the same
`NestedState`. The outer `ws status` document, its options, and every other field are
`epic:parallel-workspaces/coordinated-git`'s; this leaf owns the `nested` member of a worktree row and the text line it
adds; the exit rule it feeds is that of
[results-and-exit-codes.md](../../../../delivered/parallel-workspaces/component-runtime/spec/results-and-exit-codes.md),
`ws status`. Reference locations are paths under `tools/winter-cli/src/winter_cli/` of `winter-workspace` at `127cdf2`.
Message wording is the binary's own (`decision:functional-parity`).

## Domain

```rust
// crates/core/src/nested/state.rs — product:nested-work-inspection, engineering:nested-loss-evidence
// reference: modules/workspace/models/domain_model.py:73-118, modules/workspace/nested_status.py
pub(crate) struct NestedState {
    pub envs: Vec<NestedEnv>,              // the child's environments, in its document's order
    pub checkouts: Vec<String>,            // child source checkouts and standalones holding local work, labelled
}
pub(crate) struct NestedEnv {
    pub name: String,
    pub dirty: Vec<String>,                // repo names
    pub unpushed: Vec<String>,             // "<repo> (<reason>, ...)"
}
impl NestedState {
    pub fn env_count(&self) -> usize;                  // envs.len()
    pub fn dirty(&self) -> bool;                       // any env with a dirty repo
    pub fn unpushed(&self) -> Vec<String>;             // "<env>/<entry>" for every env entry, then checkouts
}
pub(crate) fn parse(document: &serde_json::Value) -> Result<NestedState, ShapeFault>;
pub(crate) fn reported_root(document: &serde_json::Value) -> Result<Option<&str>, ShapeFault>;
pub(crate) struct ShapeFault(pub String);          // "<field path> is <value>; expected <kind>"
```

## Interfaces

### Shape rules

```text
# decision:parallel-workspaces/nested-workspaces/child-compatibility
# reference: modules/workspace/nested_status.py:17-151
reported_root   the document is an object; schema_version is the integer 1 (else "has schema_version <v>; expected 1");
                workspace.root_path when workspace is an object and root_path a string, else None
kinds           object; array; string; count = a non-negative integer, not a boolean; flag = a boolean;
                string-or-null
read, each a ShapeFault when its kind does not hold:
  environments                              array
  environments[i]                           object; .name string; .worktrees array
  environments[i].worktrees[j]              object; .repo string; .dirty count; .ahead count; .tracking_ahead count;
                                            .upstream string-or-null; .tracking_ref_present flag, read only when
                                            .upstream is a string
  ...worktrees[j].nested, when present      object; .error string-or-null; .dirty flag; .unpushed flag
  projects, standalones                     array
  projects[i], standalones[i]               object; .repo string; .dirty count; .ahead_origin count;
                                            .local_only_commits count, when present; .stashes count, when present
fields not named are ignored, so a newer child that adds fields still reads
```

### Derivation

```text
# product:nested-work-inspection, engineering:nested-loss-evidence
env worktree w of env e
  dirty      w.dirty > 0, or w.nested present with nested.dirty true or nested.error a string   -> e.dirty += w.repo
  reason     w.tracking_ahead > 0                                         "unpushed commits"
             else w.ahead > 0 and (w.upstream null or not w.tracking_ref_present)   "unpushed branch"
             # a configured upstream whose tracking ref is missing holds nothing remotely
             w.nested.unpushed true                                       + "unpushed work in its nested workspace"
             any reason -> e.unpushed += "<w.repo> (<reasons joined ', '>)"
             # a branch pushed but not merged is ahead with its upstream present: no reason
checkout c, label "projects/<repo>" for a project, "standalone <repo>" for a standalone
  reasons    c.dirty > 0 "uncommitted changes"; c.ahead_origin > 0 "ahead of origin";
             c.local_only_commits > 0 "local-only commits"; c.stashes > 0 "stashes";
             c.local_only_commits absent "local-only commits unknown"; c.stashes absent "stashes unknown"
             any reason -> checkouts += "<label> (<reasons joined ', '>)"
# registered deviation (decision:parallel-workspaces/nested-workspaces/child-compatibility, decision:rough-parity):
# the reference reads an absent local_only_commits or stashes as 0, so such a checkout can read as holding nothing
```

### The `nested` member of `ws status`

```text
# product:nested-work-inspection; reference: modules/workspace/workspace_snapshot_service.py:133-217,482-502,
#   modules/workspace/handlers/workspace_handler.py:1198-1263,1414-1415,1462-1468
rows        each worktree row of a project with `nested` declared whose worktree exists on disk; no other row has it
read        Nesting::state(worktree path), after the env and worktree probes; the reads of one invocation run as the
            status probes of ../../coordinated-git/spec/fan-out.md, at most 8 at once; each result goes into its own
            row, so the document's order does not depend on completion order
member      {"env_count": <integer | null>, "dirty": <bool>, "unpushed": <bool>, "error": <string | null>}
  read      env_count = env_count(), dirty = dirty(), unpushed = unpushed() is not empty, error null
  RepoError env_count null, dirty false, unpushed false, error = the rendered RepoError; never a collection error,
            so one unreadable child does not stop the other rows or the command
text        after the env's table, one line per row with the member, in row order:
              "  <repo> (nested): <n> env" or "<n> envs" (n == 1 singular), then ", dirty" when dirty,
              then ", unpushed" when unpushed
              "  <repo> (nested): unreadable — <first line of error>"
exit        a row whose member has dirty true or a non-null error counts as a dirty worktree, scoped or not
            (results-and-exit-codes.md, `ws status`); unpushed alone does not change the exit code
```
