# The application architect (persona:application-architect)

The agentic-native engineer who hands real work to agents and expects it to land.

## Who they are

They can envision ambitious solutions and reason through the code that would make them work. Their ideas have
traditionally outpaced how quickly they could write that code. They excel at managing complexity across a codebase,
shaping boundaries and relationships so that humans and agents can work without repeatedly untangling the whole system.
They can turn that understanding into written designs and work with models on detailed plans and feature specs.

They are far enough into agentic development that they more or less trust the output — not blindly, but because they
trust themselves to recover it: when a run goes astray, they can drop into the exact session, see what the agent saw,
and fix or redirect it. Trust, for this person, is not faith in the model; it is confidence in their own reach.

## What their work looks like

Their unit of work spans repositories, and they rarely run one agent at a time. During the day they set several agents
going in parallel, each on its own feature, and expect none of them to trip over another's checkout, ports, or services.
When they step away, they want work to keep moving without them and to come back to changes that landed, or to a problem
stated precisely enough to act on at once. What they never expect to find is a wedged system or a mystery.

## What they need from the suite

They need to install the suite once and add only the products they want, each useful on its own. They need a system that
spans many repositories treated as normal at every layer, from the environment an agent works in to the delivery that
lands its change. They need parallel agents that cannot collide, and status they can believe at a glance. And when
something goes wrong, they need to land in the stuck agent's full session in one step, because recovering a run is what
their trust rests on.

## What serves them

The platform's one-step install and per-workspace versions; Workspace's isolated, fully running feature environments and
its commands that act across every repository at once; and Fleet's unattended execution, takeover, and board. What they
need from Fleet in particular is set out in [the Fleet vision](../../projects/fleet/charter/vision.md).
