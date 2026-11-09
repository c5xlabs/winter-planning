# Bench decisions

## Bench sessions (decision:lab/bench/bench-sessions)

**Open.** How a bench shares sessions. A bench node that resumes a session begun upstream inherits a conversation the
seed cannot reproduce. Whether the lab starts every bench session fresh, replays the upstream transcript, or refuses
such a bench decides how faithful a trial is to the graph it came from. It blocks `product:mint-refusals`.
