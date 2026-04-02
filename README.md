<div align="center">

# 🏠 home-ops

### ☸ Kubernetes cluster powering my homelab

_... managed with Flux, running on Talos Linux, deployed on a single Beelink mini-PC_ ⚡

[![Talos](https://img.shields.io/badge/Talos-v1.11.3-orange?style=for-the-badge&logo=talos)](https://www.talos.dev/)
[![Kubernetes](https://img.shields.io/badge/Kubernetes-v1.34.1-blue?style=for-the-badge&logo=kubernetes&logoColor=white)](https://kubernetes.io/)
[![Flux](https://img.shields.io/badge/Flux-v2.8.3-5468ff?style=for-the-badge&logo=flux&logoColor=white)](https://fluxcd.io/)
[![Status Page](https://img.shields.io/badge/Status_Page-online-green?style=for-the-badge)](https://status.colinpobrien.com)

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
    │                   │
    ▼                   ├── grafana.colinpobrien.com   ──► Grafana
 external-dns           ├── jellyfin.colinpobrien.com  ──► Jellyfin
 (auto DNS)             ├── status.colinpobrien.com    ──► Gatus
                        └── echo.colinpobrien.com      ──► Echo
```

All external traffic flows through a [Cloudflare Tunnel](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/) — no ports are opened on the home network. [Envoy Gateway](https://gateway.envoyproxy.io/) handles ingress routing via the Kubernetes Gateway API.

## 🖥️ Hardware

| | |
|---|---|
| **Device** | Beelink EQi12 |
| **CPU** | Intel Core i5-1235U (10C/12T, up to 4.4GHz) |
| **RAM** | 24GB LPDDR5 |
| **Storage** | 500GB NVMe SSD |
| **Network** | Dual Gigabit LAN, WiFi 6 |
| **OS** | Talos Linux v1.11.3 |
| **Role** | Single-node (control plane + worker) |

## ⚙️ Cluster Components

### 🧱 Core Infrastructure

| Component | Description |
|---|---|
| [Talos Linux](https://www.talos.dev/) | Immutable Kubernetes OS |
| [Flux](https://fluxcd.io/) | GitOps reconciliation |
| [Cilium](https://cilium.io/) | eBPF-based CNI and network policies |
| [CoreDNS](https://coredns.io/) | Cluster DNS |
| [SOPS](https://github.com/getsops/sops) | Secret encryption with [age](https://github.com/FiloSottile/age) |

### 🌐 Networking

| Component | Description |
|---|---|
| [Envoy Gateway](https://gateway.envoyproxy.io/) | Gateway API ingress controller |
| [Cloudflare Tunnel](https://github.com/cloudflare/cloudflared) | Zero-trust external access |
| [external-dns](https://github.com/kubernetes-sigs/external-dns) | Automated Cloudflare DNS records |
| [k8s-gateway](https://github.com/ori-edge/k8s_gateway) | Internal split-horizon DNS |
| [cert-manager](https://cert-manager.io/) | TLS certificates via Let's Encrypt |

### 📊 Monitoring & Observability

| Component | Description |
|---|---|
| [Prometheus](https://prometheus.io/) + [Grafana](https://grafana.com/) | Metrics, dashboards, and alerting ([grafana.colinpobrien.com](https://grafana.colinpobrien.com)) |
| [Loki](https://grafana.com/oss/loki/) | Log aggregation |
| [Alloy](https://grafana.com/oss/alloy/) | Telemetry collection |
| [Gatus](https://gatus.io/) | Endpoint health monitoring + Discord alerts ([status.colinpobrien.com](https://status.colinpobrien.com)) |
| [Trivy Operator](https://aquasecurity.github.io/trivy-operator/) | Container image vulnerability scanning |

### 📺 Applications

| App | Description |
|---|---|
| [Jellyfin](https://jellyfin.org/) | Media server ([jellyfin.colinpobrien.com](https://jellyfin.colinpobrien.com)) 🎬 |

### 🔧 System

| Component | Description |
|---|---|
| [Spegel](https://github.com/spegel-org/spegel) | Cluster-local OCI image mirror |
| [Reloader](https://github.com/stakater/Reloader) | Auto-restart on config/secret changes |
| [local-path-provisioner](https://github.com/rancher/local-path-provisioner) | NVMe-backed PersistentVolumes |
| [metrics-server](https://github.com/kubernetes-sigs/metrics-server) | Resource metrics for HPA and kubectl top |

## 📁 Repository Structure

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

## 🔌 Networking

| Name | Address |
|---|---|
| Home LAN | `192.168.1.0/24` |
| Gateway | `192.168.1.1` |
| Kube API | `192.168.1.230` |
| Internal ingress | `192.168.1.231` |
| External ingress | `192.168.1.232` |
| DNS gateway | `192.168.1.233` |

## ☁️ Cloud Dependencies

| Service | Use | Cost |
|---|---|---|
| [Cloudflare](https://cloudflare.com) | Domain, DNS, tunnel | Free |
| [GitHub](https://github.com) | Repository, CI/CD, Actions | Free |
| [Discord](https://discord.com) | Alert notifications (Grafana + Gatus) | Free |
| | | **~$0/mo** |

## 🔄 CI/CD

| Workflow | Description |
|---|---|
| [Flux Local](https://github.com/Seegar99/home-ops/actions/workflows/flux-local.yaml) | HelmRelease and Kustomization validation |
| [YAML Lint](https://github.com/Seegar99/home-ops/actions/workflows/yaml-lint.yaml) | YAML syntax validation |
| [SOPS Validate](https://github.com/Seegar99/home-ops/actions/workflows/sops-validate.yaml) | Ensures secrets are properly encrypted |
| [Kubeconform](https://github.com/Seegar99/home-ops/actions/workflows/kubeconform.yaml) | Kubernetes manifest schema validation |
| [Renovate Config](https://github.com/Seegar99/home-ops/actions/workflows/renovate-validate.yaml) | Renovate configuration validation |
| [Release](https://github.com/Seegar99/home-ops/actions/workflows/release.yaml) | Monthly calendar-versioned releases |

## 🛠️ Tooling

| Tool | Purpose |
|---|---|
| [mise](https://mise.jdx.dev/) | Dev environment and CLI tool management |
| [Renovate](https://www.mend.io/renovate) | Automated dependency updates (GitHub App) |
| [Claude Code](https://docs.anthropic.com/en/docs/claude-code) | AI-assisted cluster building and maintenance |

## 🤝 Acknowledgments

Built from the [onedr0p/cluster-template](https://github.com/onedr0p/cluster-template) with help from the [Home Operations](https://discord.gg/home-operations) community.

This cluster was built and is actively maintained with [Claude Code](https://docs.anthropic.com/en/docs/claude-code) — from initial Talos bootstrapping and Flux GitOps setup through application deployments, secret encryption, monitoring configuration, CI/CD pipelines, and ongoing troubleshooting. 🤖
