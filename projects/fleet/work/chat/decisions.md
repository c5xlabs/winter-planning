# Chat channel decisions

## Subscription model (decision:chat/subscription-model)

**Open.** Who is told about which questions, and when: everyone about everything is noise, and a routing scheme
(per-fleet, per-graph, per-persona) is design work this epic must settle before the first channel is worth having; it
blocks `product:question-messages`.

## Where the bot runs (decision:chat/bot-placement)

**Open.** Whether the bot runs inside the hub daemon or beside it, and what its failure does to the fan-out contract; it
blocks `engineering:fan-out-seam`.

## First chat platform (decision:chat/first-platform)

Telegram is the first binding, chosen for having the best inline-answer-button experience of the candidates.
