# Advanced mocking requirements

The requirements of `epic:advanced-mocking`.

## Fixtures are data (product:fixture-files)

A world is declared in YAML files, never in code, so it can be written, reviewed, and versioned like any other document.

Acceptance:

- A new world, or a change to one, needs no change to the mock-data CLI's code.
- A fixture written on the spot, outside the library, applies the same way a committed one does.
- A fixture that names a concept, a field, or a reference the fleet does not have is refused with the place it occurs,
  before anything is written.

## Fixtures compose (product:fixture-composition)

A fixture holds one kind of thing, and a world is built by applying several fixtures to the same env, one after another,
rather than by writing it whole.

Acceptance:

- Applying a shell, a graphs, a gardening, and a backlog fixture to one env gives a tenant with all four.
- Applying a directory of fixtures applies them in the order their kinds depend on, whatever their file names.

## A fixture holds one kind of thing (product:fixture-modules)

Each fixture says which kind of thing it holds, so a reader knows what it represents from its first lines.
The kinds are a tenant's shell (its projects, people, forges, sources, and repositories), a codebase (a repository's
files and history), a backlog (issues with their bodies, threads, and labels), graphs, gardening, runners (each runner
added to the tenant, with the subscriptions it declares and the brakes it holds), and activity (chunks and everything
that happens to them).

Acceptance:

- A fixture holding something its kind does not cover is refused, naming the kind.

## Copies of a fixture never collide (product:fixture-targets)

The same fixture can be applied to one env many times as separate copies, each told apart by a prefix
given when it is applied, so a fixture never needs editing to be reused.

Acceptance:

- One backlog fixture applied with two prefixes gives two copies, and neither apply touches the other.

## Runners online or offline (product:runner-presence)

A fixture says which of its runners are online, which are offline, and which never connected, and the
board shows each as declared.

Acceptance:

- A runner declared online reads online for four hours after the apply.
- A runner declared offline reads offline from the moment of the apply.
- A seeded runner, online or not, takes no work: a ready chunk stays ready until a real runner claims it.

## Filler generated from a few numbers (product:generated-filler)

Where a fixture needs bulk that nobody will read closely, such as a transcript, a long thread, a usage
history, or a stream of events, it asks for it by shape (a transcript of forty turns, a week of hourly usage) instead
of spelling it out, and the filler is generated to look like the real thing at a glance.

Acceptance:

- A fixture asks for a transcript by its number of turns and gets one the board renders as a real conversation, with
  turns from both sides and tool use among them.
- The same fixture applied to two freshly reset envs gives the same filler, word for word.
- Changing a fixture's seed gives different filler of the same shape.
- A fixture can mix the two in one place: a transcript of generated turns that ends on a turn it spells out.

## Worlds at stress scale (product:fixture-scale)

An agent can grow a tenant to the size where the fleet's limits show, hundreds of projects or thousands of
chunks and on to a hundred thousand, by a short script that applies fixtures over and over while changing only a few
values, such as the prefix, the seed, and a count.

Acceptance:

- A script applying one fixture a thousand times, each with its own prefix and seed, gives a thousand distinct copies
  with no collisions.
- A tenant reaches a hundred thousand chunks this way, spread over hundreds of projects, each chunk tied to an issue on
  the mock forge.
- Rerunning the same script on the same env writes nothing new, so a run cut short is finished by running it again.
- How long the largest world takes to build is stated beside the library, so an agent knows what to expect before it
  starts.

## Every env starts from the same world (product:starter-world)

A freshly provisioned feature env already holds the starter world, applied when the env is provisioned, so
every agent starts from the same believable fleet with its edge cases in place, and an empty board is something an agent
asks for rather than finds.

Acceptance:

- A newly provisioned env's board shows every chunk status, runners online and offline, and a backlog not yet ingested,
  with no command run by the agent.
- The runner panel of a newly provisioned env shows a parked ask and an open takeover.
- Provisioning an env that already holds state resets it first, so it too holds exactly the starter world afterwards.
- An agent that needs an empty env gets one by resetting it: the hub and runner stores, the mock forge's issues and pull
  requests, and the record of what earlier applies created.
- A reset env's hub holds no forge, source, or repository until a shell fixture is applied, and says so when asked to
  ingest.

## A starter library (product:starter-library)

The library starts with at least one fixture of every kind, so a newcomer to fixtures copies a working
example rather than starting blank.

Acceptance:

- Each kind has a fixture that applies on top of the fixtures its kind depends on.
- The library holds a set whose backlog and activity read as a believable night of a team's work.

## Applying a fixture reconciles (product:fixture-reconcile)

A fixture is applied to an env, and whatever it declares that the env lacks is created; nothing the env already holds is
removed.

Acceptance:

- Applying the same fixture to the same env a second time writes nothing.
- After a fixture is extended, applying it again creates what was added without a reset.
- Nothing an apply finds that the fixture does not declare is removed, whether an earlier fixture or a person made it.
- A record the env already holds that the fixture declares differently is left as it is and reported unreconciled,
  naming the field. Changing a world's existing records is a reset and a fresh apply, or the product's own routes.
- The records the hub creates with every new tenant, its first project and urgency, are the one exception: a fixture
  that declares them adopts them, and its first apply brings them to what it declares.
- An apply reports what it created, what it left alone, and what it could not reconcile.

## One world across every holder of state (product:one-world)

The fixtures of one apply keep the hub store, the runner store, the mock forge, and the git origins consistent with each
other.

Acceptance:

- A chunk a fixture ties to an issue shows that issue's title, body, and thread on the board.
- A chunk a fixture places in delivery has a pull request on the mock forge and a branch in the git origins that agree
  with it.
- A world outlives a restart of the env's services, the mock forge's included.

## Worlds read like real work (product:realistic-content)

A fixture's text reads like a real team's work, and the world it builds is the same on every apply.

Acceptance:

- The library's fixtures carry titles, bodies, questions, and threads written for their world, with no placeholder text.
- The fixture schema and a guide to writing fixtures are published beside the library, so an agent turns a short brief
  into a fixture that applies.
- Applying a fixture makes no model calls.

## Fixtures replace scenarios (product:scenario-retirement)

Fixtures are the one surface for whole worlds.

Acceptance:

- Every edge state `scenario board` and `scenario fleet` build, `--stress` included, is in the starter world.
- Once it is, the `scenario` verbs are removed.
