---
name: cloudstack-cmk
description: Apache CloudStack management via `cmk` (CloudMonkey) CLI
---

# CloudStack CMK Skill

**Purpose**: Manage Apache CloudStack resources via the `cmk` CLI tool.

## Use when:
- User asks to manage CloudStack cloud environment like list/start/stop/delete VMs, Kubernetes clusters, or other CloudStack resources
- Need to query CloudStack infrastructure (zones, networks, templates)
- Managing CKS (CloudStack-managed Kubernetes) clusters

## Inputs:
- Resource type: `virtualmachine`, `kubernetescluster`, `user`, etc.
- Action verb: `list`, `start`, `stop`, `delete`, `destroy`
- Sample parameters: `id=<UUID>`, `domainid=...`, `namepattern=...`

## Outputs:
- Resource listings (JSON/table)
- Operation status/confirmation

## Rules:
- **Always run**: `which cmk` first to verify binary exists
- **Always use**: `-p localcloud` flag for profile context but it can be changed by the user to another cloud profile name
- **Destructive operations** (`delete*`, `destroy*`, `expunge*`, `purge`) → **ALWAYS requires explicit user confirmation ("yes" or "approve") before executing**, regardless of resource type
  - Examples: `deleteVpc`, `deleteNetwork`, `destroyVirtualMachine`, `expungeVolume`
- **Standard lifecycle ops** (start, stop, update, list) → standard confirmation only

**Note**: Some resources (VMs, volumes) have a destroy→expunge lifecycle (~24h default). They sit in "Destroyed" state and can be recovered via `recover*` APIs until permanently expunged. Networks/VPCs/IPs delete immediately without recovery window.
- **Unknown commands**? Run `cmk -p localcloud sync` to refresh API cache at `~/.cmk/profiles/localcloud.cache`
- Query parameters via: `<verb> <resource> -h` (e.g., `list users -h`)
- Cache is JSON — use `jq` to explore endpoints. For example:
  - List all APIs starting with "list": `jq '.api[] | select(.name | startswith("list"))' ~/.cmk/profiles/localcloud.cache`
  - Find Kubernetes-related APIs: `jq -r '.api[] | select(.name | ascii_downcase | contains("kubernetes")) | .name' ~/.cmk/profilename.cache`

Note: Replace `localcloud.cache` with `<profile-name>.cache` depending on your active profile name.

## Steps:
1. Verify: `which cmk`
2. Construct command: `cmk -p localcloud <verb> <resource> [args]`
3. For params help: append `-h` flag
4. Execute via `exec` tool
5. Parse JSON output with `jq` if needed

## Command Examples:
| Action | Command |
|--------|---------|
| List VMs | `cmk -p localcloud list virtualmachines domainid=<UUID>` |
| Start VM | `cmk -p localcloud start virtualmachine id=<UUID>` |
| Stop CKS cluster | `cmk -p localcloud stop kubernetescluster id=<UUID>` |
| List clusters | `cmk -p localcloud list kubernetesclusters` |

## Do not:
- Run `destroy`/`purge` without explicit `/approve:yes/no`
- Skip `which cmk` verification — binary may be missing from PATH
- Assume commands work without running `sync` first on new setups
