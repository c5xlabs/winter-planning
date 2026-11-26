# Status

The `ws status` command: one read-only snapshot of every environment's worktrees, the project source checkouts, the
standalones, and workspace-level drift, with an exit code a script can gate on. Its options are the `ws status` row of
[command-surface.md](../../../../delivered/parallel-workspaces/component-runtime/spec/command-surface.md); its document
is the shape A `ws status` document of
[output-formats.md](../../../../delivered/parallel-workspaces/component-runtime/spec/output-formats.md), which this leaf
owns, and its exit codes extend the `ws status` block of
[results-and-exit-codes.md](../../../../delivered/parallel-workspaces/component-runtime/spec/results-and-exit-codes.md),
Exit codes. Selection is [target-patterns.md](./target-patterns.md) with `EnvRule::Present`; the probes run as
[fan-out.md](./fan-out.md) `STATUS_PROBES`; `--fetch` is `ws fetch` of [refresh.md](./refresh.md). The worktree probe
extends that of
[location-listing.md](../../../../delivered/parallel-workspaces/repository-catalog/spec/location-listing.md); drift is
`drift::detect` of
[drift-warning.md](../../../../delivered/parallel-workspaces/repository-catalog/spec/drift-warning.md), with no notice;
orphans are `scan` of [prune.md](../../workspace-bootstrap/spec/prune.md); pins are `snapshot` of
[standalone-pins.md](../../../../delivered/parallel-workspaces/reproducible-inputs/spec/standalone-pins.md); the
`nested` member is [child-state.md](../../nested-workspaces/spec/child-state.md)'s. Reference locations are paths under
`tools/winter-cli/src/winter_cli/` of `winter-workspace` at `127cdf2`; "WSS" is
`modules/workspace/workspace_snapshot_service.py`, "RRR" `modules/workspace/internal/read_repo_repository.py`, "WH"
`modules/workspace/handlers/workspace_handler.py`. Message wording is the binary's own (`decision:functional-parity`).

## Domain

```rust
// crates/core/src/status.rs — product:inspect-workspace-git, product:worktree-upstream-control
pub(crate) struct WorktreeRow {
    pub repo: String, pub branch: Option<String>, pub upstream: Option<String>,
    pub ahead: u64, pub behind: u64,                         // against origin/<main_branch>
    pub tracking_ahead: u64, pub tracking_behind: u64, pub tracking_ref_present: bool,
    pub staged: u64, pub unstaged: u64, pub untracked: u64, pub dirty: u64,
    pub last_commit_subject: Option<String>, pub pinned: bool, pub main_branch: String,
    pub nested: Option<NestedMember>,                        // child-state.md
}
pub(crate) struct EnvRow { pub name: String, pub index: i64, pub port_base: i64,
    pub feature_branch: Option<String>, pub worktrees: Vec<WorktreeRow> }   // extensions: always {}
pub(crate) struct CheckoutRow { pub repo: String, pub branch: Option<String>, pub behind_origin: u64,
    pub ahead_origin: u64, pub dirty: u64, pub local_only_commits: u64, pub stashes: u64 }
pub(crate) struct ProjectRow { pub checkout: CheckoutRow, pub drift: Vec<String> }
pub(crate) struct Snapshot {
    pub environments: Vec<EnvRow>, pub projects: Vec<ProjectRow>, pub standalones: Vec<CheckoutRow>,
    pub workspace: WorkspaceLevel, pub dashboard: (DashboardLayout, DashboardLayout),
    pub unseen: Unseen,                                      // what the document cannot show: for the exit code
}
pub(crate) struct Unseen { pub missing_worktrees: usize, pub missing_standalones: usize, pub failed_standalones: usize,
    pub failed_refreshes: usize }

pub(crate) fn collect(ports: &Ports<'_>, diagnostics: &mut Diagnostics<'_>, start: &Path, patterns: &[Pattern])
    -> Result<Snapshot, StatusCollectionError>;
pub(crate) fn exit_code(snapshot: &Snapshot, scoped: bool) -> u8;

// termination.rs — Termination gains StatusCollection(Box<StatusCollectionError>), exit 2, as
// results-and-exit-codes.md, Domain, defines it; GitOperation gains LocalWorkProbe ("local-work probe"),
// TrackingAheadBehindProbe ("tracking ahead/behind probe")
```

## Interfaces

### Order

```text
# reference: WH:272-312, WSS:88-357
1. root discovery and the load; a failure is its ConfigError, exit 1
2. --fetch: ws fetch of refresh.md with Scope::Project and the same PATTERNS, its events in their mode (under
   --json, shape C events, then the document); its failures count in Unseen.failed_refreshes
3. collect; any failure is the StatusCollectionError, exit 2, and nothing more is written
4. the document, or the text, written whole; then exit_code
nothing is fetched without --fetch; nothing is written to any repository or file
```

### Environments

```text
# reference: WSS:118-217, modules/workspace/env_status_service.py:81-140, RRR:64-145,232-252,506-619,780-801
selection   every environment present, each with a row per loaded project entry; PATTERNS filter the rows
            (target-patterns.md: an unknown literal env is the collection error); with PATTERNS, an environment
            left with no row is dropped; matched rows of all environments zero, missing ones included ->
            the collection error "no worktrees match: <patterns joined by ', '>"
probe       every worktree of every environment present is probed, matched or not, as the reference does: the
            env's feature_branch reads them all. The probes of one environment run at most STATUS_PROBES at once;
            environments one after another; the first failure in environment, then project, order is the
            collection error, after every probe of that environment ran
missing     an absent or non-repository worktree is not probed: it is warned as target-patterns.md gives, it has
            no row, and a matched one counts in Unseen.missing_worktrees
            # registered deviation, decision:parallel-workspaces/target-selection: the reference shows it as a clean
            # row of zeros; the v1 schema admits no extra field, so it is left out instead of marked
worktree probe, at the worktree, cwd = its path
  1. git status --porcelain=v2 --branch --untracked-files=all -z      non-zero -> RepoError "status probe failed"
     branch        the branch.head header; "(detached)" -> null
     upstream      the branch.upstream header, or null; present once tracking is configured, fetched or not
     tracking_*    the branch.ab header "+<a> -<b>": tracking_ahead a, tracking_behind b, tracking_ref_present
                   true; no branch.ab header -> 0, 0, false
     staged        records of type 1 or 2 whose X is not ".", plus every type u record
     unstaged      records of type 1 or 2 whose Y is not ".", plus every type u record
     untracked     "?" records
     dirty         distinct paths of staged and unstaged, plus untracked; a type 2 record counts by its new path
  2. ahead, behind: main_branch empty -> 0, 0; else git rev-list --left-right --count origin/<main>...HEAD, the
     left count behind and the right ahead; a failure when origin/<main> does not resolve -> 0, 0; otherwise
     RepoError "main-branch ahead/behind probe failed"
  3. last_commit_subject: ahead 0 -> null; else git log -1 --format=%s HEAD, trimmed; empty or failed -> null
feature_branch   the first non-pinned worktree of the environment, in project order, whose upstream starts
                 "origin/": that upstream less "origin/"; null when none (missing worktrees read as none)
index, port_base the environment's index (index-lookup.md's lookup); base_port + index * ports_per_env
index-lookup.md = ../../environment-lifecycle/spec/index-lookup.md
```

### Projects, standalones, workspace

```text
# reference: WSS:219-357,483-621, RRR:272-356,583-598, modules/workspace/env_status_service.py:142-189
projects     each loaded project entry, in merged order, probed at <root>/projects/<repo>: the worktree probe's
             steps 1 and 2, then local work:
               git rev-list --count --branches --not --remotes            -> local_only_commits
               git rev-parse --verify --quiet refs/stash exit 0 -> the line count of git stash list, else 0
               a failure -> RepoError "local-work probe failed"
             an absent or non-repository checkout reads all zeros, as the reference reads it
             a probe failure is the collection error
             row when ahead, behind, dirty, local_only_commits, or stashes is non-zero, or drift's missing names
             it (drift ["missing from projects/"], zeros, branch null); then a row per undeclared name, drift
             ["undeclared in config"], zeros
standalones  each declared standalone, in merged order, at its location:
               branch         git symbolic-ref --quiet --short HEAD, null on exit 1
               dirty          distinct staged and unstaged paths plus untracked files, as location-listing.md's
                              workspace probe, steps 2-5, counts them
               ahead, behind  against the tracking operand of its active branch (refresh.md, no commit check):
                              git rev-list --count <T>..HEAD and HEAD..<T>; no T -> 0, 0
               local work     as projects
             a failure of any of these -> a warning naming the standalone and the cause, no row, and
             Unseen.failed_standalones; the other standalones and the command go on
             absent or no repository -> warned as target-patterns.md gives, no row, Unseen.missing_standalones
             # registered deviation, target-selection: the reference writes a row of zeros for it
workspace    root_path      the workspace root, as discovered
             extensions     the resolved names of the standalones that are extensions, in merged order; config only
             orphans        prune.md's scan, read-only: {"kind", "path", "safe_to_remove", "notes"} per orphan, in
                            scan order; safe_to_remove is retained being None
             drift_missing, drift_undeclared   drift::detect's lists
             standalone_pins   pins::snapshot, each {"name", "ref", "kind", "locked_commit", "config_ref_drift",
                            "head_drift", "head_commit"}
dashboard    configured_layout: the loaded dashboard_layout; resolved_layout: itself unless "auto"; "auto" ->
             "repos-as-rows" with no environment present, else "list" when there is one project entry, else
             "repos-as-rows" when project entries outnumber the environments present, else "repos-as-columns"
             (config/models.py:147-176), counted before PATTERNS filter anything
```

### Document

```text
# reference: WH:1418-1530, schemas/ws-status-v1.json
{"schema_version": 1,
 "environments": [{"name", "index", "port_base", "feature_branch", "worktrees": [row...], "extensions": {}}],
 "projects":    [{"repo", "branch", "behind_origin", "ahead_origin", "dirty", "drift", "local_only_commits", "stashes"}],
 "standalones": [{"repo", "branch", "behind_origin", "ahead_origin", "dirty", "local_only_commits", "stashes"}],
 "workspace":   {"root_path", "extensions", "orphans", "drift_missing", "drift_undeclared", "standalone_pins"},
 "dashboard":   {"configured_layout", "resolved_layout"}}
row: {"repo", "branch", "upstream", "ahead", "behind", "tracking_ahead", "tracking_behind", "tracking_ref_present",
      "staged", "unstaged", "untracked", "dirty", "last_commit_subject", "pinned", "main_branch"} plus "nested" on
      the rows child-state.md names
every document validates against schemas/ws-status-v1.json, which the binary does not change; extensions is always
the empty object, as in-process plugin decorators are gone
```

### Exit code

```text
# extends results-and-exit-codes.md's ws status block; reference: WH:1363-1416
2   the collection error, as that block gives it, an unknown literal environment included
1   that block's rules, and also:
      with or without PATTERNS: a matched worktree that is missing (Unseen.missing_worktrees), or a failed
      refresh under --fetch (Unseen.failed_refreshes)
      without PATTERNS: a missing standalone, or one whose probe failed
    # registered deviations, decision:parallel-workspaces/target-selection and
    # decision:parallel-workspaces/coordinated-git/fetch-freshness: none of these flips the reference's code
0   otherwise
```

### Text

```text
# reference: WH:1212-1361; layout is not contract
per environment   "Env: <name>", then "Branch:   <feature_branch>" when set, then a table REPO | SYNC | DIRTY:
                  SYNC "+<ahead>, -<behind>" (each only when non-zero), DIRTY "<n> file(s)"; "  No repos" when
                  it has none; then child-state.md's nested lines
Projects:         REPO | BRANCH | SYNC | DRIFT, SYNC adding "<n> local-only" and "<n> stash(es)"
Standalones:      REPO | BRANCH | SYNC | DIRTY
Workspace:        only with drift or orphans: "missing: ...", "undeclared: ...", "orphans: <n>" with one
                  "<'  ' or '! '><kind>  <path>" line each
```
