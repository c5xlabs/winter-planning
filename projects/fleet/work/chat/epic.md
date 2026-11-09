---
epic: chat
refinement: outline
status: planned
---

# Chat channel

A standalone epic; its requirements are inline below.

## The problem

The fleet's best questions arrive at the worst times: overnight, over a weekend, while `persona:product-owner` is
anywhere but at a browser.

## The outcome

A chat bot is the channel that is already in their pocket. This epic gives the fleet one: question notifications that
arrive as messages, answers that cost one tap, and a confirmation that the agent picked the answer up and went back to
work.

Answering from afar and confirming that the agent resumed were delivered by `epic:ask-answer/remote`. Carrying a
question *out* to a channel, and deciding who subscribes to what, lands here, proven by the bot that first consumes it.

## Requirements

### Questions as chat messages (product:question-messages)

A question the fleet raises reaches the people subscribed to it as a Telegram message, the first chat platform
(`decision:chat/first-platform`). Who is told about which questions, and when, is settled by
`decision:chat/subscription-model`.

Acceptance:

- A subscribed person receives a question as a Telegram message without opening the board.

### One-tap answers (product:one-tap-answer)

A question arrives carrying its options as buttons in the message, and tapping one is the whole act of answering. A
free-text question accepts a typed reply. The first answer wins, as it does from every other client, and anyone beaten
to it is told who got there first.

Acceptance:

- Tapping an option in the message answers the question.
- A typed reply answers a free-text question.
- A person whose answer arrived second is told who answered first.

### Delivery confirmation (product:delivery-confirmation)

The message thread closes its own loop: *delivered, agent resumed* lands in the chat, so the person who answered from a
parking lot knows the fleet is moving again.

Acceptance:

- After an answer is delivered and the agent resumes, the chat thread says so.

## Open questions

- **Product owner:** Whether a Telegram account that maps to no hub user may answer at all, once the board's auth lands.
