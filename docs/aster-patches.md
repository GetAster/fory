# Aster patch ledger: fory

This repository is Aster's fork of [apache/fory](https://github.com/apache/fory). Aster carries the patches listed below on top of upstream releases. Upstream maintainers are welcome to incorporate any of them, in whole or in part, at their discretion.

Aster consumes Fory's Rust crates (`fory-core`, `fory-derive`, version 1.7.7) unmodified from crates.io; they are not mirrored or republished. The patch carried here is in the C# implementation, delivered as an Aster-built `Apache.Fory` NuGet package.

## Branch and tag model

- `main` is upstream `main` plus one commit: the Aster README preamble and this ledger. Nothing is built from `main`. It is refreshed with `git merge upstream/main` and is never reset or force-pushed. Ledger edits are commits on `main`, never on `upgrade/*`.
- `upgrade/<repo>-v<upstream version>` (for example `upgrade/iroh-v1.4.0`, `upgrade/iroh-blobs-v0.103.1`) is the patched integration line: the upstream release plus the patches in this ledger. A new upstream base always gets a new branch, cut from the upstream release tag; Aster-only fixes and re-pins append to the current branch. Per-minor names (`upgrade/iroh-blobs-v0.103`, `upgrade/salvo-v1.0`, …) are historical. Consumers pin commits on it. Published `upgrade/*` branches and tags are never rewritten; a failed attempt is superseded by new commits and a new version.
- Topic branches carry one change each, are named for the change (for example `feat/read-into`, `security/plumtree-bounds`), are based on the upstream release tag, and are cherry-picked with `git cherry-pick -x` onto the upgrade branch, so every upgrade-branch commit names its source.
- Version: the upstream version if Aster's registry has never served that version or a higher one in the series; otherwise the next patch above Aster's highest published version. A re-pin without a code change keeps the version and takes the tag suffix `-p<n>`.
- Tags: `aster-<repo>-v<published version>` (re-pins `aster-<repo>-v<published version>-p<n>`), created only after the Aster consumer trial passes against those exact commits, because tags cannot be moved.
- The fork manifest is `iroh-fork-manifest.toml` in the Aster repository (the only copy).
- Aster does not submit patches upstream; upstream maintainers are welcome to take any patch here.

Historical and current tag schemes in this repository:

- `v<upstream version>-aster.<n>`: `v1.7.6-aster.1` (annotated; names the exact source of the `Apache.Fory 1.7.6-aster.1` package). Legacy scheme; this is the real 1.7.6 tag.
- `aster-fory-v<published version>`, for new tags, where the published version is the NuGet package version `<upstream version>-aster.<n>`: first `aster-fory-v1.7.7-aster.1`. `aster-fory-v1.7.6-aster.1` is only an illustrative example of the form; no such tag exists, because 1.7.6 was tagged `v1.7.6-aster.1` before this scheme.

## Current release

| Field | Value |
|---|---|
| Upstream base | `v1.7.7` (`04cbe11a86a640879e48fa7a8edee81056b37aab`) |
| Integration branch | `upgrade/fory-v1.7.7` @ `7a299ebce2d5c7b966ef7857ae1ba1af182f01a4` (the packaging commit) |
| Tag | `aster-fory-v1.7.7-aster.1` (annotated) → `7a299ebce2d5c7b966ef7857ae1ba1af182f01a4` |
| Package | NuGet `Apache.Fory` `1.7.7-aster.1` |
| Package feed | `https://forge.emrul.dev/api/packages/Aster/nuget/index.json` |
| Source repository | `https://forge.emrul.dev/Aster/fory` |
| Aster consumer | `bindings/dotnet/src/Aster/Aster.csproj` in the Aster repository |

The integration line is: upstream `v1.7.7`, then FORY-CS-001 (`3e7593710`, a clean `cherry-pick -x` of the 1.7.6 backport `e8109b29`, itself `-x` of `345e4c77`; both trailers kept), then the packaging commit `7a299ebc` (package version and repository URL only, in `csharp/Directory.Build.props`, kept separate from the logical fix). The ledger lives on `main`.

The previous line, `upgrade/fory-v1.7.6` (tag `v1.7.6-aster.1` → packaging commit `fdeb5cd0`, package `Apache.Fory 1.7.6-aster.1`), is unchanged; its four docs commits past the tag (`63af890b9`…`1c4521745`) are historical, from before the current model, and no further ledger commits go on `upgrade/*`.

Branches (all verified anonymously on Forgejo, with no Git credential helper, where a commit is given):

| Branch | Role | Commit |
|---|---|---|
| `fix/csharp-metadata-publication` | Canonical topic branch for FORY-CS-001, on upstream `main` `261dc92eabdc5c0fe08748a3ba476ae13f4e8bc9` | `345e4c771ccb4a895181fe8cb244ef2b27fb0196` |
| `backport/csharp-metadata-publication-1.7.6` | Canonical backport onto `v1.7.6` | `e8109b29604aa409d0b65313d7d52eca9c4415fe` |
| `delivery/csharp-1.7.6-aster.1` | Backport + packaging commit + the historical ledger commits; source of the published package | tag `v1.7.6-aster.1` at `fdeb5cd01baa208d128fff7d0c06e3b3e30481e5` |
| `delivery/csharp-metadata-publication` | The mainline fix plus this ledger | — |
| `upgrade/fory-v1.7.6` | Previous integration branch; same tip as `delivery/csharp-1.7.6-aster.1` | — |
| `upgrade/fory-v1.7.7` | Current integration branch: `v1.7.7` + FORY-CS-001 + packaging; source of `Apache.Fory 1.7.7-aster.1` | `7a299ebce2d5c7b966ef7857ae1ba1af182f01a4` (tag `aster-fory-v1.7.7-aster.1`) |

Aster has adopted it: the Aster .NET binding pins `Apache.Fory [1.7.7-aster.1]` (from 2026-10-21; `[1.7.6-aster.1]` before), and Aster's Rust, Java, Python, TypeScript and Go bindings moved to upstream Fory 1.7.7 in the same change, so every binding is on 1.7.7.

Consumer status at the time of the 1.7.6 delivery (2026-10-08): the package and its source were published, but Aster's dependency adoption and release qualification were pending (Aster then used upstream `Apache.Fory 1.7.6`, with no fork pin). Adoption requires Aster to:

- pin the exact version in `bindings/dotnet/src/Aster/Aster.csproj` (`Version="[1.7.6-aster.1]"`);
- map `Apache.Fory` to the Forgejo source explicitly, keeping nuget.org for unrelated dependencies (configuration below);
- run its concurrent RPC integrity test and normal .NET release gate against a fresh exact candidate;
- keep its dependency and NuGet provenance records in agreement with this ledger (the Rust BOM is not NuGet provenance).

These steps do not follow automatically from Fory unit tests.

NuGet source configuration for CI and downstream consumers (do not rely on a developer's global NuGet configuration):

```xml
<configuration>
  <packageSources>
    <clear />
    <add key="forgejo" value="https://forge.emrul.dev/api/packages/Aster/nuget/index.json" />
    <add key="nuget.org" value="https://api.nuget.org/v3/index.json" />
  </packageSources>
  <packageSourceMapping>
    <packageSource key="forgejo"><package pattern="Apache.Fory" /></packageSource>
    <packageSource key="nuget.org"><package pattern="*" /></packageSource>
  </packageSourceMapping>
</configuration>
```

Restore with this configuration into a fresh `NUGET_PACKAGES` directory and verify the resolved archive hash before running consumer gates.

## Patches

| Id | Topic branch | What | Why / what in Aster depends on it | Commits (topic / upgrade) | Status | Conflict or porting notes |
|---|---|---|---|---|---|---|
| FORY-CS-001 | `fix/csharp-metadata-publication` (based on upstream `main` `261dc92e…`) | fix(csharp): publish consistent metadata cache snapshots. The generator publishes an immutable snapshot of the resolver version and both local metadata header hashes with acquire/release semantics. Each invocation keeps its snapshot even if a different codec publishes another. A direct header comparison replaces the redundant remote-identity/Boolean memoisation; there is no separately published match flag. Wire bytes, public APIs, compatible field dispatch and parallel root decoding are unchanged. Files: `csharp/src/Fory.Generator/ForyModelGenerator.Emission.cs`, `csharp/tests/Fory.Tests/ForyGeneratorTests.cs`, `csharp/tests/Fory.Tests/MetadataPublicationTests.cs`. | Shared metadata identity/match writes and resolver hash replacement could return default field values during parallel decoding. Aster's .NET binding (`Apache.Fory` consumer) is the user; its concurrent RPC integrity test is the adoption gate. This is a C#-local cache correction: shared protocol, type mappings and wire semantics are unchanged. | `345e4c771ccb4a895181fe8cb244ef2b27fb0196` / backport `e8109b29604aa409d0b65313d7d52eca9c4415fe` (on `v1.7.6`, clean `cherry-pick -x`); `3e7593710` on `upgrade/fory-v1.7.7` (clean `cherry-pick -x` of `e8109b29`) | carried (not in upstream 1.7.7: its six regression tests fail on bare `v1.7.7`) | No prerequisite patches. Apply the fix first, then the separate packaging change (the ledger lives on `main`, not on the upgrade branch). The metadata emitter and runtime sources are identical between the two baselines (`261dc92e` and `v1.7.6`). Future changes to `VersionHash`, local metadata hash ownership, generated compatible reads or hierarchy providers need re-review. Never carry a cached Boolean separately from its metadata identity, nor read mutable shared hashes after checking only a shared resolver version. `VersionHash()` is a semantic configuration fingerprint, not a resolver-local counter, on both baselines, so equal counters with differing configurations are not a supported case to synthesise. Independent codec tests use different registration names, hence different metadata hashes and resolver versions. Sequential registration between completed operations is supported and covered; concurrent registration is not introduced. |

### Upstreamed or dropped

| Patch | Branch / local commit | Landed upstream as | Status | Reapply guidance |
|---|---|---|---|---|
| JavaScript TypeMeta preamble ("fix(javascript): align TypeMeta preamble with python/java/rust/go xlang bindings") | `fix/xlang-typemeta-num-hash-bits` / `df3f8dbbda65d559f5b9bf3bbb6f770655059442` | [PR #3603](https://github.com/apache/fory/pull/3603), merged 2026-04-22 | upstreamed | Do not reapply. Later [PR #3647](https://github.com/apache/fory/pull/3647) metadata changes supersede the original form. |

## Porting to a new upstream release

- Keep the canonical logical fix as one commit on its own topic branch. Port it with `cherry-pick -x` and keep the source branch.
- Keep the packaging change in its own commit, separate from the logical fix; this ledger lives on `main`. Aster does not submit the fix upstream; the README invites upstream maintainers to take it.
- Never rewrite published history or tags.
- Compare against the upstream target, and check semantic equivalence when patch IDs differ (upstream may absorb the fix in a different form).
- Delivery: compare the entire existing Forgejo file inventory and hashes before publishing. Never overwrite different bytes at an existing version, never publish this fork to nuget.org, and never treat skip-duplicate as integrity evidence. The Forgejo-extracted nuspec is part of the inventory, alongside the uploaded nupkg. Deliver the tested archive as-is: rebuilding can change package bytes at the same version.

Steps for a new upstream release `v<ver>` (remote `apache` is upstream, `origin` the Forge):

```sh
git fetch apache --tags
git push origin v<ver>                                  # mirror the apache release tag to the Forge
git switch -c upgrade/fory-v<ver> v<ver>
git cherry-pick -x 3e7593710   # FORY-CS-001 (the 1.7.7 copy; 1.7.6 was e8109b29), unless upstream absorbed it
# packaging commit, separate from the fix: package version <ver>-aster.1 and the repository URL only
git commit -am "build(csharp): prepare Apache.Fory <ver>-aster.1 package"
git push origin upgrade/fory-v<ver>
```

1. Validate (commands below), from `csharp/` on the packaging commit.
2. Pack from the packaging commit, checked out on `upgrade/fory-v<ver>` itself, with `-p:RepositoryCommit=<packaging commit>` (pack command below). The nuspec records the checked-out branch: a pack from a trial branch says `refs/heads/trial/…`, so pack from the upgrade branch or pass `-p:RepositoryBranch=refs/heads/upgrade/fory-v<ver>`. A repack changes only the nuspec and the `.psmdcp` id; compare the DLLs and analyzers with the tested pack.
3. Run the Aster consumer trial against that exact archive (a local directory feed mapped exclusively to `Apache.Fory`, as in the 1.7.6 pre-publication check).
4. Tag the packaging commit, only after validation and the consumer trial pass: `git tag -a aster-fory-v<ver>-aster.1 -m "Aster Apache.Fory <ver>-aster.1" <packaging commit>` and `git push origin aster-fory-v<ver>-aster.1`.
5. Upload the tested archive, as-is, to the Forgejo NuGet feed `https://forge.emrul.dev/api/packages/Aster/nuget/index.json` (never nuget.org), with a package credential passed through an environment variable and a temporary NuGet configuration (never committed): `dotnet nuget push ../artifacts/nuget/Apache.Fory.<ver>-aster.1.nupkg --source https://forge.emrul.dev/api/packages/Aster/nuget/index.json --api-key "$FORGE_NUGET_TOKEN"`. The push uploads the `.snupkg` next to the `.nupkg` too (1.7.7 has one; 1.7.6 did not). Then verify the Forgejo file inventory, anonymously, against the local files' SHA-256 (Delivery, above): `curl -s https://forge.emrul.dev/api/v1/packages/Aster/nuget/Apache.Fory/<ver>-aster.1/files` lists each file's `sha256` (nupkg, extracted nuspec, snupkg, extracted pdb).
6. Aster bumps its `Apache.Fory` pin (`bindings/dotnet/src/Aster/Aster.csproj`, `Version="[<ver>-aster.1]"`) and runs its .NET gate: build the FFI library first (`cargo build -p aster_transport_ffi`), then `dotnet build`/`dotnet test bindings/dotnet/Aster.sln` with `-p:NativeLibraryPath=<that target dir>` and `ASTER_TEST_PYTHON` set to a Python with the raw WebTransport peer's test requirements.
7. Update this ledger on `main` (merge upstream `main` first).

A further Aster-only fix on the same upstream base is a topic branch cherry-picked `-x` onto the current upgrade branch, followed by a packaging commit at `<ver>-aster.<n+1>`; never republish different bytes at an existing version.

Validation commands (C# commands run from `csharp/`):

```sh
ENABLE_FORY_DEBUG_OUTPUT=1 dotnet test Fory.sln -c Release
dotnet build Fory.sln -c Release --no-restore
dotnet format Fory.sln --verify-no-changes
# From java/, with JDK 21 selected:
mvn -T16 install -DskipTests
# From java/fory-core/:
FORY_CSHARP_JAVA_CI=1 ENABLE_FORY_DEBUG_OUTPUT=1 mvn -T16 test -Dtest=org.apache.fory.xlang.CSharpXlangTest
```

Upstream's current validation rules require the C# peer suite for this scope, not unrelated runtime peers.

Pack command, from the packaging commit's `csharp/` directory on the upgrade branch (1.7.7: `7a299ebce2d5c7b966ef7857ae1ba1af182f01a4`; 1.7.6: `fdeb5cd01baa208d128fff7d0c06e3b3e30481e5`):

```sh
dotnet pack src/Fory/Fory.csproj -c Release \
  -p:RepositoryCommit=<packaging commit> \
  -p:RepositoryBranch=refs/heads/upgrade/fory-v<ver> \
  -p:ContinuousIntegrationBuild=true -o ../artifacts/nuget
```

Performance harness (run each baseline/fixed side separately, never concurrently):

```sh
dotnet <harness-baseline-or-fixed>/Harness.dll warm-single false <evidence-directory>
dotnet <harness-baseline-or-fixed>/Harness.dll warm-single true <evidence-directory>
dotnet <harness-baseline-or-fixed>/Harness.dll cold false <evidence-directory>
dotnet <harness-baseline-or-fixed>/Harness.dll cold true <evidence-directory>
```

Remaining work:

- Absorb an upstream release once one contains the fix (Aster does not submit upstream; the README invites upstream maintainers to take it).
- The full .NET release gate against the next exact Aster release candidate (adoption is done: the .NET binding pins `Apache.Fory [1.7.7-aster.1]`, and its suite passes on that pack).
- Higher-precision cold-start performance evidence if a strict paired sub-1% claim is needed; the bounded results below preserve the uncertainty.

## History

### 2026-10-21: `Apache.Fory 1.7.7-aster.1` on upstream v1.7.7

- New integration branch `upgrade/fory-v1.7.7`, cut from `v1.7.7` (`04cbe11a`): FORY-CS-001 `3e7593710` (clean `cherry-pick -x` of `e8109b29`, itself of `345e4c77`), then packaging commit `7a299ebc` (package version `1.7.7-aster.1`, repository URL). Tag `aster-fory-v1.7.7-aster.1` (annotated) → `7a299ebc`. Upstream 1.7.7 did not absorb the fix: its six regression tests fail on bare `v1.7.7`.
- Validation: 383 C# tests pass; Java-driven `CSharpXlangTest` 107 pass; Aster's .NET suite 592 passed / 6 skipped on the pack. Aster's Rust, Java, Python, TypeScript and Go bindings moved to upstream 1.7.7 in the same change.
- Publication on the Forge feed (never nuget.org), with a temporary NuGet configuration; the Forgejo file inventory matches the local files:

| File | SHA-256 |
|---|---|
| `apache.fory.1.7.7-aster.1.nupkg` | `d855bd0e31a819bfd4a8b886cacec1c508888f2a1d0a9101432dc74ebf2db9ff` |
| `apache.fory.nuspec` (extracted by Forgejo) | `518f1f516ac45758e3fb8548ff99f5ec55e47124a10a962b34d27845e28f2fb3` |
| `apache.fory.1.7.7-aster.1.snupkg` | `7e497a437eae9adbf45fe708f5060d3f53d301f87ea0b25717ebf7970f8f45c4` |
| `fory.pdb` (extracted from the snupkg by Forgejo) | `bdf97ed91ccbfd5dc96e88d209733a1fe3278b8959fb0524b09cc7273597408e` |

- The published archive is a repack from `upgrade/fory-v1.7.7`, because the tested trial pack recorded `refs/heads/trial/fory-v1.7.7` as its nuspec branch. Its payload is byte-identical to the tested pack (`analyzers/dotnet/cs/Fory.Generator.dll` `c4edd0e45ad67946d51c65adaa7b4a3effc3c85ba595659667ebc11045230b0d`, `lib/net8.0/Fory.dll` `46916b0ba64c49982aa7924415462334be4845b38d6bf6a3b905c930563dec50`); only the nuspec branch and the `.psmdcp` id differ. The nuspec names `refs/heads/upgrade/fory-v1.7.7` at `7a299ebc`. This release also ships the symbol package, which 1.7.6 did not.


### 2026-10-08: FORY-CS-001 and `Apache.Fory 1.7.6-aster.1`

**Validation (source).** macOS ARM64 (exact .NET SDK/runtime versions retained with the evidence). Baseline full C# suite: 377 passed, no failures or skips; Release build zero warnings/errors. Six new tests fail on the baseline with `Count=0, Message=""`: first-use publication, independent codecs and schema replacement, each with reference tracking disabled and enabled. With the fix the six pass and the suite is 383 passed, zero failures/skips. Java-driven `org.apache.fory.xlang.CSharpXlangTest`: 107 passed, zero failures/skips. Release build and `dotnet format Fory.sln --verify-no-changes` pass. The 1.7.6 source baseline independently fails all six regressions; the backport passes all 383 C# tests, builds with zero warnings/errors and passes formatting, and its own Java prerequisite build and `CSharpXlangTest` pass all 107 tests with zero failures/errors/skips (the mainline result is not substituted for this run). Linux x64 (SDK 9.0.318, runtime 9.0.20): all four controlled first-use/independent-codec cases fail on the original, while the two unscheduled replacement cases pass in that run; the fixed suite passes all 383. These platform differences reflect the deliberately scheduled versus unscheduled cases; no first-attempt failures were discarded.

The controlled tests run real annotated models through the generator, then add only a scheduling call to the emitted source. Baseline first-use instrumentation pauses between the identity and Boolean stores; corrected instrumentation pauses after complete snapshot publication, outside locks. Both callers' complete values and bounded completion are asserted. Original and instrumented emitted sources and first-attempt failures are retained in the validation evidence.

The exact candidate package passes all six regressions on macOS and Linux, using its own analyzer and runtime from a fresh isolated NuGet cache. Each platform also passes three fresh independent-codec processes per tracking mode, 2 million complete-value-checked decodes per process.

This work proves a codec publication defect and its correction. It does not establish which interleaving caused Aster's original intermittent CI failure, and does not resolve the separate cancellation-test investigation.

**Bounded performance comparison.** Same annotated `Count=37, Message="correct"` model, persisted valid payloads, 4 worker threads and complete-value assertions on both revisions; each warmed process performs 2 million decodes after sequential worker warm-up; seven adjacent baseline/fixed pairs per case:

| Warm single codec | Baseline ns/decode | Corrected ns/decode | Median paired delta | Allocations |
|---|---|---|---|---|
| No reference tracking | 80.030 | 79.912 | -0.580% | 144 B/decode on both |
| Reference tracking | 79.634 | 78.845 | -0.810% | 144 B/decode on both |

- The first correction measured a 1.255% paired no-reference slowdown over 12 pairs. The final emitter marks the small matching helper for inlining; snapshot construction stays no-inline. Final measurements are within the repository's 1% threshold; this is not a claim of a significant speedup. JIT disassembly shows the initial and final corrected Tier1 bodies both inline matching and are 1676 bytes; a causal speedup from the annotation is not established.
- Independent differently configured codecs corrupted values on the unmodified baseline in both tracking modes, so there is no valid baseline throughput for that case. Corrected runs passed three fresh processes per mode. Initial corrected medians: 117.215 ns/decode and 183.210 B/decode without refs; 123.197 ns/decode and 184.096 B/decode with refs. A version replacement allocates a 40-byte snapshot; interleaved configurations can replace the single cache slot on every read. Warm single-configuration reads allocate no cache entries.
- The exact 1.7.6 candidate measured against its own 1.7.6 source baseline, five adjacent pairs per warm case: no-ref medians 84.107/80.703 ns per decode (median paired -3.900%); ref 90.403/82.733 ns (median paired -7.605%). Warm single-codec allocations remain 144 B/decode.
- Cold first use includes runtime JIT and lazy initialisation. No-ref (five pairs) medians 12.157/12.000 ms, paired -0.814%; ref (15 pairs) medians 12.246/12.352 ms, paired +1.480% (median-to-median +0.869%). Dispersion and differing summary results leave a small cold-start regression uncertain: do not claim a proven cold-start speedup or a confidently sub-1% paired result. All samples are retained; no further sampling was used to select a passing result. First-codec allocations rise by 16 bytes (13216 → 13232 without refs; 13272 → 13288 with refs), replacing the old lock object with a 40-byte immutable snapshot. A second resolver configuration adds 40 bytes (8576 → 8616 without refs; 8632 → 8672 with refs). Second-codec cold timing medians were 66.792/61.250 µs without refs and 65.166/69.875 µs with refs; they include lazy runtime work and are not a steady-state comparison. These cold allocation and timing costs are recorded as trade-offs of the correctness fix.

**Package provenance.** The upstream NuGet package's `.nuspec` declares repository commit `92483805f93fca4c20a3837f2380b5ee16ef58e7`; upstream `v1.7.6` and `v1.7.6-rc1` resolve to that exact commit, and its build props declare version 1.7.6.

| Baseline artifact | SHA-256 |
|---|---|
| `Apache.Fory.1.7.6.nupkg` | `fe9bd97e8f8f6797c2fc3ce43dd13050a3fd1b5487ca37c1c3800ef5bf450a5c` |
| `Fory.Generator.dll` | `41eafe168a65ab3ce246bc28a29292aa8e18b6327a4544735852d2f024bd4d2d` |
| `lib/net8.0/Fory.dll` | `4f1ba6a5e1c8d11138ac4312247193f6b76a7532695a243c8de2365ce22bdde0` |

These verify the retained package identity and its declared source; source builds are not claimed to be byte-identical to the published binaries. The locally built package `Apache.Fory.1.7.6-aster.1.nupkg` has SHA-256 `ebd3973298fbefcfdebc223827c53785125b58d309327d0dad1a2e74ba75b341`. It contains the net8.0 runtime, the netstandard generator under analyzer assets, Apache-2.0 license metadata and the README, and has no package dependencies. Its repository URL, delivery branch and commit match the packaging source.

| Candidate payload | SHA-256 |
|---|---|
| `analyzers/dotnet/cs/Fory.Generator.dll` | `c90658ad237ee81f3fbaa85090162ed95c9c309d99ebef50157589a7e649ad1f` |
| `lib/net8.0/Fory.dll` | `ddf999c23d785372977f92b018987ef3f00edfcc3b292244e593099c2a07b1bd` |

Pre-publication local consumption: the test source mapped `Apache.Fory` exclusively to a directory feed holding this exact archive, with nuget.org for unrelated test dependencies, and used `Version="[1.7.6-aster.1]"`. No checked-in generated serializer is compiled: the package analyzer generates the ordinary harness model, and the controlled tests load the same packaged generator to emit their annotated models. The ordinary generated source SHA-256 is identical on both platforms: `c8721c84ca0c7dce0b42b9140e76f4aa0938c642a458790474b28fc91be6c788`. Fresh-cache consumer validation passed on macOS and Linux.

**Publication.** Explicitly authorised on 2026-10-08. The original tested archive was uploaded without rebuilding; anonymous download reproduces SHA-256 `ebd3973298fbefcfdebc223827c53785125b58d309327d0dad1a2e74ba75b341`. The complete Forgejo file inventory:

| File | SHA-256 |
|---|---|
| `apache.fory.1.7.6-aster.1.nupkg` | `ebd3973298fbefcfdebc223827c53785125b58d309327d0dad1a2e74ba75b341` |
| `apache.fory.nuspec` (extracted by Forgejo) | `5e43aa332cde9bafad970df20a0d54a9f257f8b429716bb1f0482ff661026978` |

The extracted nuspec hash matches the entry inside the tested archive. The symbol package was not uploaded. An `agents` package credential was passed through an environment variable and a temporary NuGet configuration and removed after upload; no credential is committed or included in the evidence.

Post-publication anonymous fresh-cache consumption passed on macOS ARM64: `Apache.Fory` at exact `[1.7.6-aster.1]` resolved from Forgejo; archive, runtime, analyzer and generated-source SHA-256 hashes all match the tested candidate. All six regression tests passed, followed by 2 million complete-value-checked parallel independent-codec decodes per tracking mode. The source mapping, resolved assets, commands and logs are retained with the evidence (`PUBLISHED_VALIDATION.txt` and the `forgejo-consumer` evidence directory). No Aster dependency was changed at that point.

**Evidence.** Aster maintainers retain (not published) the original and instrumented generated source, baseline/fixed source diffs, all first-attempt failure logs, TRX reports, xlang reports, paired benchmark JSON, JIT disassembly, consumer source/configuration/assets, package inventories, and the local package archive.

### 2026-04-22: JavaScript preamble upstreamed

The JavaScript TypeMeta preamble patch (`df3f8dbb`) was merged upstream as PR #3603. See "Upstreamed or dropped".
