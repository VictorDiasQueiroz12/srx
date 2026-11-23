# Core v0.1.0 release candidate

**Decision as of 2026-11-22: do not tag or publish this commit.** Priority P1 is
proposed for the backlog, not a delivery commitment. The evaluated candidate
was the `syrox` Cargo version `0.1.0` at main commit
[`68567abb9c751459d7c737cec6056a3dd17cbaa1`][commit]. `v0.1.0` is the proposed
release tag, not an existing tag. The [CI run][ci] succeeded for that exact
commit, including both builds and all three kernel lanes. Later same-commit
[scheduled CI][later-policy] and the [independent audit][later-audit] failed on
[`RUSTSEC-2026-0592`][advisory] in `url 2.5.8`. The earlier green run is build
evidence, not a reason to publish. No core release or `release.yml` run was
listed on GitHub at review time. This is not a certification or a SLSA level.

## Dependency fix on the qualification branch

On 2026-11-23, the qualification branch applied the same dependency fix as
[PR #40][url-fix]: `url 2.5.9` in `syrox` and `syrox-engine`, with
`form_urlencoded 1.2.3` in `Cargo.lock`. No policy exception was added.
Locked Cargo metadata resolved those versions, and cargo-deny `0.20.2`
passed advisories, bans, licenses and sources locally. A comparison using
`cargo deny --locked --offline check` and the same advisory data reproduced the
audit failure at `68567ab` and passed with the patch. The policy expiry check
found the same six scoped exceptions unexpired. This checks the patched
source, not the old binaries or a new release. The
[latest main audit at review time][newer-audit] still failed on `url 2.5.8`;
all artifact and VM evidence below remains scoped to `68567ab`.

The portable suite passed locally (603 passed, 6 ignored). The 10 default
GitHub-acquisition tests and the opt-in live HTTPS acquisition test passed
with the patched dependency. The latter reopens a pinned catalog snapshot
offline; it does not verify a core release attestation.

## Target matrix

| OS | Architecture | Rust target / linked libc inventory | Qualification |
| --- | --- | --- | --- |
| Linux | x86-64 | `x86_64-unknown-linux-gnu` / glibc (`libc6-dev` `2.39-0ubuntu8.12`) | [Build][gnu-build], [VM acceptance][gnu-vm], [native suite][native-vm], local install |
| Linux | x86-64 | `x86_64-unknown-linux-musl` / musl (`musl-dev` `1.2.4-2`) | [Build][musl-build], [VM acceptance][musl-vm], local install |

Both archives contain a static x86-64 ELF `srx` and `syrox-worker` without
`PT_INTERP` or `DT_NEEDED`. The libc versions above come from the build inventory;
they are not minimum host libc requirements. Builds used Ubuntu 24.04 and
Rust 1.98.1. Native execution evidence is scoped to the pinned CI guest and
Linux 6.12.109; it does not qualify all distributions, older kernels, ARM64,
32-bit Linux, macOS or Windows. Running packages involves additional kernel,
systemd user manager, Bubblewrap and Landlock requirements. The core target
matrix is separate from the catalog's package support.

## Candidate bytes and dependency evidence

The CI build jobs produced the binaries and payloads. Both Actions ZIP
SHA-256s matched the Actions API metadata. Each internal `SHA256SUMS` covered
all 528 payload files; on each target the two ELF files matched `manifest.json`,
and all 872 source digests in that manifest matched files in the pinned commit.
The VM reports' guest binary digests match the CLI and worker digests below.
Actions artifacts are access- and retention-limited; these links were listed
as expiring on 2027-02-20. Retain them before expiry.

| Evidence (SHA-256) | GNU | musl |
| --- | --- | --- |
| Actions ZIP | [GNU ZIP][gnu-zip] `e180b3bdb39f99122e3a2ce2b996dcf19dea8d0af32e134ec1086a6ab8f453c7` | [musl ZIP][musl-zip] `a2b67b08cbe9f2ed1408e19337af56eac7d58fee1ed6827198037062c41a6ae4` |
| `srx` | `ca6c67fd878e89634b09fe9e51e74b4c86a19f43eaa5215ce1cf183b9c8139e9` | `6a878b11997d50e4e7f257bc08d3365ed65657589f9b6a4ddbeb01f408d59a3a` |
| `syrox-worker` | `7cdaf410fbcd57bd83c2fba8eb837d35919b15411edaeaeebdae6a98bf1afbd4` | `61e371a96d0d0b5475ca1291fff2523825e6e7266c2f35420f5c58f975b7c8b3` |
| `sbom.spdx.json` | `f2ad543971072f1455f39c2293a13b249faadd887508b7ad88d40003892f2ee6` | `d8277a9d259a77486b69340b2f222c1fb1ad09f63b2d5dd2aa48f4627457825a` |
| `grype.json` | `f1982502de24e009cf862ad52b8b33a26d882c113b888c198fc13b6946529c7f` | `e4e07d6da18e30cb9537c00d42d5252bf48b7a773d63279e3233571674f005a6` |
| Local unsigned `srx-<target>.tar.gz` | `29f3361f525ebdea8dd9f548a0cde83572040d947c975a5ce3cc595e89b1929d` | `03c263810d4718cc05fbb5ceb3d815aef2ac8f251883944c86a73b233745ee8a` |

The last row records local test packages made from the verified payloads and
checked against generated `.sha256` sidecars. It is not a prediction of
publication digests; repacking or file metadata can change the archive hash.

- **SPDX:** Both inventories validated against the [SPDX 2.3 JSON schema][spdx]
  and each listed 187 packages. The listed Cargo and native binary archive
  checksums matched the retained archives. This is a content check, not a
  signed SPDX binding or independent proof of complete coverage.
- **Vulnerabilities:** Both Grype `0.121.0` reports used a valid `v6.1.10` DB
  built `2026-11-21T06:20:06Z` and reported zero matches at scan time. The
  [push dependency-policy job][cargo-policy] passed cargo-deny advisories,
  bans, licenses and sources, and found six scoped exceptions unexpired.
  That finding was superseded for release purposes by the two later audit
  failures. Rebuild with `url` at `2.5.9` or newer on main; rerun checks with
  current databases and recheck exceptions.
- **Licenses and source:** Each payload included 332 files under `licenses/`
  (including the project license, Rust notices and native copyright files),
  178 Cargo archives and 13 native archives (including Ubuntu source and
  packaging/patch inputs). Neither package contains a license/notice copy for
  `yaml-edit 0.3.6`; the retained crate has an `Apache-2.0` declaration and a
  README referring to `LICENSE`, but no such file. The crate's recorded
  [upstream revision][yaml-edit] also has no top-level license file.
  Resolve upstream license text/notices and review static GNU relinking/source
  obligations before approval. A green Cargo license policy check does not close
  either item.

## Installation, example and recovery

- **Installation:** Both local archives were extracted and verified in an
  isolated Linux network namespace with no routes. Twice per target,
  `srx install-self --prefix <disposable-directory>` succeeded with
  `PATH=/nonexistent`. Installed `srx --version` printed `srx 0.1.0`,
  `host inspect` succeeded, and the installed CLI and worker matched the
  payload byte for byte.
- **Build and run:** `host_build_prefix_materializes_and_runs_at_its_action_path`
  passed on both [GNU][gnu-vm] and [musl][musl-vm]; see `guest.gates` in
  `test-evidence/ci-vm/report.json` and `gate-1.log` in the respective
  [GNU][gnu-vm-zip] and [musl][musl-vm-zip] artifacts. This builds and runs a
  static fixture with the actual candidate. It is not a catalog Hello result.
  The local musl Hello 2.12.1 sample used catalog commit
  [`fa061bf30399571661f9cf52601707dad9d554a2`][catalog]. The candidate refreshed
  the catalog lock in a disposable clone, checked, locked and planned the demo
  successfully. The local build/run failed because the host `setpriv` rejected
  the `ioctl-dev` Landlock right. Rerun Hello on a qualified host; the core VM
  fixture is not a substitute for the catalog example.
- **Offline verification:** The outer and inner checksum checks passed
  without network; appending one byte to a copy caused the original checksum
  check to reject it. GitHub CLI 2.105.0 with independently fetched trust roots
  failed closed for each target and both predicates because the official
  bundles did not exist. This is a negative test, not successful offline
  verification of provenance or SPDX.
- **Recovery drill:** For both targets, the operator exercise identified
  affected cache archives and installer-managed CLI/worker bytes by SHA-256,
  moved them into quarantine, removed the selection links, and confirmed the
  candidates no longer resolved on the disposable `PATH`. Quarantined bytes
  retained their digests. Repository maintainers would restrict publishing and
  retain the run, logs and digests; the affected App/account owner would
  revoke credentials; the security maintainer would coordinate the advisory
  and replacement through the normal queue. No live credential revocation,
  audit-log export, advisory or replacement was performed.

## Before closing qualification

1. Confirm the `url` fix is on main and resolve the license/notice and static
   GNU distribution review. Select a new commit and preserve its successful
   build, scan and kernel evidence, plus a current passing dependency audit.
   Do not tag the rejected commit or an unmerged branch.
2. Before tagging, have a maintainer check that the [release workflow][release]
   can read artifacts from the qualified CI run (`actions: read`) and that
   the [immutable-release setting][immutable] is enabled (checking it requires
   administrative read access). With separate approval for publication, verify
   both final archive digests and their GitHub OIDC provenance and SPDX
   attestations online and offline, including rejection of a modified copy.
   Retain the bundles, trusted-root snapshot, verifier output and immutable
   release link.
3. Reinstall from the actual signed archives and rerun GNU Hello on a qualified
   host. Retain the build output and expected greeting.
   Repeat the recovery exercise with the maintainers in a non-production
   setting that permits real credential revocation and replacement; link the
   evidence and the published assets in [controls.md](controls.md).

[commit]: https://github.com/ryro-hq/srx/commit/68567abb9c751459d7c737cec6056a3dd17cbaa1
[ci]: https://github.com/ryro-hq/srx/actions/runs/44987638099
[gnu-build]: https://github.com/ryro-hq/srx/actions/runs/44987638099/job/137101678711
[musl-build]: https://github.com/ryro-hq/srx/actions/runs/44987638099/job/137101678679
[gnu-vm]: https://github.com/ryro-hq/srx/actions/runs/44987638099/job/137102681513
[musl-vm]: https://github.com/ryro-hq/srx/actions/runs/44987638099/job/137102681461
[native-vm]: https://github.com/ryro-hq/srx/actions/runs/44987638099/job/137102681478
[gnu-zip]: https://github.com/ryro-hq/srx/actions/runs/44987638099/artifacts/14740890504
[musl-zip]: https://github.com/ryro-hq/srx/actions/runs/44987638099/artifacts/14740746596
[gnu-vm-zip]: https://github.com/ryro-hq/srx/actions/runs/44987638099/artifacts/14740836450
[musl-vm-zip]: https://github.com/ryro-hq/srx/actions/runs/44987638099/artifacts/14740299933
[cargo-policy]: https://github.com/ryro-hq/srx/actions/runs/44987638099/job/137101594583
[later-policy]: https://github.com/ryro-hq/srx/actions/runs/45023574434/job/137211570310
[later-audit]: https://github.com/ryro-hq/srx/actions/runs/45023760083/job/137212160741
[newer-audit]: https://github.com/ryro-hq/srx/actions/runs/45200837785/job/137783572237
[advisory]: https://rustsec.org/advisories/RUSTSEC-2026-0592.html
[url-fix]: https://github.com/ryro-hq/srx/pull/40/commits/1b2cd8f488f5002c002448b5b82cac73d65e3e21
[spdx]: https://github.com/spdx/spdx-spec/blob/v2.3/schemas/spdx-schema.json
[yaml-edit]: https://github.com/jelmer/yaml-edit/tree/780a686e5b882941b710c0ef19fd8d061d03bcc8
[catalog]: https://github.com/ryro-hq/srx-pkgs/tree/fa061bf30399571661f9cf52601707dad9d554a2
[release]: https://github.com/ryro-hq/srx/blob/68567abb9c751459d7c737cec6056a3dd17cbaa1/.github/workflows/release.yml
[immutable]: https://docs.github.com/en/rest/repos/repos#check-if-immutable-releases-are-enabled-for-a-repository
