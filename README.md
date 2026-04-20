# Apache CloudStack CMK Skill for OpenClaw

[![OpenClaw](https://img.shields.io/badge/OpenClaw-2026.4.9-blue)](https://openclaw.ai) [![License](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](LICENSE)

A comprehensive OpenClaw skill for managing **Apache CloudStack** environments via the [`cmk`](https://github.com/apache/cloudstack-cloudmonkey) (CloudMonkey) CLI. Provides programmatic access to 870+ CloudStack APIs with safety-guarded approval workflows.

---

## 🌟 Features

- **Full API Coverage**: Access all Apache CloudStack operations (VMs, Networks, Kubernetes clusters, Security Groups, etc.) through standardized `cmk` commands
- **Safety First**: Built-in approval workflows for destructive operations (`destroy`, `purge`) with distinction between permanent and recoverable deletions
- **API Discovery**: Automatic caching of CloudStack APIs via `cmk sync` — query with `jq` when documentation is sparse
- **OpenClaw Integration**: Native support for sub-agents, background tasks, and structured error handling

## 🚀 Prerequisites

Before using this skill, ensure the [`cmk`](https://github.com/apache/cloudstack-cloudmonkey) (CloudMonkey) CLI is installed on your system.

```bash
# Example installation (ensure you follow the official instructions)
# Download the binary from GitHub releases or use a package manager if available
```

## 📚 Quick Start

### Installation

```bash
# Clone skill into your OpenClaw workspace
git clone https://github.com/chunkyen/cloudstack-cmk-skill.git \
  ~/.openclaw/workspace/skills/cloudstack-cmk/
```

### Configuration

Configure your CloudStack endpoint in `~/.cmk/config`:

```bash
# Set up profile
cmk -p localcloud set url https://your-cloudswalk-api.com/client/api
cmk -p localcloud set username admin@domain
cmk -p localcloud set password YOUR_SECRET_KEY

# Alternatively, use API Key and Secret Key
cmk -p localcloud set apikey YOUR_API_KEY
cmk -p localcloud set secretkey YOUR_SECRET_KEY

# Sync APIs from your server (downloads 870+ endpoints to cache)
cmk -p localcloud sync
```

## 🛠️ Usage Examples

### Basic Operations

```bash
# List VMs in zone
cmk -p localcloud list virtualmachines zone=x79zone | jq '.virtualmachine[] | {name, state, ipaddress}'

# Start a stopped Kubernetes cluster
cmk -p localcloud start kubernetescluster id=63a2cd00-d81f-4e49-aa21-b5fd0daf6e4a

# Download kubeconfig for CKS (CloudStack-managed Kubernetes)
cmk -p localcloud getKubernetesClusterConfig id=<UUID> | \
  jq -r '.clusterconfig.configdata' > ~/.kube/config-v135
```

### Advanced: API Cache Discovery

When you need an obscure command not obvious from help text:

```bash
# Find all Kubernetes-related APIs
cat ~/.cmk/profiles/localcloud.cache | \
  jq -r '.api[]? | select(.name|test("kube"; "i"))' | head -50
```

## ⚠️ Safety & Approvals

This skill enforces a **two-tier approval system** for destructive operations:

| Operation Type | Examples | Recovery Possible? | Approval Required |
|----------------|----------|-------------------:|------------------:|
| **Permanent Destruction** | `destroy virtualmachine`, `purge volumes` | ❌ No (data permanently removed) | ✅ Explicit `/approve` required |
| **Recoverable Delete** | `delete kubernetescluster` | ✅ Yes (soft delete, purge later) | ⚠️ Standard confirmation |

## 📖 Documentation Reference

- **Local API Cache**: `~/.cmk/profiles/localcloud.cache` — JSON file containing all discovered APIs with parameters, types, and descriptions.
- **Web API Docs**: [Apache CloudStack 4.22 API Documentation](https://cloudstack.apache.org/api/apidocs-4.22/)

## 🔧 Common Commands Reference

| Command | Description | Example |
|---------|-------------|---------|
| `list virtualmachines` | List VMs with filters | `cmk -p localcloud list virtualmachines zone=x79zone state=Running` |
| `stop/start kubernetescluster` | Power control CKS clusters | `cmk -p localcloud stop kubernetescluster id=<UUID>` |
| `getKubernetesClusterConfig` | Download kubeconfig for v1.34+ clusters | See Quick Start above |
| `sync` | Update API cache from server | `cmk -p localcloud sync`

## 🤝 Contributing

This skill follows the OpenClaw AgentSkills specification:
- Place in `~/.openclaw/workspace/skills/cloudstack-cmk/SKILL.md`
- Uses `exec` tool for non-interactive CLI operations
- Supports sub-agents for complex multi-step workflows

## 📝 License

Licensed under the Apache License, Version 2.0 — see [LICENSE](LICENSE) for details.

---

*Built with ❤️ for OpenClaw and Apache CloudStack users.*
