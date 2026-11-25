# Guidance

The Workspace topics this epic adds: one per command it adds, and the `setup` family that
`decision:agent-ready-operation/context-navigation/setup-guidance` assigns to it. Topic sources, references,
reachability, and the coverage tests are those of [context-navigation
spec.md](../../../agent-ready-operation/context-navigation/spec.md), Topic source and Coverage of commands; each topic
below is one Markdown file under `crates/core/topics/` of `winter-workspace`, at the path its id gives. A topic names
only commands and options the binary holds. Reference sources are paths of `winter-workspace` at `b57b020`; a topic
restates their procedure for this binary's commands and drops what this binary has no command for.

## Data

```text
# product:create-unforked-workspace ("a route to the selected component's guidance");
# product:framework-guidance of epic:agent-ready-operation/context-navigation
init                  "Create a workspace here"
  states              what init writes and refuses (creation.md); that it clones nothing and runs no git init;
                      what to run next
  references          setup, setup/walkthrough
ws/init               "Reconcile source checkouts and standalones"
  states              the targets and their order; the per-repository steps and their order; what a repeat does
                      and does not redo; the exit code (reconciliation.md)
  references          ws/prune
ws/prune              "Remove material no configuration declares"
  states              the four kinds; every retention reason; --dry-run; --force; --json needing one of them; that
                      retained material is deleted by hand (prune.md)
  references          ws/init
setup                 "Set up a workspace"
  states              the branch index: one line per setup leaf the build holds
  references          setup/walkthrough, setup/contributing, setup/project-setup
setup/walkthrough     "Walk through workspace setup"
  states              the re-runnable procedure, in order: create the workspace (init); declare each project
                      repository (repo add), one at a time, order mattering for listings; set the git identity in
                      .winter/config.local.toml under [git.user]; reconcile (ws init); optionally write the project
                      setup and contributing documents. Each step checks what exists and skips what is done
  references          init, ws/init, repo/add, setup/contributing, setup/project-setup
setup/contributing    "Write the workspace's contributing guide"
  states              the questions and output of context/project/contributing.md: commit conventions, pre-commit
                      checks, delivery method, post-delivery, multi-repository considerations
  references          setup/walkthrough
setup/project-setup   "Write the workspace's project setup guide"
  states              the questions and output of context/project/project-setup.md, and which needs go to
                      configuration instead: dependencies to a repository's cmd, generated files to git_excludes, the
                      rest to the document; provisioning and services only as far as the build holds them
  references          setup/walkthrough
root                  gains references to init and setup; the ws group topic gains ws/init and ws/prune
```

```text
sources
  setup/walkthrough    skills/setup/SKILL.md, steps 1-7 and 10; steps 8 and 9 (service orchestration, extension setup
                       steps) belong to epic:running-environments/service-orchestration and
                       epic:agent-ready-operation/extension-management, which extend this topic when they land
  setup/contributing   context/winter/setup/contributing.md
  setup/project-setup  context/winter/setup/project-setup.md, sections 1, 2, and 5; sections 3, 4, and 6 come with
                       epic:running-environments/resource-provisioning and epic:running-environments/service-orchestration
size                   the root stays within engineering:entry-size of epic:agent-ready-operation/context-navigation
```
