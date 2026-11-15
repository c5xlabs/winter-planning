# The domain boundary

The frontend's own domain, and the one place the wire is translated (`decision:ui-toolkit/frontend-domain`).

## System

```text
generated client (fleet/src/lib/api/{hub,runner})          wire types: CadenceCreateRequest, Problem, …
      │
*.query.ts · *.mutations.ts      data access: the only callers of the client (unchanged)
      │
*.translate.ts                   wire ⇄ domain, refusal → domain field; the only other importers of wire types
      │
*.domain.ts                      domain types and pure domain functions; no Angular, no wire
      │
*.service.ts                     inject*Service(): domain in, domain out; composes data access and translators
      │
container component  ──▶  presentational view  ──▶  kit
```

| Concept file                                   | Imports wire types | Imports Angular               | Pinned by                                   |
| ---------------------------------------------- | ------------------ | ----------------------------- | ------------------------------------------- |
| `<concept>.query.ts`, `<concept>.mutations.ts` | yes                | yes                           | existing specs                              |
| `<concept>.translate.ts`                       | yes                | no                            | `<concept>.translate.spec.ts`, no `TestBed` |
| `<concept>.domain.ts`                          | no                 | no                            | `<concept>.domain.spec.ts`, no `TestBed`    |
| `<concept>.form.ts` (validation rules)         | no                 | `@angular/forms/signals` only | `<concept>.form.spec.ts`                    |
| `<concept>.service.ts`                         | no                 | yes                           | `<concept>.service.spec.ts`                 |
| `*.model.ts`, components, views                | no                 | per existing rules            | existing specs                              |

Focused boundary tests (`*.query.spec.ts`, `*.mutations.spec.ts`, `*.translate.spec.ts`) may import wire types to pin
their boundary's contract. Wire builders needed by those tests live in pure `*.translate.ts` fixture modules. All other
specs and stories use domain builders and remain in the boundary sweep.

A unit's files sit in its feature folder, as `*.query.ts` and `*.model.ts` do today, under the existing
`bzh:frontend-placement` and `bzh:frontend-package-layers` tables. A concept two features share goes down to the
project's kernel (`core/`), or to `fleet` where both apps reach it.

| blizzard-context                                                                                         | Change                                                                                                                                                                           |
| -------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `architecture/frontend-structure/wire.md`                                                                | `bzh:frontend-wire-conformist` replaced by `bzh:frontend-domain-boundary`, its pending-override exception reworded with it; `bzh:frontend-open-vocabulary` scoped to translators |
| `architecture/frontend-structure/containers.md`                                                          | `bzh:frontend-containers-compose`: a judgment once sent to the wire lives in a `*.domain.ts`                                                                                     |
| `architecture/frontend-structure/mutations.md`                                                           | standardizes versioned form edits on the shared `EditBaseline` contract in `spec/forms.md`: one captured read, unchanged by refresh, sent as `If-Match`, and retained on refusal |
| `architecture/frontend-structure/placement.md`                                                           | documents the temporary `core/forms` bootstrap exemption and its removal when both apps adopt awaiting-human forms (`decision:ui-toolkit/shared-form-bootstrap`)                 |
| `architecture/frontend-structure/kit.md`                                                                 | `bzh:frontend-kit-floor` dependency list (`decision:ui-toolkit/kit-dependencies`)                                                                                                |
| every other file that cites the old rule, in blizzard-context and in blizzard's own source and exemplars | the citation replaced                                                                                                                                                            |
| `verification/blizzard/commands/web/static-checks.md`                                                    | the boundary sweep's scope, report, and exemption list; the open-vocabulary sweep's narrowed scope                                                                               |
| `exemplars/` web slice                                                                                   | gains cadence's versioned-edit example and proposal acceptance's completed-toolkit assembly example, with their domain, translator, form, and service                            |

| `web/` gate                    | Change                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| ------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Domain-boundary detector (new) | fails a production file, spec, or story that imports a symbol originating in generated `types.gen.ts`, except data-access/translator files, their focused specs defined above, and `fleet/src/lib/api/`; resolves TypeScript symbols through path aliases, renamed imports, namespace members, and transitive named or `export *` barrel re-exports, including `fleet`; checks the declaration's origin, not the immediate import source or spelling, so a domain type may share a wire type's name; a unit on `DOMAIN_BOUNDARY_EXEMPT` passes |
| `DOMAIN_BOUNDARY_EXEMPT` (new) | every unit that reads a wire type outside the boundary when the sweep lands; a unit leaves it in the landing that moves it; empty at delivery                                                                                                                                                                                                                                                                                                                                                                                                  |
| `wire-conformist-sweep.js`     | renamed for the new rule; its client-call and backend-citation detectors are kept as they are                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| `open-vocabulary-sweep.js`     | scoped to translators, where wire vocabularies are read                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| `eslint.config.js`             | restricted imports, as `*.model.ts` has today: no Angular in `*.domain.ts` or `*.translate.ts`; only `@angular/forms/signals` in `*.form.ts`; `@angular/aria` only in the kit                                                                                                                                                                                                                                                                                                                                                                  |

| Detector fixture                                                                          | Result outside the boundary, without an exemption |
| ----------------------------------------------------------------------------------------- | ------------------------------------------------- |
| Direct generated import, including an alias                                               | fail                                              |
| Generated type imported from `fleet`, through named or star re-exports                    | fail                                              |
| Generated type reached through a second barrel, a namespace member, or an `import()` type | fail                                              |
| Domain declaration with the same name as a generated type, imported through a barrel      | pass                                              |
| Generated import in a data-access file or translator                                      | pass                                              |
| Generated import in its focused query/mutation/translator spec                            | pass                                              |
| Generated import in a component/model spec or story, including through a builder barrel   | fail                                              |

The detector uses each project's TypeScript module resolution. Existing wire re-exports remain usable by data access and
translators; a barrel does not grant another importer a boundary exemption.

Until a feature moves, its existing query export may keep the old wire projection for exempt callers while a domain
projection is added in the same data-access file (`decision:ui-toolkit/incremental-replacement`). The cache, query keys,
and invalidations remain shared; no component-side translation or duplicate cache is introduced. Once all callers move,
the old projection and its compatibility exports are removed.

## Domain

```ts
// bzh:frontend-domain-boundary — a wire vocabulary becomes a closed domain union with an explicit unknown.
// The translator is the only place an unforeseen wire value is handled (bzh:frontend-open-vocabulary, now scoped here).
export type ChunkStatus =
  | 'not-ready' | 'ready' | 'running' | 'delivering' | 'waiting-on-human' | 'needs-human'
  | 'paused' | 'stopped' | 'done' | 'unknown';

// A judgment the wire already carries, such as a chunk's `terminal`, is translated, not re-derived. A judgment the
// frontend makes from what it holds is a pure domain function, pinned by its spec.
```

## Interfaces

```ts
// garden/core/cadences.query.ts — a read hands its consumers domain types; the cache keeps the wire response.
export function injectHubCadencesQuery() {
  const project = injectProjectFilter();
  return injectQuery(() => ({
    queryKey: lensedKey(hubCadencesKey, project()),
    queryFn: async (): Promise<CadenceView[]> => { /* as today */ },
    select: (wire: CadenceView[]): readonly Cadence[] => wire.map(fromCadenceView), // translator
  }));
}

// garden/core/cadence.translate.ts, beside garden/core/cadence.domain.ts: in the kernel, because the query in
// garden/core and more than one garden feature read them.
export function fromCadenceView(wire: CadenceView): Cadence;
export function toDraft(cadence: Cadence): CadenceDraft;
export function toEditBaseline(cadence: Cadence): EditBaseline<CadenceDraft>; // id, revision, and detached draft
export function toCreateRequest(draft: CadenceDraft): CadenceCreateRequest;
export function toEditRequest(before: CadenceDraft, after: CadenceDraft): CadenceEditRequest; // sparse: changed only
export const CADENCE_REFUSALS: RefusalMap<CadenceDraft>; // spec/forms.md
```

`Cadence` carries the hub's `id` and `revision` as domain data. `EditBaseline` and its edit/save lifecycle are defined
once in [forms.md](./forms.md#domain); every versioned form service uses that contract.

A view-model file (`*.model.ts`) that derived from a wire view today takes the domain type instead. Its spec builds
domain literals, not wire literals, and `fleet/testing/builders` gains a domain builder beside each wire builder that a
translator spec still needs.
