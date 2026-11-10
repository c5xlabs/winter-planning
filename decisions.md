# Suite decisions

## Context dogfooding (decision:context-dogfooding)

Every project uses the suite's own `winter-canon`-governed context structure for development and review rather than
separate, ad hoc setup and extension documents, so the suite exercises the conventions it supports. The canonical home
may be a dedicated `{name}-context` repository or the application repository's root-relative `./context`.

## Homebrew channel (decision:homebrew-channel)

The suite's initial Homebrew channel is the C5X Labs tap, with each released CLI installed through
`brew install c5xlabs/tap/<name>`, so distribution does not wait for Homebrew core inclusion. Formulae select prebuilt
release artifacts rather than compiling the tools on the user's machine.
