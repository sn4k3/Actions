# Shared GitHub Actions workflows

Reusable workflows for extracting .NET release metadata and publishing NuGet packages.

## Extract release information

The workflow reads `Version`, `VersionPrefix`/`VersionSuffix`, or `AssemblyVersion` from an MSBuild project or props file and returns the matching `CHANGELOG.md` entry.

```yaml
jobs:
  release-information:
    uses: sn4k3/Actions/.github/workflows/extract-csproj-information.yml@main
    with:
      csproj-path: Directory.Build.props

  consume:
    needs: release-information
    runs-on: ubuntu-latest
    steps:
      - run: echo "Releasing ${{ needs.release-information.outputs.version }}"
```

Supported changelog headings include both `# v1.2.3 (10/09/2026)` and Keep a Changelog-style `## [1.2.3] - 2026-09-10` entries.

## Publish a NuGet package

Call the publisher from a repository containing the package source and `CHANGELOG.md`. Dry runs are enabled by default: they pack the project and retain the packages as a workflow artifact without granting publish jobs write access.

```yaml
name: Publish package

on:
  workflow_dispatch:
    inputs:
      dry_run:
        description: Pack without publishing
        type: boolean
        default: true

jobs:
  publish:
    permissions:
      contents: write
      packages: write
    uses: sn4k3/Actions/.github/workflows/nuget-publish.yml@main
    with:
      dry_run: ${{ inputs.dry_run }}
      project-path: .
      csproj-path: Directory.Build.props
      dotnet-version: 10.0.x
    secrets:
      NUGET_TOKEN: ${{ secrets.NUGET_TOKEN }}
```

For stable consumers, replace `@main` with a release tag or commit SHA. A non-dry run publishes to NuGet.org and, by default, GitHub Packages; then it creates an idempotent `v<version>` tag and GitHub release with the packages attached.
