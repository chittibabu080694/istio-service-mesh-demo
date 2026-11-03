# Production-Grade Service Mesh with Istio & Kubernetes

A hands-on cloud-native project demonstrating advanced microservices traffic management, zero-trust security, and resilience testing using **Istio Service Mesh** deployed on a **Kubernetes** cluster.

---

## 🚀 Architecture & Features

- **Automatic Sidecar Injection**: Leverages Envoy proxy sidecars injected automatically into application pods via namespace labeling.
- **Canary Deployments & Traffic Splitting**: Utilizes Istio `DestinationRule` and `VirtualService` to split production traffic dynamically (e.g., 80% to Version 1, 20% to Version 2 Canary).
- **Zero-Trust Security (mTLS)**: Enforces `STRICT` mutual TLS encryption for all intra-cluster service communication without modifying application code.
- **Resiliency & Fault Injection**: Tests system reliability and fault tolerance by introducing intentional HTTP delays and error injections.
- **Observability**: Integrates with Istio control plane telemetry (`istiod`) and proxy synchronization health checks.

---

## 🛠️ Tech Stack
- **Orchestration**: Kubernetes (`Minikube` / AWS EC2 `t2.2xlarge`)
- **Service Mesh**: Istio (v1.31) with Envoy Proxies
- **Networking**: `VirtualService`, `DestinationRule`, `PeerAuthentication`
- **Container Runtime**: Docker

---

## 📋 Quick Start Guide

### 1. Prerequisites
- Running Kubernetes cluster (`kubectl` configured)
- `istioctl` installed

### 2. Enable Istio & Automatic Sidecar Injection
```bash
istioctl install --set profile=demo -y
kubectl label namespace default istio-injection=enabled --overwrite

###3. Deploy Microservices & Traffic Rules
Bash
kubectl apply -f manifests/app-v1.yaml
kubectl apply -f manifests/app-v2.yaml
kubectl apply -f manifests/traffic-rule.yaml

### 4. Apply Security & Fault Testing
kubectl apply -f manifests/mtls-strict.yaml
kubectl apply -f manifests/fault-injection.yaml

---

## 5. Portfolio Architecture Document (`architecture.md`)
Save this as `architecture.md` in your repository for technical recruiters or system architecture reviews:

```markdown
# Architectural Overview: Istio Service Mesh Implementation

## 1. System Context & Topology
In modern microservices architectures, decoupling application logic from network routing, security, and telemetry is critical for scalability. This project implements a service mesh abstraction layer using **Istio** and **Envoy** proxies running inside a containerized Kubernetes cluster.

## 2. Core Components Breakdown

### Data Plane (Envoy Proxies)
- Intercepts all inbound and outbound network traffic for microservices.
- Automatically handles service discovery, health checks, load balancing, and mTLS encryption/decryption transparently.

### Control Plane (`istiod`)
- **Pilot**: Converts high-level Istio routing rules (`VirtualService`, `DestinationRule`) into standard Envoy configuration and dynamically propagates them to sidecars.
- **Citadel**: Manages automated cryptographic certificate generation, rotation, and mutual TLS identity provisioning.

## 3. Traffic Management Mechanics
- **Subsets**: `DestinationRule` establishes version-specific logical subsets (`v1` and `v2`) using Kubernetes pod labels.
- **Weighted Routing**: `VirtualService` acts as an intelligent router, allocating traffic percentages at the L7 HTTP layer without requiring client-side load balancing logic.

## 4. Security & Zero-Trust Posture
- Enforcing `PeerAuthentication` with `mode: STRICT` ensures that plaintext communication within the cluster is rejected. 
- Identity is verified cryptographically using SPIFFE IDs embedded in Istio-managed X.509 certificates exchanged during the TLS handshake.

┌───────────────────────────────────────────────┐
│ Kubernetes Cluster (Default Namespace)        │
│                                               │
│  ┌─────────────────[47m [0m                        │
│  │ Ingress Gateway │                           │
│  └────────┬────────┘                           │
│           │                                   │
│           ▼                                   │
│  ┌─────────────────────────────────────────┐  │
│  │ VirtualService (Traffic Split / Faults) │  │
│  └────┬───────────────────────────────┬────┘  │
│       │ 80%                           │ 20%   │
│       ▼                               ▼       │
│  ┌──────────────┐               ┌──────────┐  │
│  │ backend-v1   │               │ backend-v2│  │
│  │ + Envoy Proxy│               │+Envoy Prox│  │
│  └──────────────┘               └──────────┘  │
└───────────────────────────────────────────────┘
