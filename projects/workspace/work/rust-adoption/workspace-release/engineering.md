# Workspace release engineering requirements

For `epic:rust-adoption/workspace-release`.

## Shared release integration (engineering:release-consumer)

Workspace integrates its release unit with `epic:platform/release-automation` and inherits
`engineering:release-versions`, `decision:shared-release-capability`, and the applicable shared governance. Product
builds, artifact metadata, and reported versions identify the same independently released Workspace version. Serves
`product:complete-workspace-install`.

## Tested artifact completeness (engineering:artifact-completeness)

Released artifacts contain everything needed for first-party Workspace behavior except declared operating-system, Git,
and provider runtime prerequisites; no first-party Python program is required. Installation smoke tests run help,
version, context navigation, and representative isolated Workspace behavior on every supported target. Serves
`product:complete-workspace-install`.

## Compatibility evidence (engineering:compatibility-evidence)

Release acceptance records the tested platform handoff, accepted configuration versions, and extension protocol range,
with rejection tests for unsupported versions. Existing workspace files and caches are not modified by artifact
installation or smoke tests. Serves `product:workspace-upgrade-guidance`.

## Replacement acceptance gate (engineering:replacement-evidence-gate)

Release acceptance checks the capability inventory against approved outcomes, provider conformance and runtime results,
and migration/handoff evidence; unresolved required outcomes fail replacement qualification. Artifact generation can
precede this gate, but generated artifacts alone cannot qualify a replacement release. Serves
`product:replacement-release-readiness`.
