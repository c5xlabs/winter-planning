---
name: wire-launcher-dispatch
depends-on:
  - prepare-cached-dispatch
  - prepare-legacy-launch
  - classify-native-routes
code:
  - winter/crates/platform/src/cli.rs::parse
  - winter/crates/platform/src/cli.rs::control_reply
  - winter/crates/cli/src/main.rs::main
---

# Wire the complete modern and legacy dispatch surface

Connect classification, modern dispatch, and legacy setup through [System](../spec.md#system),
[Interfaces](../spec.md#interfaces), and [Flows](../spec.md#flows). Deliver the public entry points for
`product:reach-component`, `product:respect-component-version`, `product:legacy-workspace-dispatch`,
`product:legacy-init`, `product:launcher-verbosity`, and `product:explain-dispatch-failure`, with
`engineering:legacy-adapter-boundary` and `engineering:fixture-only-dispatch`. `decision:singleton-launcher-succession`
adds compatibility to the foundation executable; it does not replace an installation or adopt Rust Workspace.

## Change

- Activate the classifier and advertise modern groups, opt-in legacy forwarding, Python-only initial source overrides,
  temporary init, and root verbosity in platform help only now that both dispatch implementations exist.
- In main, capture the launch instant at entry, then obtain cwd and one native inherited environment snapshot only for
  the selected dispatch route. Inject the build-time host target.
- Resolve absolute XDG config/cache directories with the documented HOME fallbacks for modern dispatch only; ignore
  relative XDG values and keep legacy free of those prerequisites.
- Route modern groups exclusively to cache dispatch, legacy/init exclusively to injected preparation, and foundation
  controls to configuration-independent replies. Preserve the existing sample gate.
- Keep reply writes/normal exit in main, launcher diagnostics on stderr, and successful dispatch as same-PID
  replacement.

## Acceptance

- Built production CLI service tests launch each cached stand-in independently and launch enabled Python stand-ins with
  ordinary, opaque-prefix, source-override, and bootstrap spellings.
- Explicit Rust groups ignore the legacy flag and never fall back to Python on missing selections/components; disabled
  legacy commands never invoke Rust. Unknown commands and unsupported overrides exit 2 without dispatch effects.
- Bare invocation, verbosity-only invocation, both help/version aliases, and verbose inspection work with unusable cwd
  and no HOME/XDG/config/cache/source access. Lookup spies or isolated fixtures distinguish bypass from ignored errors.
- Modern XDG fixtures cover absolute, empty, relative, and HOME-fallback inputs; required capture failures exit 126.
  Legacy succeeds without modern XDG/HOME prerequisites.
- Native environment/cwd/timing inputs reach the selected adapter once per dispatch; inherited launch metadata cannot
  substitute for the launcher-entry timestamp.
- Root verbosity stays on stderr without contaminating structured component stdout; modern component verbosity remains
  after the group, and legacy verbosity stays in Python argv.
- Both build configurations retain foundation inspection/reply-write behavior and sample isolation. No installation,
  workspace adoption, migration, or silent legacy opt-in occurs.

## Out of scope

- `prove-dispatch-boundaries` adds the full process/concurrency acceptance matrix; no manual singleton replacement is
  performed by this slice.
