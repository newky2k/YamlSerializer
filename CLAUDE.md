# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

`YamlSerializerNet` — a fork of Osamu Takeuchi's YamlSerializer (YAML 1.2 parser, serializer and deserializer for native C# objects). One NuGet package (`PackageId` = `YamlSerializerNet`, project `YamlSerializer/YamlSerializer.csproj`, assembly `YamlSerializer`) plus an NUnit test project (`YamlSerializerTest/YamlSerializerTestNew.csproj`).

## Build and test

```bash
dotnet restore YamlSerializer.sln
dotnet build YamlSerializer.sln -c Release
dotnet test YamlSerializerTest/YamlSerializerTestNew.csproj -c Release
```

- The library multi-targets `net461;netstandard2.1;net6.0;net8.0;net9.0;net10.0;net11.0` and has no package dependencies. The test project targets `net10.0-windows` and `net11.0-windows` (it uses `System.Drawing.Common`). Build with the .NET 10 and .NET 11 SDKs (`global.json` rolls forward to the latest major); on Linux or macOS add `-p:EnableWindowsTargeting=true`.
- The tests are written for Windows. On Linux about 14–18 of 181 fail (GDI+, line endings, culture and ordering differences); the unmodified code fails the same tests, so run the suite on Windows for a real result.
- `YamlSerializerTestNew.csproj` sets `RootNamespace` to `YamlSerializerTest` so the embedded `Properties.Resources` resource name matches the generated designer code. Do not remove it.
- `GeneratePackageOnBuild=true`, so every build of the library produces a `.nupkg` (with `README.md` and `DSIcon.png`). Release builds add SourceLink and an embedded PDB.

## Branching

- `development` is the working branch: branch from it and open pull requests into it. Never commit or open a pull request directly against `main`.
- `main` is the release branch. Every push to `main` runs `release.yml`, which publishes the package to nuget.org and creates a GitHub release, so only merge `development` into `main` when a release is intended, and only when asked to release.

## CI

GitHub Actions (`.github/workflows/`) on `windows-latest`:

- `ci.yml` builds Release for pull requests into `main` and `development`; it publishes nothing and does not run the tests.
- `release.yml` runs on every push to `main` (changes only to Markdown or workflow files are skipped; run it by hand to test a workflow change). It builds Release as `1.3.<yyMM>.<run number>` plus `RELEASE_SUFFIX` (empty for a stable version), pushes `YamlSerializerNet` to nuget.org with Trusted Publishing (OIDC, `NUGET_USER` secret, `nuget` environment), then tags the commit `v<version>` and creates a GitHub release with the package attached.
