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

1. Looks up `tools/<name>/catalog.yaml` in this repository.
2. Uses its `detect` signatures to check whether the tool is already available
   locally on the customer's server.
3. If it is not available and the recipe is installable: downloads the pinned
   asset, verifies its sha256 against the value recorded here, extracts it, and
   makes the binary available locally — nothing beyond what the recipe declares.
   A `system` recipe is detect-and-verify only; the catalog never installs an
   operating-system package and returns `system_package_not_installable` instead.
4. Requests the declared credential environment variables, runs the verification
   command, and uses the log-source hint when configuring monitoring.
5. If not found in this catalog: reports back that the tool is not
   available. No other action is taken — this repository is the only source
   the agent is allowed to install from.

## `catalog.yaml` fields

```yaml
name: <tool name>
description: <one line>
superseded_by: <replacement tool name>  # optional; used by deprecated entries
license: <SPDX identifier, must already be on the product's allowlist>
capabilities_hint: [<short tags describing what it's good for>]
install_method: archive | pip | system
source:                         # omitted for install_method: system
  repo: <upstream project URL, for humans>
  version: <exact pinned version/tag>
  asset: <exact download URL for this version>
  sha256: <sha256 of that exact file, verified against vendor checksums>
  archive: tar.gz | tgz | wheel   # only for install_method: archive/pip
  binary_path: <path to the executable inside the extracted archive>
                # omitted for install_method: pip
detect:                         # at least one signature; no other sub-keys
  process_name: <string or list of process-name substrings>
  listening_port: <port integer or list of port integers>
  binary_path: <absolute path or list of absolute executable paths>
credential_env_vars: [<UPPER_SNAKE credential variable names; may be empty>]
verify_command: <non-empty command line; exit 0 means connected. NOT run through a shell: no ; | & > < ` or $( ; first word must be this tool's own binary; only $NAME of credential_env_vars is expanded>
log_source_hint:
  kind: none | file | syslog_identifier  # kind: none permits no other keys
  path: <default log path template; required only for kind: file>
  tag: <syslog tag; required only for kind: syslog_identifier>
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
| `mgc` | v1.9.0 | MIT | Deprecated; superseded by `mindalert-graph` |
| `gcalcli` | 4.5.1 | MIT | Google Calendar |
| `usql` | v0.21.6 | MIT | Multi-database SQL (MySQL, SQL Server, Oracle, SQLite, Snowflake, …) |
| `docker` | 29.8.2 | Apache-2.0 | Container inspection/logs/lifecycle |
| `gws` | v0.22.5 | Apache-2.0 | Google Workspace APIs |
| `ssh` | system | BSD-3-Clause | Secure remote commands |
| `tgctl` | v0.5.1 | MIT | Telegram Bot API messaging and forum threads |
| `cpdctl` | v1.10.22 | Apache-2.0 | IBM Cloud Pak for Data and DataStage |
| `iics` | v0.5.6 | Apache-2.0 | Informatica IICS/IDMC resources |
| `mindalert-slack` | v0.1.1 | MIT | Slack history and messaging |
| `mindalert-discord` | v0.1.0 | MIT | Discord history and messaging |
| `mindalert-graph` | v0.1.0 | MIT | Outlook mail and Teams messages |

The original seven entries were verified 2026-09-27; Docker, GWS and SSH were
verified 2026-10-03; tgctl, cpdctl and iics were verified 2026-10-03.
The three MindAlert tools were verified against their v0.2.1 release assets
and published checksums on 2026-10-03. Deprecated entries remain available for
compatibility and identify their replacement with `superseded_by`.
