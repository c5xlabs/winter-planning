# Repository catalog engineering requirements

For `epic:parallel-workspaces/repository-catalog`.

## One identity model for every input path (engineering:catalog-identity-consistency)

Hand-authored layers, catalog edits, and materialization consumers use the same name, role, and location validation;
normalized path and source aliases cannot bypass collision checks. Serves `product:repository-roles`,
`product:repository-addresses`, and `product:edit-repository-inventory`.

## Containment survives filesystem changes (engineering:catalog-path-confinement)

Managed-path authorization accounts for resolved filesystem ancestry at the point of effects, not just lexical absence
of `..`; a changed symlink or path replacement cannot redirect a previously validated catalog operation. Serves
`product:repository-addresses`.

## Catalog interpretation stays inert (engineering:catalog-effect-separation)

Catalog reads and edits have no implicit clone, hook, provisioning, or extension-entrypoint dependency; consumers
receive declared roles without discovering executable behavior as a side effect. Serves `product:inventory-presence` and
`product:edit-repository-inventory`.
