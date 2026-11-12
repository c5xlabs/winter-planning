# Sentinel agents decisions

## The scopes an agent may declare (decision:sentinel-agents/scopes)

**Open.** Which scopes an agent may be singular within, starting from one per tenant and one per project, and whether
any finer scope is worth having. **Decider:** product owner.

## Where the conversation lives (decision:sentinel-agents/conversation-home)

**Open.** The hub holds every turn (`product:every-turn-recorded`), but resuming happens in a harness on a runner, and a
harness resumes from its own session state, not from the hub's record. How a volunteer rebuilds a resumable
conversation from the turns the hub holds, whether a runner that still holds the live session may resume it directly,
and what the factory's compaction or clearing does to the turns it keeps. **Decider:** engineering.

## When a turn is given up (decision:sentinel-agents/turn-timeout)

**Open.** How long a granted turn may run before the hub takes it back, what heartbeat keeps a long turn alive, and what
happens to the output of a turn that arrives after its grant was withdrawn. **Decider:** engineering.
