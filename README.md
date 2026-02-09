# GitOps Demo: A devcontainer rendition that ships with all batteries included

This repository contains a small GitOps demo with 3 Argo CD applications.
You can setup the environment locally with VSCode or create a GitHub Codespace, the latter option is easier on public wifi.

## Prerequisites

- Docker Desktop (or another OCI runtime) running on your machine.
- VS Code (for local devcontainers) and the "Dev Containers" extension (Remote - Containers).
- Optional: GitHub Codespaces access (for Codespaces workflow).
- If working outside the devcontainer: task (go-task), kind, kubectl.

## Option #1: Open in VS Code (Dev Container)

1. Install the "Dev Containers" extension in VS Code.
2. Open this repository in VS Code.
3. Use the Command Palette (Ctrl/Cmd+Shift+P) and choose "Dev Containers: Reopen in Container" (or click the green remote status bar and select "Reopen in Container").
4. Wait for the container to build and start; the integrated terminal in VS Code will be inside the devcontainer.

## Option #2: Open in GitHub Codespaces

1. On GitHub, open this repository page, click the green "Code" button -> "Open with Codespaces" -> "New codespace".
2. Or use the Codespaces extension in VS Code and create a new codespace from this repo.
3. The codespace starts with the repository and devcontainer environment ready to use.

## Creating the cluster

### Bootstrap (create cluster and install ArgoCD)

Run the bootstrap task from inside the devcontainer (or any environment with task/kind/kubectl available):

    task bootstrap

What this does (from Taskfile.yaml):

- create-cluster: kind create cluster --name gitops-demo --wait 5m
- install-argocd: deploys ArgoCD into the cluster and waits for argocd-server
- disable-argocd-auth: patches configmaps to allow anonymous admin access and restarts argocd-server
- apply-app-of-apps: kubectl apply -f app-of-apps.yaml

### After bootstrap, access the ArgoCD UI by running the port-forward task below

    task port-forward-argocd

This runs the Taskfile command: kubectl port-forward svc/argocd-server -n argocd 8080:80
Then open <http://localhost:8080> in your browser. To stop the forwarding, press Ctrl+C in the terminal running the task.

### Cleanup

To tear down the kind cluster and cleanup resources, run:

    task cleanup

This calls the Taskfile cleanup task which currently executes: task delete-cluster (which runs: kind delete cluster --name gitops-demo).

## Troubleshooting

- If the Argo CD forwarded port returns a 404 in your Github Codespace, you should go to "ports" -> right click 8080 port -> "Change port protocol" and change it to "HTTPS".
- Sometimes port-forward fails, when the argocd-server service is restarted. Re-run the "task port-forward-argocd" when that happens.
