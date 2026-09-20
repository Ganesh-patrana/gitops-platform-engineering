This repository defines a self-serve, zero-touch Internal Developer Platform (IDP) built on Google Kubernetes Engine (GKE). It is architected specifically to support modern machine learning pipelines and Retrieval-Augmented Generation (RAG) workloads using a strict GitOps methodology.

## 🏗️ Architecture Stack

* **Control Plane (GitOps):** ArgoCD
* **Infrastructure Engine:** Crossplane (GCP Provider)
* **Observability:** Prometheus & Grafana (`kube-prometheus-stack`)
* **AI Infrastructure:** Qdrant Vector Database
* **Cloud Provider:** Google Cloud Platform (GCP)

## 📂 Repository Structure (App-of-Apps Pattern)

The platform relies on the ArgoCD App-of-Apps pattern to strictly decouple platform bootstrapping, cloud infrastructure provisioning, and application deployment.

```text
├── root-app.yaml                  # Core ArgoCD Bootstrap Application
├── bootstrap/                     # Cluster-level dependency watchers
│   ├── crossplane.yaml            # Crossplane Control Plane
│   ├── infrastructure-app.yaml    # Watcher for cloud infrastructure
│   └── apps-app.yaml              # Watcher for Kubernetes workloads
├── infrastructure/                
│   └── crossplane/                # GCP Cloud Resources (Storage Buckets, etc.)
└── apps/                          
    ├── observability.yaml         # Prometheus/Grafana Stack
    └── qdrant.yaml                # AI Vector Database + ServiceMonitors
🚀 Bootstrapping the Cluster
To recreate this entire platform on a fresh GKE cluster, no imperative Helm or Terraform commands are required. The entire stack is reconciled from Git.

Install the ArgoCD core:

Bash
kubectl create namespace argocd
kubectl apply -n argocd -f [https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml](https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml)
Apply the Root Application:

Bash
kubectl apply -f root-app.yaml
Retrieve Dashboard Credentials:

Bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
kubectl port-forward svc/argocd-server -n argocd 8080:443
🧠 Engineering Notes: Crossplane CRD Race Conditions
When bootstrapping Crossplane via ArgoCD, the ProviderConfig relies on Custom Resource Definitions (CRDs) installed by the Provider. Because ArgoCD aggressively dry-runs the entire manifest tree before applying, this creates a race condition where the sync fails due to unrecognized APIs (e.g., gcp.upbound.io/v1beta1).

Solution Implemented: A GitOps Two-Step Commit. The Provider is deployed first to successfully register the CRDs with the Kubernetes API server, followed by the ProviderConfig injection.
