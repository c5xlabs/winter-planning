---
epic: provider-kit
refinement: outline
status: planned
---

# Provider kit

A standalone epic; its requirements are inline below.

## The problem

Blizzard's seams are real, and every binding behind them ships inside the wheel, so someone who wants their own work
source writes a pull request against blizzard and waits. Criterion 5 is proven at two live bindings, and blizzard cannot
honestly claim the seam is a seam until one of them comes from outside.

## The outcome

A binding is something a stranger can write and ship: an interface they can read, discovery that finds what they
installed, and a suite that tells them they got it right before a chunk depends on it. Work sources and workspaces come
first, the two seams whose next occupants are likeliest to come from outside.

## Left for later

- The human channel joins once `epic:chat` builds its fan-out seam.

## What this epic is not

Delivery and the coding harness stay in-tree for now, carrying the fencing and execution guarantees.

## Requirements

### A declared interface (product:binding-interface)

What a binding must provide and what it may rely on is written for someone who does not read blizzard's internals, which
is the test of whether the seam was ever really one.

Acceptance:

- An author who has not read blizzard's source writes a binding from the interface alone.

### Discovery without a code change (product:binding-discovery)

An installed binding is found and usable without editing blizzard, which is the difference between a plugin and a patch.

Acceptance:

- Installing a binding makes it usable with no change to blizzard.

### A conformance suite (product:conformance-suite)

A binding proves itself before it runs a single chunk, so the author learns about a misunderstanding from a test rather
than from a wedged night.

Acceptance:

- An author runs the suite against their binding before any chunk uses it.

### One binding out of the tree (product:out-of-tree-binding)

One existing binding moves out of the tree, to prove the path is walkable by someone who is not us.

Acceptance:

- One binding that shipped inside blizzard is installed from its own package and works as it did before.
- It passes the conformance suite from outside the tree.

## Open questions

- **Engineering:** The discovery mechanism, and how much of the host language's packaging conventions blizzard should
  adopt rather than invent.
- **Engineering:** What the suite can genuinely assert versus what it can only smoke-test, since a work source that lies
  about its items will pass anything cheap.
- **Engineering:** How the interface is versioned, and what an older binding does when it meets a newer hub.
