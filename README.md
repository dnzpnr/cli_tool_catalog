# CLI Tool Catalog

A verified, version-pinned, hash-pinned catalog of the CLI tools our
team-agent product knows how to fetch and install for a customer. This
repository is **not** a place to download and run arbitrary software — every
entry here has already been license-checked (permissive licenses only —
MIT/BSD/Apache-family, matching the product's dependency policy) and its
exact release artifact hash-verified against the vendor's own published
checksums where available.

## How this is used

Each tool lives in its own directory under `tools/<name>/`, containing one
`catalog.yaml` file — the *install recipe*, not the tool's binary itself (we
never bundle third-party binaries in this repo; we pin an exact upstream
release and its hash instead, so a customer always gets the vendor's own
signed/published artifact).

When an operator asks (via Mattermost chat) for a new tool to be monitored,
the agent:

1. Checks whether that tool is already available locally on the customer's
   server.
2. If not, looks up `tools/<name>/catalog.yaml` in this repository.
3. If found: downloads the pinned asset, verifies its sha256 against the
   value recorded here, extracts it, and makes the binary available locally
   — nothing beyond what the recipe declares.
4. If not found in this catalog: reports back that the tool is not
   available. No other action is taken — this repository is the only source
   the agent is allowed to install from.

## `catalog.yaml` fields

```yaml
name: <tool name>
description: <one line>
license: <SPDX identifier, must already be on the product's allowlist>
capabilities_hint: [<short tags describing what it's good for>]
install_method: archive | pip
source:
  repo: <upstream project URL, for humans>
  version: <exact pinned version/tag>
  asset: <exact download URL for this version>
  sha256: <sha256 of that exact file, verified against vendor checksums>
  archive: tar.gz | tgz | wheel   # only for install_method: archive/pip
  binary_path: <path to the executable inside the extracted archive>
                # omitted for install_method: pip
```

## Updating an entry

Bump `version`, `asset` and `sha256` together, in one change — never one
without the others. Re-verify `sha256` against the vendor's own published
checksum file when one exists (`gh`, `jira-cli` both publish one); when it
doesn't, compute it directly from the downloaded file yourself.

## Current tools

| Tool | Version | License | Covers |
|---|---|---|---|
| `gh` | v2.101.0 | MIT | GitHub issues/PRs/releases |
| `jira-cli` | v1.7.0 | MIT | Jira tickets |
| `himalaya` | v2.1.0 | MIT | Email (IMAP/JMAP/Gmail/Microsoft Graph) |
| `gdrive` | 3.9.1 | MIT | Google Drive files |
| `mgc` | v1.9.0 | MIT | Microsoft 365 (Outlook/OneDrive/Teams/Calendar) |
| `gcalcli` | 4.5.1 | MIT | Google Calendar |
| `usql` | v0.21.6 | MIT | Multi-database SQL (MySQL, SQL Server, Oracle, SQLite, Snowflake, …) |

All verified 2026-09-27.
