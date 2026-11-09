# Worker profiles decisions

## Overriding one field (decision:worker-profiles/field-override)

**Open.** Whether a node may name a profile and override a single field, which is obviously convenient and quietly
reintroduces the restatement this epic exists to remove; it blocks `product:node-names-profile`.

## Where profiles live (decision:worker-profiles/profile-home)

**Open.** Whether profiles live beside the graph that uses them, or in hub configuration where an operator can change
them without touching a graph; it blocks `product:named-profile`.
