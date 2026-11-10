# Test suite epic order

What each of the test-suite initiative's epics waits on, behind `decision:test-suite/epic-order`.

- The foundation lands first and the hub tier follows it.
- The runner tier comes after them because it needs real product work: a host whose loop can be stepped from outside,
  and reads of the state its tests check through the ORM. The fleet tier follows on that host.
- The journey follows the fleet tier.
- The controlled clock needs only what the hub and runner tiers already built, the stepped sweeps and the stepped host,
  so it may start beside the fleet tier and the journey rather than after them. The journey may adopt it once both have
  landed.
- Optimization comes last. Its work speeds the tiers that stay in Python and the pipeline around them, but the gate it
  measures is the one the tier epics leave behind, so its spec stays provisional until they land.
