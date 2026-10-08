# Aster Fory fork patch ledger

## Current pins

The C# metadata publication correction is implemented. Nothing in this wave
is published or adopted by Aster. The canonical fix and the package backport are
separate from local main, which remains unchanged.

| Repo/package             | Upstream base tag and full SHA                    | Fork tag/version                | Pinned fork SHA                            | Branch                                   |
| ------------------------ | ------------------------------------------------- | ------------------------------- | ------------------------------------------ | ---------------------------------------- |
| Fory upstream correction | main `261dc92eabdc5c0fe08748a3ba476ae13f4e8bc9`   | unreleased                      | `345e4c771ccb4a895181fe8cb244ef2b27fb0196` | `fix/csharp-metadata-publication`        |
| Apache.Fory backport     | v1.7.6 `92483805f93fca4c20a3837f2380b5ee16ef58e7` | local candidate `1.7.6-aster.1` | `fdeb5cd01baa208d128fff7d0c06e3b3e30481e5` | `delivery/csharp-1.7.6-aster.1`          |
| Aster consumer           | Apache.Fory 1.7.6, same source as above           | upstream package                | no fork pin adopted                        | `bindings/dotnet/src/Aster/Aster.csproj` |

## Current patch set

| Stable patch ID | Canonical branch                  | Commit subject                                           | Why it is needed                                                                                                             | Upstream issue/PR      | Status                                                        |
| --------------- | --------------------------------- | -------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------- | ---------------------- | ------------------------------------------------------------- |
| FORY-CS-001     | `fix/csharp-metadata-publication` | fix(csharp): publish consistent metadata cache snapshots | Shared metadata identity/match writes and resolver hash replacement can return default field values during parallel decoding | pending, not submitted | validated locally on macOS and Linux; packaged, not published |

## Patch sources and stack

| Canonical branch                  | Exact base                                 | Patch commit                               | Backport commit/source                                                                                           | Dependencies and reapply order                                              |
| --------------------------------- | ------------------------------------------ | ------------------------------------------ | ---------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------- |
| `fix/csharp-metadata-publication` | `261dc92eabdc5c0fe08748a3ba476ae13f4e8bc9` | `345e4c771ccb4a895181fe8cb244ef2b27fb0196` | `e8109b29604aa409d0b65313d7d52eca9c4415fe` on `92483805f93fca4c20a3837f2380b5ee16ef58e7`, clean `cherry-pick -x` | no prerequisite patches; fix first, separate packaging changes, then ledger |

Keep the canonical logical fix as one commit on its own branch. Backport with
`cherry-pick -x`; retain that source branch. Keep packaging and this ledger off
the upstream contribution. Never rewrite published history or tags. For future
upgrades, compare against the upstream target, and check semantic equivalence
when patch IDs differ.

## FORY-CS-001

Affected files:

- `csharp/src/Fory.Generator/ForyModelGenerator.Emission.cs`
- `csharp/tests/Fory.Tests/ForyGeneratorTests.cs`
- `csharp/tests/Fory.Tests/MetadataPublicationTests.cs`

The generator publishes an immutable snapshot of the resolver version and both
local metadata header hashes with acquire/release semantics. Each invocation
retains its snapshot even if a different codec publishes another one. A direct
header comparison replaces the redundant remote-identity/Boolean memoization;
there is no separately published match flag. Wire bytes, public APIs, compatible
field dispatch and parallel root decoding remain unchanged.

`VersionHash()` is a semantic configuration fingerprint, not a resolver-local
counter, on both selected baselines. Equal counters with differing configurations
are therefore not a supported case to synthesize. Independent codec tests use
different registration names, hence different metadata hashes and resolver
versions. Sequential registration between completed operations is supported and
covered; concurrent registration is not introduced.

Conflict notes: the metadata emitter and runtime sources are identical between
the two selected baselines. Future changes to `VersionHash`, local metadata hash
ownership, generated compatible reads or hierarchy providers need re-review.
Never carry a cached Boolean separately from its metadata identity, nor read
mutable shared hashes after checking only a shared resolver version.

### Validation

macOS ARM64; exact installed .NET SDK/runtime versions are retained in the evidence.
Baseline full C# suite: 377 passed, no failures/skips; Release build has zero
warnings/errors. Six new baseline tests fail with `Count=0, Message=""`:
first-use publication, independent codecs and schema replacement, each with
reference tracking disabled and enabled. The corrected six cases pass.
Corrected suite: 383 passed, zero failures/skips. Java-driven
`org.apache.fory.xlang.CSharpXlangTest`: 107 passed, zero failures/skips.
Release build and `dotnet format Fory.sln --verify-no-changes` pass.
The verified 1.7.6 source baseline independently fails all six regressions;
the backport passes all 383 C# tests, builds with zero warnings/errors and
passes formatting. Linux x64 with SDK 9.0.318/runtime 9.0.20: all four controlled original
first-use/independent-codec cases fail, while the two unscheduled replacement
cases pass in that run. The fixed source suite passes all 383 tests. These
platform differences reflect the deliberately scheduled versus unscheduled
cases; no first-attempt failures were discarded.

The exact candidate package passes all six regressions on macOS and Linux,
using its own analyzer and runtime from a fresh isolated NuGet cache. Each
platform also passes three fresh independent-codec processes per tracking mode,
2 million complete-value-checked decodes per process.

Commands (C# commands run from `csharp/`):

```sh
ENABLE_FORY_DEBUG_OUTPUT=1 dotnet test Fory.sln -c Release
dotnet build Fory.sln -c Release --no-restore
dotnet format Fory.sln --verify-no-changes
# From java/, with JDK 21 selected:
mvn -T16 install -DskipTests
# From java/fory-core/:
FORY_CSHARP_JAVA_CI=1 ENABLE_FORY_DEBUG_OUTPUT=1 mvn -T16 test -Dtest=org.apache.fory.xlang.CSharpXlangTest
```

This is a C#-local cache correction: shared protocol, type mappings and wire
semantics are unchanged. The current upstream validation rules require the C#
peer suite, not unrelated runtime peers, for this scope.

### Bounded performance comparison

The same annotated `Count=37, Message="correct"` model, persisted valid payloads,
4 worker threads and complete-value assertions are used on both revisions.
Each warmed process performs 2 million decodes after sequential worker warmup.
Seven adjacent baseline/fixed pairs per case give these retained medians:

| Warm single codec     | Baseline ns/decode | Corrected ns/decode | Median paired delta | Allocations          |
| --------------------- | ------------------ | ------------------- | ------------------- | -------------------- |
| No reference tracking | 80.030             | 79.912              | -0.580%             | 144 B/decode on both |
| Reference tracking    | 79.634             | 78.845              | -0.810%             | 144 B/decode on both |

The first correction measured a 1.255% paired no-reference slowdown over 12
pairs. The final emitter explicitly marks the small matching helper for inlining;
snapshot construction stays no-inline. Final measurements are within the 1%
repository threshold, not a claim of a significant speedup. JIT disassembly
shows the initial and final corrected Tier1 bodies both inline matching and
have 1676 bytes; a causal speedup from the annotation is not established.

Independent differently configured codecs corrupted values on the unmodified
baseline in both tracking modes, so there is no valid baseline throughput
number for that case. Corrected runs passed three fresh processes per mode.
Initial corrected medians: 117.215 ns/decode and 183.210 B/decode without refs;
123.197 ns/decode and 184.096 B/decode with refs. A version replacement allocates
a 40-byte snapshot; interleaved configurations can replace the single cache
slot on every read. Warm single-configuration reads allocate no cache entries.
The exact 1.7.6 candidate was also measured separately against its original
1.7.6 source baseline, five adjacent pairs per warm case: no-ref medians
84.107/80.703 ns per decode (median paired -3.900%); ref 90.403/82.733 ns
(median paired -7.605%). Warm single-codec allocations remain 144 B/decode.

Cold first-use includes runtime JIT and lazy initialization. No-ref (five pairs)
medians were 12.157/12.000 ms, paired -0.814%; ref (15 pairs) medians were
12.246/12.352 ms, paired +1.480%. The latter median-to-median difference is
+0.869%; dispersion and differing summary results leave a small cold-start
regression uncertain. Do not claim a proven cold-start speedup or a confidently
sub-1% paired result. All samples are retained; no further sampling was used to
select a passing result. First-codec allocations rise by 16 bytes (13216 to
13232 without refs; 13272 to 13288 with refs), replacing the old lock object
with a 40-byte immutable snapshot. A second resolver configuration adds 40 bytes
(8576 to 8616 without refs; 8632 to 8672 with refs). These bounded cold costs
are part of the correctness fix.

Harness invocation (run each baseline/fixed side separately, never concurrently):

```sh
dotnet <harness-baseline-or-fixed>/Harness.dll warm-single false <evidence-directory>
dotnet <harness-baseline-or-fixed>/Harness.dll warm-single true <evidence-directory>
dotnet <harness-baseline-or-fixed>/Harness.dll cold false <evidence-directory>
dotnet <harness-baseline-or-fixed>/Harness.dll cold true <evidence-directory>
```

The controlled tests run real annotated models through the generator, then add
only a scheduling call to emitted source. Baseline first-use instrumentation
pauses between identity and Boolean stores. Corrected instrumentation pauses
after complete snapshot publication, outside locks. Both callers' complete
values and bounded completion are asserted. Original and instrumented emitted
sources and first-attempt failures are retained in the validation evidence.

### Package provenance

The installed NuGet package's `.nuspec` declares repository commit
`92483805f93fca4c20a3837f2380b5ee16ef58e7`; upstream `v1.7.6` and `v1.7.6-rc1`
resolve to that exact commit, and its build props declare version 1.7.6.

| Baseline artifact       | SHA-256                                                            |
| ----------------------- | ------------------------------------------------------------------ |
| Apache.Fory.1.7.6.nupkg | `fe9bd97e8f8f6797c2fc3ce43dd13050a3fd1b5487ca37c1c3800ef5bf450a5c` |
| Fory.Generator.dll      | `41eafe168a65ab3ce246bc28a29292aa8e18b6327a4544735852d2f024bd4d2d` |
| lib/net8.0/Fory.dll     | `4f1ba6a5e1c8d11138ac4312247193f6b76a7532695a243c8de2365ce22bdde0` |

These verify the retained package identity and its declared source provenance;
source builds are not claimed to be byte-identical to the published binaries.
The packaging commit is `fdeb5cd01baa208d128fff7d0c06e3b3e30481e5`, on `delivery/csharp-1.7.6-aster.1`.
It changes only package version and repository URL, separately from the logical
fix. Canonical backport branch `backport/csharp-metadata-publication-1.7.6`
remains at `e8109b29604aa409d0b65313d7d52eca9c4415fe`. The exact locally built package is `Apache.Fory.1.7.6-aster.1.nupkg`,
SHA-256 `ebd3973298fbefcfdebc223827c53785125b58d309327d0dad1a2e74ba75b341`.
The archive contains the net8.0 runtime, netstandard generator under analyzer
assets, Apache-2.0 license metadata, README, and no package dependencies.
Its repository URL, delivery branch and commit match the packaging source above.

| Candidate payload                        | SHA-256                                                            |
| ---------------------------------------- | ------------------------------------------------------------------ |
| `analyzers/dotnet/cs/Fory.Generator.dll` | `c90658ad237ee81f3fbaa85090162ed95c9c309d99ebef50157589a7e649ad1f` |
| `lib/net8.0/Fory.dll`                    | `ddf999c23d785372977f92b018987ef3f00edfcc3b292244e593099c2a07b1bd` |

Pack command, from the backport's `csharp/` directory:

```sh
dotnet pack src/Fory/Fory.csproj -c Release \
  -p:RepositoryCommit=fdeb5cd01baa208d128fff7d0c06e3b3e30481e5 \
  -p:ContinuousIntegrationBuild=true -o ../artifacts/nuget
```

The tested archive must be delivered as-is; rebuilding can change package bytes
at the same proposed version. Fresh-cache consumer validation passed on macOS and Linux as detailed above.
The local test source maps `Apache.Fory` exclusively to a directory feed holding
this exact archive, with nuget.org for unrelated test dependencies, and uses
`Version="[1.7.6-aster.1]"`. No checked-in generated serializer is compiled:
the package analyzer generates the ordinary harness model, and the controlled
tests load that same packaged generator to emit their annotated models.
The ordinary generated source SHA-256 is identical on both platforms:
`c8721c84ca0c7dce0b42b9140e76f4aa0938c642a458790474b28fc91be6c788`.
This proves local artifact consumption, not Forgejo delivery.

## Delivery and consumer adoption

Target feed: `https://forge.emrul.dev/api/packages/Aster/nuget/index.json`.
Package ID stays `Apache.Fory`. Proposed version `1.7.6-aster.1` was absent
(anonymous exact-package HTTP 404) during preparation on 2026-10-08; publication
must recheck immutability. A local package is **packaged, not published**.
Publication and Aster adoption require their separate follow-up authorization.

The package must contain the corrected runtime and analyzer, then be consumed
with analyzers enabled from an isolated NuGet cache. Before publication, compare
any existing complete file inventory and hashes; never overwrite a different
artifact at an existing version, publish to nuget.org, or use skip-duplicate as
integrity evidence. Use the authorized `ASTER_PACKAGE_TOKEN` environment with
the `agents` account through temporary NuGet configuration, removed afterward.
Record remotely retrieved package hashes and anonymous consumption separately
from local validation. Do not record credentials here.

The following delivery configuration is prepared for the eventual publication;
remote candidate restore cannot be validated until that publication exists.
Use it in CI and downstream consumers explicitly, not a developer's global
configuration:

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

After publication, restore with this config into a fresh `NUGET_PACKAGES`
directory and verify the resolved archive hash before running consumer gates.
Use an exact NuGet version constraint, `Version="[1.7.6-aster.1]"`.

Aster must pin the exact replacement version in
`bindings/dotnet/src/Aster/Aster.csproj`, map `Apache.Fory` to the Forgejo source
explicitly (nuget.org remains available for unrelated dependencies), and run its
concurrent RPC integrity test plus normal .NET release gate against a fresh
exact candidate. Its dependency and NuGet provenance records must agree with
this ledger; the Rust BOM is not NuGet provenance. These adoption steps remain
pending and do not follow automatically from Fory unit tests.

## Graduated patches

| Patch               | Local commit                               | Upstream absorption                                                     | Reapply guidance                                                                                                         |
| ------------------- | ------------------------------------------ | ----------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| JavaScript preamble | `df3f8dbbda65d559f5b9bf3bbb6f770655059442` | [PR #3603](https://github.com/apache/fory/pull/3603), merged 2026-04-22 | Do not reapply. Later [PR #3647](https://github.com/apache/fory/pull/3647) metadata changes supersede the original form. |

## Hand-back evidence and remaining work

Local evidence is retained at `/Users/emrul/dev/aster/fory-csharp-evidence`:
original and instrumented generated source, baseline/fixed source diffs, all
first-attempt failure logs, TRX reports, xlang reports, paired benchmark JSON,
JIT disassembly, consumer source/configuration/assets and package inventories.
The local package is at
`/Users/emrul/dev/aster/fory-csharp-1.7.6/artifacts/nuget/Apache.Fory.1.7.6-aster.1.nupkg`.

The recommendation is the narrowly patched 1.7.6 fork package above, subject to
separately authorized publication and Aster qualification. Pending work:

- Upstream issue/PR submission and upstream release absorption.
- Source publication and NuGet publication to Forgejo, immutable remote inventory
  verification, anonymous fresh-cache restore and resolved source verification.
- Aster dependency/provenance adoption, concurrent RPC integrity test and full
  .NET release gate against the fresh exact release candidate.
- Higher-precision cold-start performance evidence if a strict paired sub-1%
  claim is needed; the bounded initial results above preserve the uncertainty.

This work proves a codec publication defect and its correction. It does not
establish which interleaving caused Aster's original intermittent CI failure,
and does not resolve the separate cancellation-test investigation.
