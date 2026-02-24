<div align="center">

# 🏠 home-ops

### ☸ Kubernetes cluster powering my homelab

_... managed with Flux, running on Talos Linux, deployed on a single Beelink mini-PC_ ⚡

[![Talos](https://img.shields.io/badge/Talos-v1.11.3-orange?style=for-the-badge&logo=talos)](https://www.talos.dev/)
[![Kubernetes](https://img.shields.io/badge/Kubernetes-v1.34.1-blue?style=for-the-badge&logo=kubernetes&logoColor=white)](https://kubernetes.io/)
[![Flux](https://img.shields.io/badge/Flux-GitOps-5468ff?style=for-the-badge&logo=flux&logoColor=white)](https://fluxcd.io/)
[![Renovate](https://img.shields.io/github/actions/workflow/status/Seegar99/home-ops/renovate.yaml?branch=main&label=&logo=renovatebot&style=for-the-badge&color=1a1f6c)](https://github.com/Seegar99/home-ops/actions/workflows/renovate.yaml)

</div>

---

## 📖 Overview

This repository defines the entire state of my single-node Kubernetes cluster using a GitOps approach. Every application, secret, and piece of infrastructure is declared in code and automatically reconciled by [Flux](https://fluxcd.io/).

The cluster runs on a **Beelink EQi12** mini-PC (Intel i5-1235U, 24GB RAM, 500GB NVMe) with [Talos Linux](https://www.talos.dev/) as the OS. Based on the [onedr0p/cluster-template](https://github.com/onedr0p/cluster-template).

## 🏗️ Architecture

```
Internet
    │
    ▼
Cloudflare Tunnel ──► Envoy Gateway ──► Apps
    │                       │
    ▼                       ▼
 external-dns         HTTPRoutes
 (DNS records)    (ingress routing)
```

All external traffic flows through a [Cloudflare Tunnel](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/) — no ports are opened on the home network. [Envoy Gateway](https://gateway.envoyproxy.io/) handles ingress routing via the Kubernetes Gateway API.

## 🖥️ Hardware

| | |
|---|---|
| **Device** | Beelink EQi12 |
| **CPU** | Intel Core i5-1235U (10C/12T) |
| **RAM** | 24GB LPDDR5 |
| **Storage** | 500GB NVMe SSD |
| **Network** | Dual Gigabit LAN |
| **OS** | Talos Linux v1.11.3 |

## ⚙️ Cluster Components

### 🧱 Core Infrastructure

| Component | Description |
|---|---|
| [Talos Linux](https://www.talos.dev/) | Immutable Kubernetes OS |
| [Flux](https://fluxcd.io/) | GitOps reconciliation |
| [Cilium](https://cilium.io/) | CNI and network policies |
| [CoreDNS](https://coredns.io/) | Cluster DNS |
| [SOPS](https://github.com/getsops/sops) | Secret encryption (age) |

### 🌐 Networking

| Component | Description |
|---|---|
| [Envoy Gateway](https://gateway.envoyproxy.io/) | Gateway API ingress |
| [Cloudflare Tunnel](https://github.com/cloudflare/cloudflared) | Zero-trust external access |
| [external-dns](https://github.com/kubernetes-sigs/external-dns) | Automated Cloudflare DNS records |
| [k8s-gateway](https://github.com/ori-edge/k8s_gateway) | Internal split-horizon DNS |
| [cert-manager](https://cert-manager.io/) | TLS certificate management |

### 📊 Monitoring

| Component | Description |
|---|---|
| [Prometheus](https://prometheus.io/) + [Grafana](https://grafana.com/) | Metrics and dashboards (`grafana.colinpobrien.com`) |
| [Loki](https://grafana.com/oss/loki/) | Log aggregation |
| [Alloy](https://grafana.com/oss/alloy/) | Telemetry collection |

### 📺 Applications

| App | Description |
|---|---|
| [Jellyfin](https://jellyfin.org/) | Media server 🎬 |
| [Homarr](https://homarr.dev/) | Dashboard (`home.colinpobrien.com`) 🏠 |

### 🔧 System

| Component | Description |
|---|---|
| [Spegel](https://github.com/spegel-org/spegel) | Peer-to-peer container image registry |
| [Reloader](https://github.com/stakater/Reloader) | Auto-restart on config changes |
| [local-path-provisioner](https://github.com/rancher/local-path-provisioner) | NVMe-backed PersistentVolumes |
| [metrics-server](https://github.com/kubernetes-sigs/metrics-server) | Resource metrics |

## 📁 Repository Structure

```
kubernetes/
├── apps/                   # Application manifests
│   ├── cert-manager/       # TLS certificates
│   ├── default/            # User-facing apps (echo, jellyfin, homarr)
│   ├── flux-system/        # Flux operator and instance
│   ├── kube-system/        # Core services (cilium, coredns, storage)
│   ├── monitoring/         # Observability stack
│   └── network/            # Ingress and DNS
├── bootstrap/              # Initial cluster bootstrap manifests
└── flux/                   # Flux configuration
talos/                      # Talos machine configuration and patches
```

## 🔌 Networking

| Name | CIDR / Address |
|---|---|
| Home LAN | `192.168.1.0/24` |
| Gateway | `192.168.1.1` |
| Kube API | `192.168.1.230` |
| Internal gateway | `192.168.1.231` |
| External gateway | `192.168.1.232` |
| DNS gateway | `192.168.1.233` |

## 🛠️ Tooling

| Tool | Purpose |
|---|---|
| [mise](https://mise.jdx.dev/) | Dev environment and CLI tool management |
| [Renovate](https://www.mend.io/renovate) | Automated dependency updates |
| [flux-local](https://github.com/allenporter/flux-local) | HelmRelease and Kustomization diffs |
| [GitHub Actions](https://github.com/features/actions) | CI/CD workflows |

## 🤝 Acknowledgments

Built from the [onedr0p/cluster-template](https://github.com/onedr0p/cluster-template) with help from the [Home Operations](https://discord.gg/home-operations) community and [Claude Code](https://docs.anthropic.com/en/docs/claude-code). 🤖
