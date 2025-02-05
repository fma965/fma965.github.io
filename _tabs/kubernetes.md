---
icon: mdi mdi-kubernetes
order: 60
layout: page
---

I decided to use the following setup for my Kubernetes cluster

- Debian 12
- K3S
- KubeVIP
- MetalLB
- Traefik
- Longhorn

# Nodes

Currently this cluster consists of 9 nodes

### Master Nodes
k3s-master-01 @ F9-HV1
k3s-master-02 @ F9-HV2
k3s-master-03 @ F9-HV3

### Worker Nodes
k3s-worker-01 @ F9-HV1
k3s-worker-02 @ F9-HV2
k3s-worker-03 @ F9-HV3

### Storage (Longhorn) Nodes
k3s-longhorn-01 @ F9-HV1
k3s-longhorn-02 @ F9-HV2
k3s-longhorn-03 @ F9-HV3

# Management
I currently mostly use K9S and occasionaly rancher to manage the cluster.

# GitOps
I use ArgoCD to deploy my Kubernetes workloads with full history tracking etc using Git.

The Git repo is currently private but this is due to potential secrets and stuff being contained in there, i will eventually migrate these to a secret storage and then share the repo.

# Longhorn
Longhorn is used for AppData for my K3S Cluster, Longhorn also routinely does backups to my MinIO instance