# Local Development Environment for Kubernetes with Crossplane, LocalStack, ArgoCD, Podman, Devbox, and Task

This repository provides a local development environment for creating Kubernetes clusters using [Kind](https://kind.sigs.k8s.io/), provisioning AWS resources using [Crossplane](https://crossplane.io/), simulating AWS services locally with [LocalStack](https://localstack.cloud/), and deploying applications using [ArgoCD](https://argo-cd.readthedocs.io/) for GitOps-based workflows. The setup uses **Podman** as the container runtime and leverages **Devbox** and **Task** tools for automation.

## Overview

The goal of this repository is to enable developers to:
- Set up a local Kubernetes cluster using Kind.
- Use Crossplane to manage cloud resources declaratively.
- Simulate AWS services locally using LocalStack for testing and development.
- Deploy applications using ArgoCD and manage them via a Web UI.
- Automate the setup process using Devbox and Task.

This setup is ideal for local development, testing, and experimentation without incurring cloud costs.

## Features

- **Kind**: Lightweight Kubernetes clusters for local development.
- **Crossplane**: Declarative cloud resource provisioning directly from Kubernetes.
- **LocalStack**: Local simulation of AWS services for testing and development.
- **ArgoCD**: GitOps tool for deploying and managing applications in Kubernetes.
- **Podman**: A daemonless container runtime for managing containers.
- **Devbox**: A tool for creating reproducible development environments.
- **Task**: A task runner for automating commands and workflows.

## Prerequisites

Before you begin, ensure you have the following installed on your machine:
- [Podman](https://podman.io/)
- [Devbox](https://www.jetpack.io/devbox/)

### 1. Install Devbox
For Linux and MacOs:
```bash
curl -fsSL https://get.jetify.com/devbox | bash
```

For Windows: https://jetify-com.vercel.app/docs/devbox/installing_devbox/?install-method=wsl

Find more details here: https://jetify-com.vercel.app/docs/devbox/quickstart/

DevBox will install everything else what is needed to work with local development environment in isolated manner. 

### 2. Replace Docker Desktop with Podman (Optional)

```bash
alias docker=podman
export KIND_EXPERIMENTAL_PROVIDER=podman
```

Podman machine resources used in this project:
- CPU: 8
- Memory: 12GB
- Disk size: 100GB

Run podman machine with root to use host ports http (80) and https (443) in kind cluster configuration (`./kind-cluster/cluster_config.yaml`).
```bash
podman machine set --rootful=true
```

## Getting Started

Follow these steps to set up the local development environment:

### 1. Clone the Repository

```bash
git clone https://github.com/CloudOpsDemos/local-dev-k8s-crossplane-localstack.git
cd local-dev-k8s-crossplane-localstack
```

### 2. Run DevBox

```bash
devbox shell
```

It may take some time till devbox ends up installation of all tools and dependencies (~5min).

## Task Tool Usage

This repository includes a `Taskfile.yml` to automate common setup and management tasks. Below is the list of available tasks and their descriptions:

### Task List

| Task Name                          | Description                                                                 |
|------------------------------------|-----------------------------------------------------------------------------|
| **01-setup**                       | Ensure `kind` and `kubectl` are installed and ready to use.                 |
| **02-create-cluster**              | Create a Kubernetes cluster using Kind with Podman runtime.                 |
| **03-delete-cluster**              | Delete the Kubernetes cluster.                                              |
| **04-install-ingress**             | Install `ingress-nginx` to the cluster.                                     |
| **05-delete-ingress**              | Delete `ingress-nginx` from the cluster.                                    |
| **06-test-ingress**                | Test `ingress-nginx` with a sample application.                             |
| **07-delete-ingress**              | Delete ingress and sample application resources.                            |
| **08-add-argocd**                  | Add ArgoCD to the cluster.                                                  |
| **09-delete-argocd**               | Delete ArgoCD from the cluster.                                             |
| **10-add-crossplane-localstack**   | Add Crossplane and Localstack to the cluster.                               |
| **11-delete-crossplane-localstack**| Delete Crossplane from the cluster.                                         |
| **12-install-providers**           | Install AWS provider for Crossplane.                                        |
| **13-add-test-app**                | Deploy test app to the cluster.                                             |
| **14-delete-test-app**             | Delete test app from the cluster.                                           |

---

### How to Use Task

1. **List Available Tasks**
    To see the list of available tasks, run:
    ```bash
    task --list
    ```
2. **Run a specific task**
    Replace <task-name> with the name of the task you want to run (e.g., `02-create-cluster`).
    ```bash
    task <task-name>
    ```

## Troubleshooting

### Task step dependencies

It may happen that running task too fast invokes issues related with status of provisioned resources. E.g. `task 04-install-ingress` will need some time for running. Check its status first:

```bash
local-dev-k8s-crossplane-localstack % k get pods -n ingress-nginx
NAME                                        READY   STATUS      RESTARTS   AGE
ingress-nginx-admission-create-k4kl7        0/1     Completed   0          2m46s
ingress-nginx-admission-patch-bzddz         0/1     Completed   0          2m46s
ingress-nginx-controller-758f86dfc8-krxlj   1/1     Running     0          2m46s
```

It make sens to run the next task (`06-test-ingress`) only if `ingress-nginx-controller` is running.

### DevBox returns complete:13: command not found: compdef (MacOS 15)

```bash
devbox shell
Info: Ensuring packages are installed.
✓ Computed the Devbox environment.
Starting a devbox shell...
complete:13: command not found: compdef
```

Add to `~/.zshrc` file:
```bash
autoload -Uz compinit && compinit
source ~/.zshrc
```

### DebBox returns 'git' not found (MacOS 15)

```bash
sudo xcode-select --reset
sudo xcode-select --switch /Library/Developer/CommandLineTools
```

### ArgoCD is not accesible via http://argocd.local:38080

Check if `kubectl port-forward` is processing:
```bash
ps aux | grep port-forward
```

If not, run command manually:
```bash
kubectl port-forward svc/argocd-server -n argocd 38080:80 > /dev/null 2>&1 &
```