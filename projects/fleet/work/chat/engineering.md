# Chat channel engineering

Engineering for `epic:chat`.

## What already holds

- `epic:ask-answer/remote` delivered the answer-back route and the closed-loop confirmations the bot consumes.

## Brief requirements

- engineering:fan-out-seam — This epic owns the notification fan-out seam and builds it with the Telegram bot as its
  first binding. Serves `product:question-messages`.
- engineering:chat-adapter-seam — The chat layer is a seam, so a second chat platform is an adapter, not a rewrite.
  Serves `product:question-messages`.
- engineering:shared-answer-route — The bot submits answers through the same hub route as every other client, with the
  same first-write-wins arbitration. Serves `product:one-tap-answer`.
- engineering:no-lost-questions — A bot failure loses no question; the question row is durable wherever the bot runs.
  Serves `product:question-messages`.

## Open questions

- **Engineering:** How a Telegram account maps to a hub user once the board's auth lands.
