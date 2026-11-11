# CLI context engineering requirements

Engineering for `epic:cli-context`.

## Markdown source and delivery (engineering:context-markdown-source)

Store authored context as human-readable Markdown files in the Blizzard repository and serve it as Markdown through the
CLI. Packaging may embed generated constants or include packaged Markdown resources, but the editable source of truth
remains the `.md` files; this serves `product:blizzard-usage-context` and `product:installed-context`.

Acceptance:

- Contributors can read and edit every authored topic directly as a `.md` file, with ordinary Markdown headings,
  paragraphs, lists, and code fences; guidance is not maintained as hard-coded strings in application code.
- Any embedded constants or other packaged representation are generated from those sources during build or packaging,
  not edited independently. Source-to-payload checks detect missing topics and stale or altered embedded content.
- Topic output preserves the authored Markdown body rather than an independently authored or HTML-rendered version.
  Index pages follow the same source and delivery path as leaves; generated metadata and navigation do not replace their
  authored bodies with a synthetic root or tree.
- The installed command reads its packaged or embedded representation without needing the repository, a
  current-directory relative file, or a source-tree path; tests cover the artifact produced by the packaging step.

## Bounded resource lookup (engineering:context-resource-boundary)

Resolve topic addresses through a fixed catalog of packaged resources, not a path supplied to the filesystem. This
serves `product:context-navigation` and keeps local-document discovery outside the packaged-resource boundary.

Acceptance:

- Absolute paths, traversal segments, encoded traversal, and arbitrary file names cannot select an unlisted resource.
- Catalog validation detects duplicate or ambiguous addresses, absent resources, missing read-trigger cues, unreadable
  directory indexes, and navigation targets missing from the catalog.
- Directory aliases resolve only to their cataloged `index.md`; explicit indexes and their aliases return the same
  canonical document. Missing indexes do not trigger filesystem discovery or a synthesized listing.
- Resolution is independent of the caller's current directory and never imports or executes a document as code.

## Packaged proof (engineering:context-package-proof)

Exercise the installed distribution rather than relying on editable-checkout resource access. This serves
`product:installed-context` and `product:workspace-participation`.

Acceptance:

- A built-wheel test installs into an isolated environment, reads every catalog entry from outside the source tree, and
  confirms document version metadata agrees with the installed package.
- Tests deny network access and use an empty home, no credentials, no workspace configuration, and no running daemons;
  reading the corpus changes none of those locations, including session/configuration stores.
- Existing daemon settings and tracing configuration cannot make a context read contact a service or start a daemon.

## Checked operating examples (engineering:context-example-proof)

Hold runnable navigation and command examples to the shipped command surface, and keep operational facts at one source
instead of copying an independent command manual into the context corpus. This serves `product:factory-operation` and
`product:workspace-participation`, with topic coverage following `product:task-taxonomy`.

Acceptance:

- Every routed topic command resolves through the public CLI, and structured command examples validate against its
  argument/option surface without executing mutating examples.
- A fixture walkthrough uses fake identifiers and a controlled hub boundary to inspect a selected project's chunk and
  delivery evidence; it neither discovers sibling checkouts nor accesses a real fleet.
- The fixture has an explicit missing-binding/missing-CLI path. Coverage checks keep role, target, authentication,
  delivery, and mutation cautions reachable from the root and the adoption example.
- Corpus and entry-hook maintenance follow the product's applicable context conventions; changes to a documented command
  update its example validation in the same change.

## Usage-catalog isolation (engineering:context-usage-boundary)

Keep context resolution independent of artifact scopes, graph declarations, configured prompts, and leased worker
identity. This serves `product:blizzard-usage-context`; packaged-resource helpers may be reused without coupling the
command to factory-data resolution.

Acceptance:

- Context reads succeed without worker environment variables and never invoke worker, artifact, graph, or prompt
  clients; tests make those calls fail if attempted.
- An artifact name absent from the context catalog is refused as an unknown topic, not resolved through another scope.
- Changing runtime prompts or factory records cannot change the installed usage corpus; existing artifact and prompt
  command contracts remain unchanged.
