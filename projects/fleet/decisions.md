# Fleet decisions

## Where mobile investment goes (decision:mobile-path)

**Open.** Whether blizzard invests in the progressive-web-app shell or in native applications (`epic:android`,
`epic:ios`) as its answer to the operator away from a browser; it blocks `epic:pwa`.

## Where each scope of settings lives on the CLI (decision:settings-cli-scopes)

**Decider:** owner

Each scope of settings has its own group, named for who controls it, and all three read and write a key the same way:
`blizzard hub admin settings` holds the hub-wide settings a hub administrator controls, `blizzard hub settings` a
tenant's, controlled by its admins, and `blizzard runner settings` a runner host's, with no tenant layer. The hub-wide
settings sit under `hub admin`, beside the hub administrator's other verbs and their `/api/admin` routes, rather than
inside the tenant's group, so a tenant's group never reaches past its tenant.
