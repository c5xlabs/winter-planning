---
name: pull-advance
depends-on:
  - resolve-and-materialize
---

# Advancing or holding a pin during a pull

Builds `PullMode`, `Integration`, `IntegrateCheckout`, `Advanced`, and the Advance a pin on pull flow of
[standalone-pins.md](../spec/standalone-pins.md). Delivers the held-reference bullet of
`product:standalone-reference-policy`, under
`decision:parallel-workspaces/reproducible-inputs/resolution-and-locks`. The standalone integration of
`epic:parallel-workspaces/coordinated-git` implements `IntegrateCheckout` and calls `advance`; nothing does here.

## Change

- A tag or commit pin reports `held_pin` with the declared ref in every mode and touches nothing.
- A branch pin is guarded, stashed when dirty and `autostash` is set, integrated onto `origin/<ref>` in the given mode,
  and popped; a fast-forward upserts the new HEAD and reports `re_pinned`, and any other integration is returned with
  the lock unchanged.
- The types and `advance` are re-exported by name from `lib.rs` for their component tests.

## Acceptance

- Component tests with a recording `IntegrateCheckout`: a tag pin reports `held_pin` and the integration is not called;
  a branch pin is integrated onto `origin/<ref>` in the mode given; a fast-forward rewrites the entry to the new HEAD
  and reports `re_pinned` with `commits` 0; `Merged`, `Rebased`, `UpToDate`, and `Diverged` are returned with the lock's
  bytes unchanged.
- Component tests: a dirty branch pin without `autostash` is `pin_error` and the integration is not called; with it
  the stash is popped after a fast-forward and after a diverged integration; a failed pop is logged naming the checkout
  and leaves the outcome.
- `cargo dev gate slice` exits 0.
