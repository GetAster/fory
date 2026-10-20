# Aster patch ledger: fory

This repository is Aster's fork of [apache/fory](https://github.com/apache/fory). Aster carries the patches listed below on top of upstream releases. Upstream maintainers are welcome to incorporate any of them, in whole or in part, at their discretion.

Aster consumes Fory's Rust crates (`fory-core`, `fory-derive`, version 1.7.6) unmodified from crates.io; they are not mirrored or republished. The patch carried here is in the C# implementation, delivered as an Aster-built `Apache.Fory` NuGet package.

## Branch and tag model

- `main` is the front door: upstream's README with an Aster preamble, plus this ledger. Nothing is built from `main`.
- `upgrade/<version>` is the patched integration line for one upstream release wave: the upstream release plus the patches in this ledger. Consumers pin it by commit.
- Topic branches carry one change each and are named for what the change is, for example `feat/read-into` or `security/plumtree-bounds`. Each is based on the upstream release tag and is cherry-picked with `git cherry-pick -x` onto the upgrade branch, so every upgrade-branch commit names its source.
- Release tags mark the commits that are released and pinned. Published tags are never moved or rewritten. Historical tags in this repository use several schemes (listed below); new tags use `aster-<repo>-v<version>`, where `<version>` is the published version of the repository's main package (for example `aster-salvo-v1.0.4`, `aster-fory-v1.7.6-aster.1`).
- `main` is refreshed by merging upstream `main` into it. It is not force-pushed.

Tag schemes used in this repository:

- `v<upstream version>-aster.<n>`: `v1.7.6-aster.1` (annotated; names the exact source of the `Apache.Fory 1.7.6-aster.1` package).

## Current release

| Field | Value |
|---|---|
| Upstream base | `v1.7.6` (`92483805f93fca4c20a3837f2380b5ee16ef58e7`) |
| Integration branch | `upgrade/fory-v1.7.6`, at the tip of `delivery/csharp-1.7.6-aster.1` |
| Tag | `v1.7.6-aster.1` → packaging commit `fdeb5cd01baa208d128fff7d0c06e3b3e30481e5` |
| Package | NuGet `Apache.Fory` `1.7.6-aster.1` |
| Package feed | `https://forge.emrul.dev/api/packages/Aster/nuget/index.json` |
| Source repository | `https://forge.emrul.dev/Aster/fory` |
| Aster consumer | `bindings/dotnet/src/Aster/Aster.csproj` in the Aster repository |

The integration line is: upstream `v1.7.6`, then the backport of FORY-CS-001 (`e8109b29604aa409d0b65313d7d52eca9c4415fe`, a clean `cherry-pick -x` of `345e4c771ccb4a895181fe8cb244ef2b27fb0196`), then the packaging commit `fdeb5cd0…` (package version and repository URL only, kept separate from the logical fix), then ledger-only commits. Ledger commits after the tag do not change the immutable package or its provenance.

Branches (all verified anonymously on Forgejo, with no Git credential helper, where a commit is given):

| Branch | Role | Commit |
|---|---|---|
| `fix/csharp-metadata-publication` | Canonical topic branch for FORY-CS-001, on upstream `main` `261dc92eabdc5c0fe08748a3ba476ae13f4e8bc9` | `345e4c771ccb4a895181fe8cb244ef2b27fb0196` |
| `backport/csharp-metadata-publication-1.7.6` | Canonical backport onto `v1.7.6` | `e8109b29604aa409d0b65313d7d52eca9c4415fe` |
| `delivery/csharp-1.7.6-aster.1` | Backport + packaging commit + this ledger; source of the published package | tag `v1.7.6-aster.1` at `fdeb5cd01baa208d128fff7d0c06e3b3e30481e5` |
| `delivery/csharp-metadata-publication` | The mainline fix plus this ledger | — |
| `upgrade/fory-v1.7.6` | Integration branch under the current model; same tip as `delivery/csharp-1.7.6-aster.1` | — |

Aster has since adopted it: the Aster .NET binding pins `Apache.Fory [1.7.6-aster.1]`.

Consumer status at the time of delivery (2026-10-08): the package and its source were published, but Aster's dependency adoption and release qualification were pending (Aster then used upstream `Apache.Fory 1.7.6`, with no fork pin). Adoption requires Aster to:

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
| FORY-CS-001 | `fix/csharp-metadata-publication` (based on upstream `main` `261dc92e…`) | fix(csharp): publish consistent metadata cache snapshots. The generator publishes an immutable snapshot of the resolver version and both local metadata header hashes with acquire/release semantics. Each invocation keeps its snapshot even if a different codec publishes another. A direct header comparison replaces the redundant remote-identity/Boolean memoisation; there is no separately published match flag. Wire bytes, public APIs, compatible field dispatch and parallel root decoding are unchanged. Files: `csharp/src/Fory.Generator/ForyModelGenerator.Emission.cs`, `csharp/tests/Fory.Tests/ForyGeneratorTests.cs`, `csharp/tests/Fory.Tests/MetadataPublicationTests.cs`. | Shared metadata identity/match writes and resolver hash replacement could return default field values during parallel decoding. Aster's .NET binding (`Apache.Fory` consumer) is the user; its concurrent RPC integrity test is the adoption gate. This is a C#-local cache correction: shared protocol, type mappings and wire semantics are unchanged. | `345e4c771ccb4a895181fe8cb244ef2b27fb0196` / backport `e8109b29604aa409d0b65313d7d52eca9c4415fe` (on `v1.7.6`, clean `cherry-pick -x`) | carried (upstream issue/PR not yet submitted) | No prerequisite patches. Apply the fix first, then the separate packaging change, then the ledger. The metadata emitter and runtime sources are identical between the two baselines (`261dc92e` and `v1.7.6`). Future changes to `VersionHash`, local metadata hash ownership, generated compatible reads or hierarchy providers need re-review. Never carry a cached Boolean separately from its metadata identity, nor read mutable shared hashes after checking only a shared resolver version. `VersionHash()` is a semantic configuration fingerprint, not a resolver-local counter, on both baselines, so equal counters with differing configurations are not a supported case to synthesise. Independent codec tests use different registration names, hence different metadata hashes and resolver versions. Sequential registration between completed operations is supported and covered; concurrent registration is not introduced. |

### Upstreamed or dropped

| Patch | Branch / local commit | Landed upstream as | Status | Reapply guidance |
|---|---|---|---|---|
| JavaScript TypeMeta preamble ("fix(javascript): align TypeMeta preamble with python/java/rust/go xlang bindings") | `fix/xlang-typemeta-num-hash-bits` / `df3f8dbbda65d559f5b9bf3bbb6f770655059442` | [PR #3603](https://github.com/apache/fory/pull/3603), merged 2026-04-22 | upstreamed | Do not reapply. Later [PR #3647](https://github.com/apache/fory/pull/3647) metadata changes supersede the original form. |

## Porting to a new upstream release

- Keep the canonical logical fix as one commit on its own topic branch. Port it with `cherry-pick -x` and keep the source branch.
- Keep packaging changes and this ledger out of the upstream contribution, in separate commits.
- Never rewrite published history or tags.
- Compare against the upstream target, and check semantic equivalence when patch IDs differ (upstream may absorb the fix in a different form).
- Delivery: compare the entire existing Forgejo file inventory and hashes before publishing. Never overwrite different bytes at an existing version, never publish this fork to nuget.org, and never treat skip-duplicate as integrity evidence. The Forgejo-extracted nuspec is part of the inventory, alongside the uploaded nupkg. Deliver the tested archive as-is: rebuilding can change package bytes at the same version.

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

Pack command, from the backport's `csharp/` directory:

```sh
dotnet pack src/Fory/Fory.csproj -c Release \
  -p:RepositoryCommit=fdeb5cd01baa208d128fff7d0c06e3b3e30481e5 \
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

- Submit the upstream issue/PR and absorb the upstream release.
- Aster dependency/provenance adoption, the concurrent RPC integrity test and the full .NET release gate against the fresh exact release candidate.
- Higher-precision cold-start performance evidence if a strict paired sub-1% claim is needed; the bounded results below preserve the uncertainty.

## History

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
