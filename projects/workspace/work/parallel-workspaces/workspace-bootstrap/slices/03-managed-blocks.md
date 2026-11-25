---
name: managed-blocks
depends-on:
  - module-scaffold
---

# Managed block text functions

Adds the pure text functions of [managed-content.md](../spec/managed-content.md), Domain and Interfaces, Managed block
text: `Markers` for exclude and Markdown files, `replace_or_insert` at either position, `strip`, `blocks`, and `shape`.
Delivers the text half of `product:managed-content-ownership`: content outside a block is kept byte for byte.

## Change

- `crates/core/src/managed_block.rs` holds the functions, re-exported from `lib.rs`; no command calls them yet.

## Acceptance

- Component tests of `replace_or_insert`: an existing block is replaced in place and the lines around it are unchanged;
  an absent block is appended after one blank line, or prepended before one, with no blank line added beside an existing
  blank one; a begin without its end is replaced to end of file; the result always ends with "\n".
- Component tests of `strip`: the block and the one blank line above it go; content without the begin marker is returned
  unchanged.
- Component tests of `blocks` and `shape`: blocks are listed in file order with their lines; a begin without end, or two
  begins, is `Malformed`.
- `cargo dev gate slice` exits 0.
