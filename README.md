# Cluster Config (GitOps with Argo CD)

This repository holds Kubernetes configuration for applications deployed via Argo CD.

## Structure
- `apps/<app>/base`: raw Kubernetes manifests (deployments, services)
- `apps/<app>/overlays/prod`: Kustomize overlay (namespace, image tags)
- `argocd/applications`: Argo CD `Application` resources pointing to overlays

## Bootstrap
1. Install Argo CD in the cluster (once):
   ```bash
   kubectl create ns argocd
   kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
   ```
2. Apply Applications:
   ```bash
   kubectl apply -n argocd -k argocd/applications
   ```
3. Argo CD will sync apps in `apps/<app>/overlays/prod`.

## CI Flow (in app repos)
- Build/push images to Docker Hub with tags `latest` and `${GITHUB_SHA}`.
- Update image tags in the overlay using Kustomize and commit here:
  ```bash
  cd apps/<app>/overlays/prod
  kustomize edit set image your-dockerhub-username/<app>-backend=${DOCKERHUB_USERNAME}/<app>-backend:${GITHUB_SHA}
  kustomize edit set image your-dockerhub-username/<app>-frontend=${DOCKERHUB_USERNAME}/<app>-frontend:${GITHUB_SHA}
  ```
- Argo CD reconciles changes and deploys to the cluster.

