# OrderKit - CS_VNET8_SDKMSBUILD_CPM_MICRO

One branch of the C# tool-evaluation corpus. Test **data**, not a product: tool
execution is not part of the build, and evaluation happens later.

## Branch

`CS_VNET8_SDKMSBUILD_CPM_MICRO`

## Cell

| Axis | Value |
|---|---|
| Bundler | SDK-style MSBuild |
| Package manager | Central Package Management |
| Architecture | Microservices |

Projects:

- `OrderKit.Domain` (`src/OrderKit.Domain`)
- `OrderKit.Contracts` (`src/OrderKit.Contracts`)
- `OrderKit.Pricing` (`src/Services/OrderKit.Pricing`)
- `OrderKit.Risk` (`src/Services/OrderKit.Risk`)
- `OrderKit.Reporting` (`src/Services/OrderKit.Reporting`)

## Target framework

`net8.0` - .NET 8

The first family in this corpus that is STILL IN SUPPORT (LTS, to 10 November 2026), and the first whose target framework MATCHES the pinned SDK -- every earlier family was built by an SDK newer than its own target. packages.config remains dead, so six of the 24 cells stay impossible.

## Language version

C# 12.0, taken as the default of the target framework. `LangVersion` is deliberately
never written to any project file, which is what keeps the family axis one number
instead of two.

The domain layer is written to that language subset exactly, and the lock is
verified by execution rather than asserted:

```
csc -langversion:12.0   ->  clean
csc -langversion:11.0   ->  error CS8025: Feature 'async function' is not available
```

## Supported tools

6 of the 19 tools in the roster are expected to fire on this branch. The rest exit
3 (cannot run here) or 4 (absent from this host), each with its reason in both
its runner (`ToolRunner/Tools/`) and `trigger.json`.

**A tool that does not fire is a result, not a failure.** What a family with *no*
firing tools cannot do is distinguish "correctly detected nothing" from "the scan
never ran" - so the source-level tools below are the built-in negative control:

- Lizard
- jscpd
- Semgrep
- PyDriller
- git log --numstat

| Tool | Status | Why |
|---|---|---|
| Lizard | active | Python tokeniser. Class B: own parser, never loads a compiler, so the TFM is irrelevant  |
| jscpd | active | Node, own tokeniser. Class B. |
| Dolos | not-installed | Node, tree-sitter C# grammar. Class B, and the sheet's declaration ('C# language-version |
| Semgrep | active | Python, tree-sitter C# grammar. Class B. |
| Opengrep | not-installed | Standalone binary, absent from this host. Class B when present. NOTE: the sheet declares |
| Trivy | not-installed | Standalone Go binary, absent from this host. Class C: reads manifests/lockfiles. |
| PyDriller | active | Python, reads git only. Class C: never touches the source language. |
| git log --numstat | active | stdlib git. Class C. Counts Co-authored-by trailers, not just the author field -- the Ty |
| diff-cover | skipped | Class C: reads Cobertura/OpenCover XML. No coverage report can be produced on this famil |
| Roslyn analyzers (cognitive complexity) | not-installed | The sheet says 'Roslyn', which is a compiler platform, not a package. Resolved here to S |
| Roslyn SAST analyzers | not-installed | The sheet says 'roslyn sast', also not a package. Resolved here to SecurityCodeScan + Mi |
| Roslyn SemanticModel.AnalyzeDataFlow | not-installed | PROPOSED REPAIR, not in the source sheet. The sheet assigns Data Flow to Stryker.NET, a  |
| dotnet list package --vulnerable | not-installed | This branch is SDK-style with PackageReference under Central Package Management, so `dot |
| Coverlet | skipped | net8.0 is fully supported; nothing about this target framework excludes Coverlet. It ins |
| AltCover | skipped | net8.0 is supported. AltCover rewrites compiled assemblies to record visits, and OrderKi |
| MiniCover | skipped | MiniCover 3.10.0 supports the .NET 8 SDK this branch uses (net10.0;net9.0;net8.0), so th |
| Stryker.NET | skipped | Stryker.NET supports this SDK-style net8.0 solution. It mutates and rebuilds the code un |
| OpenTelemetry (.NET) + OTLP exporter | skipped | net8.0 is supported. Spans come from a running, compiled app with the OpenTelemetry SDK  |
| unilyze | active | Real tool (github.com/bigdra50/unilyze, MIT, v0.6.1) -- not absent. Verified: live cyclomatic/cognitive comp |
## Build

```text
msbuild OrderKit.sln /t:Restore
msbuild OrderKit.sln /t:Build /p:Configuration=Release
```

Restore will not succeed on a host where NuGet is unreachable; see `dataset.json`
`blockedHosts`.

## Test

```text
msbuild OrderKit.sln /t:Build /p:Configuration=Release   # the test step is wired into the build
```

19 tests are declared. They are **not** executed in this corpus's build, because no
test framework can be restored.

## Tool data folders

This repo carries only `Tool Triggering (Synthetic Data)` from testable-platform/csharp-corpus's original per-branch task-data folders.

| Folder | What it holds |
|---|---|
| `Tool Triggering (Synthetic Data)` | 19 folders of planted data, one per runner, written to trigger its tool at this family's language version. Each folder's `DATA.md` says what is planted and what a correct run should measure. |

**This repo's own folder contains no project
manifest** -- no `.csproj`, `.sln`, `packages.config` or lockfile. One there
would make the platform treat the folder as a project of its own and run the
whole roster against it.

## Tool entry points

The harness is the C# console project `ToolRunner/` (one runner per tool under
`ToolRunner/Tools/`). It is kept out of `OrderKit.sln` and out of the tool data
folder, so it is neither built with the corpus nor detected as part of it.

```text
dotnet run --project ToolRunner                     # banner
dotnet run --project ToolRunner -- --verify         # every runner present
dotnet run --project ToolRunner -- --run            # run them all, compare to dataset.json
dotnet run --project ToolRunner -- --check          # cross-file consistency audit
dotnet run --project ToolRunner -- --tool semgrep   # one tool's runner
```

## What is expected to fire

`dataset.json` asserts the answer rather than describing it, so a platform that
reports nothing for a given tool can be *proven* correct instead of assumed correct.
Every runner's expected exit code is recorded there, and
`ToolRunner --run` exits **2** if any tool produces a result the dataset
does not already record - a new finding, as distinct from a measurement.

<!-- dummy PR 2026-11-16 10:28:00 -->
<!-- dummy PR 2026-11-17 07:33:47 -->
