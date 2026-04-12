# microlab

This repository is a small, opinionated lab for running a minimal Kubernetes environment on an older laptop. This lab proves you can learn Kubernetes on older hardware without buying a new machine.

## System Snapshot

```yaml
OS: Ubuntu 24.04.4 LTS (Noble Numbat) x86_64
Host: Inspiron 3537 (A09)
Kernel: Linux 6.8.0-107-generic
Uptime: 2 hours, 15 mins
Packages: 475 (dpkg)
Shell: bash 5.2.21
Terminal: /dev/pts/0
CPU: Intel(R) Core(TM) i3-4010U (4) @ 1.70 GHz
GPU 1: AMD Radeon HD 8600M Series
GPU 2: Intel Haswell-ULT Integrated Graphics Controller @ 1.00 GHz [Integrated]
Memory: 1.31 GiB / 1.75 GiB (75%)
Swap: 71.97 MiB / 2.00 GiB (4%)
Disk (/): 6.60 GiB / 97.87 GiB (7%) - ext4
Local IP (enp1s0): 192.168.0.42/24
Battery (DELL 49VTP27J): 0% [AC Connected]
Locale: C.UTF-8
```

This is my actual, decade-old laptop (Dell Inspiron 3537); the specs above are exact and reflect physical hardware, not a virtual machine.

## Goals

- Provide a reproducible repo layout using a `base` and `lab` overlay (`apps/base` and `apps/lab`).
- Keep resource usage minimal so an older laptop can run k3s + the apps.
- Show pragmatic defaults (ephemeral storage by default, single replica workloads).

## Repo structure (high level)

- `apps/base/` — upstream app manifests (deployments, services, ingress, PVC templates).
- `apps/lab/` — local overlay for this machine: small tweaks, disable PVCs, use `emptyDir`, single replicas, node selectors.

This split lets you keep a clean base set of manifests while applying device-specific compromises in `apps/lab`.

## Quick start (on the laptop)

Follow these exact steps — this is how I installed k3s on my laptop:

```bash
# update and upgrade
sudo apt update
sudo apt upgrade -y

# install k3s
curl -sfL https://get.k3s.io | sh -s - --write-kubeconfig-mode 0644

# copy the `k3s.yaml` to home directory
mkdir -p ~/.kube && sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config

# change ownership of `k3s.yaml` file
whoami
sudo chown "$(whoami):$(whoami)" ~/.kube/config

# check nodes
k3s kubectl get nodes
```

If you want a lighter k3s footprint, pass `--disable` flags (for example `--disable traefik`) during install.

Apply the lab overlay from this repository (from the repo root):

```bash
kubectl apply -k apps/lab/
```

That applies the `lab` overlay which contains the small-device tweaks (single replicas, `emptyDir` for storage, node selectors, etc.).

## Low-resource tips

- Keep replicas = 1 and avoid heavy controllers or operators.
- Use `emptyDir` or small hostPath mounts for ephemeral data when persistence isn't required.
- Disable k3s components you don't need (traefik, servicelb) with `INSTALL_K3S_EXEC` or `--disable` flags.
- Use small images and set conservative resource requests/limits.
- Add a swapfile if you have little RAM — it's not ideal for performance but reduces crashes during installs.

## Persistence note

`emptyDir` is ephemeral. If you need true persistence, prepare a proper PVC backed by host storage or an external NAS. The `apps/base` contains PVC templates; the `apps/lab` overlay intentionally avoids binding to cluster-scoped PVs so the cluster can run without extra storage configuration.
