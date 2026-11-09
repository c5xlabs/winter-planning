# Live preview engineering

Engineering for `epic:preview`.

## What already holds

- The hub never reaches into a runner, which is what lets a runner sit on a laptop behind NAT.

## Brief requirements

- engineering:runner-opened-tunnel — Preview traffic rides a tunnel the runner opens and the hub multiplexes, so the hub
  never connects into a runner (`decision:preview/reachability`). Serves `product:open-preview`.
- engineering:workspace-serving-address — An optional workspace-provider capability names the application address a
  workspace hosts; a provider that hosts nothing declines. Serves `product:open-preview`.

## Open questions

- **Engineering:** The tunnel's transport and how a session is authorized, since an opened tunnel is a hole in the
  boundary the design exists to protect.
