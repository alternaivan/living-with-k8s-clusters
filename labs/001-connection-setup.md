# Connection to Kubernetes Cluster Setup Guide

This guide will help you get started with a couple of K8s implementation - k3s, minikube, and connection to AKS.

## K3s

Lightweight Kubernetes. Easy to install, half the memory, all in a binary of less than 100 MB. K3s is a fully compliant Kubernetes distribution with the following enhancements:

- Distributed as a single binary or minimal container image.
- Lightweight datastore based on sqlite3 as the default storage backend. etcd3, MySQL, and Postgres are also available.
- Wrapped in simple launcher that handles a lot of the complexity of TLS and options.
- Secure by default with reasonable defaults for lightweight environments.
- Operation of all Kubernetes control plane components is encapsulated in a single binary and process, allowing K3s to automate and manage complex cluster operations like distributing certificates.
- External dependencies have been minimized; the only requirements are a modern kernel and cgroup mounts.
- Packages the required dependencies for easy "batteries-included" cluster creation:
  - containerd / cri-dockerd container runtime (CRI)
  - Flannel Container Network Interface (CNI)
  - CoreDNS Cluster DNS
  - Traefik Ingress controller
  - ServiceLB Load-Balancer controller
  - Kube-router Network Policy controller
  - Local-path-provisioner Persistent Volume controller
  - Spegel distributed container image registry mirror
  - Host utilities (iptables, socat, etc)

To get started, go to the below link:

https://docs.k3s.io/quick-start

## Minikube

minikube is local Kubernetes, focusing on making it easy to learn and develop for Kubernetes.

All you need is Docker (or similarly compatible) container or a Virtual Machine environment, and Kubernetes is a single command away: minikube start

What you’ll need:

- 2 CPUs or more
- 2GB of free memory
- 20GB of free disk space
- Internet connection
- Container or virtual machine manager, such as: Docker, QEMU, Hyperkit, Hyper-V, KVM, Parallels, Podman, VirtualBox, or VMware Fusion/Workstation

To get started, visit the following link:

https://minikube.sigs.k8s.io/docs/start/?arch=%2Fwindows%2Fx86-64%2Fstable%2F.exe+download

## Understanding kubectl Command Structure

The `kubectl` command line tool is used to interact with Kubernetes clusters. The general structure of a `kubectl` command is as follows:

```bash
kubectl [command] [TYPE] [NAME] [flags]
```

Here's what each part means:

- [`command`]: This specifies the operation that you want to perform on one or more resources, such as create, get, describe, delete.
- [`TYPE`]: This is the type of resource to perform the operation on. Resources include pods (po), services (svc), deployments (deploy), and others.
- [`NAME`]: This is the name of the resource. Names are case-sensitive. If the name is omitted, details for all resources are displayed.
- [`flags`]: These are optional and allow you to specify additional options.

Here's an example command:

```bash
kubectl get pods my-pod
```

In this example, get is the command, pods is the type, and my-pod is the name of the resource we're retrieving information about.

