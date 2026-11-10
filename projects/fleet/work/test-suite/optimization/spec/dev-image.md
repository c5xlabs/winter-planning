# The dev image

Paths are relative to the `blizzard` repo root.

## CI

```yaml
# .github/workflows/push.yml — engineering:amd64-dev-image
dev-image:
  name: build + publish dev hub image (edge + sha)
  needs: [gate, upper-tiers, dev-build, rehearsal]
  steps:
    # … checkout, the dev wheel, the tags and annotations, as today
    # docker/setup-qemu-action is removed
    - name: set up buildx
      uses: docker/setup-buildx-action@v3
    # … log in to GHCR
    - name: build and push the hub image (linux/amd64), retried on a transient GHCR error
      run: |
        # … the retry loop, as today
        docker buildx build \
          --file packaging/docker/Dockerfile \
          --platform linux/amd64 \
          --push \
          --metadata-file "$metadata" \
          "${tag_args[@]}" \
          "${annotation_args[@]}" \
          .
    # edge and sha-<sha> are single-platform; signing and provenance are unchanged
    - name: sign the pushed image (keyless)
      run: cosign sign --yes --recursive "ghcr.io/paul-gross/blizzard-hub@${IMAGE_DIGEST}"
    - name: attest build provenance of the pushed image
      uses: actions/attest-build-provenance@4d101475d8b20a2381f78447822ac1eab6504dd8 # v4.2.2
```

```yaml
# .github/workflows/release.yml — unchanged: QEMU, platforms: linux/amd64,linux/arm64, and a bill of materials per
# platform. product:arm64-releases
```

## Docs

`docs/ci.md`, beside the dev channel's description: the first arm64 build of any change is the release candidate's.
