# Releases and recovery

Core releases use a verified SemVer tag pointing to a commit on main. Catalog
releases use `catalog-<full-commit-sha>` after both pinned userspaces qualify every
matrix entry. Documentation-only changes do not publish catalog releases. Older
published snapshots and source pins are retained without rewriting history.

## Core candidate and installation

The [v0.1.0 qualification record](release-qualification-v0.1.0.md) lists the
target matrix and names the exact main commit and CI run. For any candidate,
check that the run for that commit includes both target artifacts and the kernel
acceptance jobs. A green run that skipped those jobs is not release evidence.
Review the latest dependency audits before approving the tag; a historical green
run does not override a newer failure. Retain the CI reports and candidate digests
before their Actions retention expires.

Build jobs have read permissions. The final trusted publication job downloads the
qualified artifacts, verifies SHA256, attaches license files and SPDX SBOMs,
requires a successful Grype report, and requests GitHub OIDC provenance and SBOM
attestations using the pinned `actions/attest` action. Releases are published as
immutable assets. A workflow file alone is not evidence that a release was qualified.

## Verify before installing

Before using a downloaded artifact, verify its checksum and expected repository
and workflow identity. The signed subjects for core are the `.tar.gz` archives,
not the Actions ZIPs, loose binaries or `.sha256` sidecars. Set `expected_commit`
from the reviewed release identity, not from the artifact.

```sh
set -eu
artifact=srx-x86_64-unknown-linux-gnu.tar.gz  # or the musl archive
sha256sum --strict --check "$artifact.sha256"
for predicate in https://slsa.dev/provenance/v1 https://spdx.dev/Document; do
  gh attestation verify "$artifact" --repo ryro-hq/srx \
    --signer-workflow ryro-hq/srx/.github/workflows/release.yml \
    --source-digest "$expected_commit" --deny-self-hosted-runners \
    --predicate-type "$predicate"
done
```

For release qualification, compare the attested archive's internal SPDX file
with the published `.spdx.json` copy and retain the verification results.

The publisher exports `sha256:DIGEST.jsonl` attestation bundles and a
`trusted_root.jsonl` snapshot and verifies both provenance and SPDX predicates
using those files before publication. On a machine with network access, obtain
and retain the bundle and trust roots through trusted channels before going
offline, for example with `gh attestation download "$artifact" --repo ryro-hq/srx`
and `gh attestation trusted-root > trusted_root.jsonl`. A root shipped with an
untrusted artifact does not establish trust. In a disconnected environment with
a GitHub CLI version that supports these flags, set `artifact` and
`expected_commit` as above and run both checks:

```sh
set -eu
sha256sum --strict --check "$artifact.sha256"
digest=$(sha256sum "$artifact" | cut -d' ' -f1)
for predicate in https://slsa.dev/provenance/v1 https://spdx.dev/Document; do
  gh attestation verify "$artifact" --repo ryro-hq/srx \
    --bundle "sha256:$digest.jsonl" --custom-trusted-root trusted_root.jsonl \
    --signer-workflow ryro-hq/srx/.github/workflows/release.yml \
    --source-digest "$expected_commit" --deny-self-hosted-runners \
    --predicate-type "$predicate"
done
```

Record success for the original archive and rejection of a modified copy against
the original bundle. The artifact digest, issuer and expected signer workflow
must all match. An offline verifier cannot learn about a subsequent revocation;
check security advisories and refresh trusted roots before importing new material.

Only after verification, extract into a new directory and install to a
user-owned prefix. `install-self` installs the worker embedded in `srx`.

```sh
set -eu
prefix="$HOME/.local/srx"
payload=$(mktemp -d)
tar -xzf "$artifact" -C "$payload"
(cd "$payload" && sha256sum --strict --check SHA256SUMS)
chmod u+x "$payload/srx"
"$payload/srx" install-self --prefix "$prefix"
"$prefix/bin/srx" --version
cmp "$prefix/current/syrox-worker" "$payload/syrox-worker"
```

## Recovery and revocation

If a release or account is compromised: revoke the relevant App installation/key
and active tokens, restrict publishing, preserve Actions runs/audit logs and hashes,
assess source and Store exposure, publish a GitHub Security Advisory identifying
exact affected/fixed identities, then rebuild and attest a replacement through the
normal queue. Do not silently replace an immutable release or remove retained lock
history. Tell operators explicitly how to identify and remove affected cached or
installed bytes. Never claim that an updater or bot can recall those bytes.

For a non-production exercise, record the affected tag or candidate commit,
target, and SHA-256 of each affected archive/binary. Use disposable caches and
installation prefixes to identify bytes by those hashes, preserve a quarantined
copy, remove the affected copies from active use, and confirm that the candidate
no longer resolves on the test `PATH`. Record who would restrict publishing,
revoke credentials, retain audit evidence, issue the advisory and authorize the
replacement. Use a separate test App or repository if practicing live administrative
steps; a local quarantine or tabletop is not evidence of credential revocation.

[Maintainer governance](../../GOVERNANCE.md#access-review-succession-and-recovery)
identifies who owns recovery and defines access review and handoff. The
[v0.1.0 local quarantine drill](release-qualification-v0.1.0.md#installation-example-and-recovery)
is evidence of local recovery only; a non-production credential revocation and
replacement exercise remains pending. See the
[controls record](controls.md) for outstanding independent-review evidence.
