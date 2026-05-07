# basis-server-builder

Builds bare-metal binaries of [BasisVR/basis](https://github.com/BasisVR/basis) `BasisNetworkConsole` for use by the runner's bare-metal launch path. Upstream does not publish releases — this repo fills that gap.

## What it builds

A daily GitHub Actions workflow checks two upstream branches:

| Channel  | Source branch                              | Tag prefix |
| -------- | ------------------------------------------ | ---------- |
| nightly  | `developer`                                | `nightly-` |
| stable   | latest `long-term-support-*` (lex max)     | `stable-`  |

For each channel that has new commits, the workflow runs `dotnet publish --self-contained` against `Basis Server/BasisServerConsole/BasisNetworkConsole.csproj` (`net10.0`) for these RIDs:

- `win-x64`
- `linux-x64`
- `osx-arm64`

Each output is packaged as `basis-server-<rid>.tar.gz` and attached to a release tagged `<channel>-<short-sha>` (e.g. `nightly-abc1234`). Nightly releases are marked prerelease; stable releases are marked latest.

## Schedule

- Cron: every day at 06:00 UTC
- Manual: `workflow_dispatch` (with optional `force` input to rebuild even if a release for the current SHA already exists)

If the upstream SHA hasn't changed since the last build, the channel is skipped. Releases are immutable.

## Consuming the releases

The runner reads `basis_release_repo` from `.config` and queries `GET /repos/{owner}/{repo}/releases?per_page=30`, filtering tags by the configured `release_channel` prefix and picking the most recent. It then downloads the asset whose name matches the host RID, extracts to `<runner>/.data/basis/<tag>/`, and runs `BasisNetworkConsole[.exe]` directly.
