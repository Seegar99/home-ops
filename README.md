<div align="center">

<img src="https://raw.githubusercontent.com/kubernetes/kubernetes/master/logo/logo.svg" align="center" width="144px" height="144px"/>

### Colin's Home Operations

_... managed with Flux, Renovate, and GitHub Actions_

</div>

<div align="center">

[![Discord](https://img.shields.io/discord/673534664354430999?style=for-the-badge&label&logo=discord&logoColor=white&color=blue)](https://discord.gg/home-operations)
[![Talos](https://img.shields.io/badge/Talos-v1.11.3-blue?style=for-the-badge&logo=talos&logoColor=white)](https://talos.dev)
[![Kubernetes](https://img.shields.io/badge/Kubernetes-v1.34.1-blue?style=for-the-badge&logo=kubernetes&logoColor=white)](https://kubernetes.io)
[![Flux](https://img.shields.io/badge/Flux-v2.8.3-blue?style=for-the-badge&logo=flux&logoColor=white)](https://fluxcd.io)
[![Renovate](https://img.shields.io/github/actions/workflow/status/Seegar99/home-ops/renovate.yaml?branch=main&label=&logo=renovatebot&style=for-the-badge&color=blue)](https://github.com/Seegar99/home-ops/actions/workflows/renovate.yaml)

</div>

<div align="center">

[![Status Page](https://img.shields.io/badge/Status_Page-status.colinpobrien.com-green?style=for-the-badge)](https://status.colinpobrien.com)
[![Grafana](https://img.shields.io/badge/Grafana-grafana.colinpobrien.com-orange?style=for-the-badge&logo=grafana&logoColor=white)](https://grafana.colinpobrien.com)

</div>

---

## Overview

This is a mono repository for my home Kubernetes cluster. I adhere to Infrastructure as Code (IaC) and GitOps practices using [Talos Linux](https://talos.dev), [Flux](https://fluxcd.io), [Renovate](https://www.mend.io/renovate), and [GitHub Actions](https://github.com/features/actions).

The cluster runs on a single **Beelink EQi12** mini-PC. All configuration is declared in this repo and automatically reconciled — push to `main` and Flux handles the rest.

Based on the [onedr0p/cluster-template](https://github.com/onedr0p/cluster-template).

---

## Kubernetes

### Core Components

- **Networking**: [Cilium](https://cilium.io) provides eBPF-based CNI. [Envoy Gateway](https://gateway.envoyproxy.io) handles ingress via the Kubernetes Gateway API. [Cloudflared](https://github.com/cloudflare/cloudflared) secures external access through a zero-trust tunnel — no ports opened on the home network. [external-dns](https://github.com/kubernetes-sigs/external-dns) keeps Cloudflare DNS records in sync automatically.
- **Certificates**: [cert-manager](https://cert-manager.io) automates TLS certificate lifecycle with Let's Encrypt.
- **Secrets**: [SOPS](https://github.com/getsops/sops) with [age](https://github.com/FiloSottile/age) encryption for secrets stored safely in this public repository.
- **Storage**: [local-path-provisioner](https://github.com/rancher/local-path-provisioner) for NVMe-backed PersistentVolumes. [Spegel](https://github.com/spegel-org/spegel) provides a cluster-local OCI image mirror.
- **Automation**: [Renovate](https://www.mend.io/renovate) watches for dependency updates and creates PRs automatically. [Reloader](https://github.com/stakater/Reloader) restarts workloads on config/secret changes.

### GitOps

[Flux](https://fluxcd.io) watches the `kubernetes/` directory and reconciles the cluster to match the state of the `main` branch. It recursively discovers `kustomization.yaml` files which reference Flux Kustomizations (`ks.yaml`), each controlling a HelmRelease or set of raw manifests.

```
Push to main  -->  Flux detects changes  -->  Reconciles cluster  -->  Apps updated
```

[Renovate](https://www.mend.io/renovate) scans the repo for outdated dependencies (Helm charts, container images, GitHub Actions) and opens PRs. Once merged, Flux applies the changes automatically.

### Directory Structure

```
kubernetes/
├── apps/                   # Application manifests
│   ├── cert-manager/       #   TLS certificate management
│   ├── default/            #   User-facing apps (echo, jellyfin, gatus)
│   ├── flux-system/        #   Flux operator and instance
│   ├── kube-system/        #   Core services (cilium, coredns, storage)
│   ├── monitoring/         #   Observability stack
│   └── network/            #   Ingress and DNS
├── bootstrap/              # Initial cluster bootstrap manifests
├── components/             # Reusable Kustomize components (SOPS)
└── flux/                   # Flux system configuration
talos/                      # Talos machine config and patches
```

---

## Applications

### Monitoring & Observability

| App | Description |
|-----|-------------|
| [Prometheus](https://prometheus.io) | Metrics collection and alerting rules |
| [Grafana](https://grafana.com) | Dashboards and visualization ([grafana.colinpobrien.com](https://grafana.colinpobrien.com)) |
| [Loki](https://grafana.com/oss/loki) | Log aggregation |
| [Alloy](https://grafana.com/oss/alloy) | Telemetry collection (logs, metrics) |
| [Gatus](https://gatus.io) | Endpoint health monitoring ([status.colinpobrien.com](https://status.colinpobrien.com)) |
| [Trivy Operator](https://aquasecurity.github.io/trivy-operator) | Container vulnerability scanning |

### User-Facing

| App | Description |
|-----|-------------|
| [Jellyfin](https://jellyfin.org) | Media server ([jellyfin.colinpobrien.com](https://jellyfin.colinpobrien.com)) |

### Networking

| App | Description |
|-----|-------------|
| [Envoy Gateway](https://gateway.envoyproxy.io) | Gateway API ingress controller |
| [Cloudflare Tunnel](https://github.com/cloudflare/cloudflared) | Zero-trust external access |
| [external-dns](https://github.com/kubernetes-sigs/external-dns) | Automated DNS record management |
| [k8s-gateway](https://github.com/ori-edge/k8s_gateway) | Internal split-horizon DNS |
| [cert-manager](https://cert-manager.io) | TLS certificates via Let's Encrypt |

---

## Network

```
Internet
    │
    ▼
Cloudflare Tunnel ──► Envoy Gateway ──► Apps
    │                   │
    ▼                   ├── grafana.colinpobrien.com   ──► Grafana
 external-dns           ├── jellyfin.colinpobrien.com  ──► Jellyfin
 (auto DNS)             ├── status.colinpobrien.com    ──► Gatus
                        └── echo.colinpobrien.com      ──► Echo
```

| Name | Address |
|------|---------|
| Home LAN | `192.168.1.0/24` |
| Gateway | `192.168.1.1` |
| Kube API | `192.168.1.230` |
| Internal ingress | `192.168.1.231` |
| External ingress | `192.168.1.232` |
| DNS gateway | `192.168.1.233` |

---

## Hardware

| | |
|---|---|
| **Device** | Beelink EQi12 |
| **CPU** | Intel Core i5-1235U (10C/12T, up to 4.4GHz) |
| **RAM** | 24GB LPDDR5 |
| **Storage** | 500GB NVMe SSD |
| **Network** | Dual Gigabit LAN, WiFi 6 |
| **OS** | Talos Linux v1.11.3 |
| **Role** | Single-node (control plane + worker) |

---

## Cloud Dependencies

| Service | Use | Cost |
|---------|-----|------|
| [Cloudflare](https://cloudflare.com) | Domain, DNS, tunnel | Free |
| [GitHub](https://github.com) | Repository, CI/CD, Actions | Free |
| [Discord](https://discord.com) | Alert notifications | Free |
| | | **~$0/mo** |

---

## CI/CD

| Workflow | Description |
|----------|-------------|
| [Renovate](https://github.com/Seegar99/home-ops/actions/workflows/renovate.yaml) | Automated dependency updates |
| [YAML Lint](https://github.com/Seegar99/home-ops/actions/workflows/yaml-lint.yaml) | YAML syntax validation |
| [SOPS Validate](https://github.com/Seegar99/home-ops/actions/workflows/sops-validate.yaml) | Ensures secrets are encrypted |
| [Kubeconform](https://github.com/Seegar99/home-ops/actions/workflows/kubeconform.yaml) | Kubernetes manifest validation |
| [Renovate Config](https://github.com/Seegar99/home-ops/actions/workflows/renovate-validate.yaml) | Renovate config validation |
| [Release](https://github.com/Seegar99/home-ops/actions/workflows/release.yaml) | Monthly calendar-versioned releases |

---

## Acknowledgments

Built from the [onedr0p/cluster-template](https://github.com/onedr0p/cluster-template) with help from the [Home Operations](https://discord.gg/home-operations) community.
