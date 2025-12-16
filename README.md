# Kubernetes Manifests for the Node.js Demo

This repo contains only the Kubernetes configuration (manifests and deployment workflow) for the Node.js sample app whose source and image build live in `github-actions-kubernetes-1/`. Keeping manifests here separates cluster config from application code.

## How it connects to the app repo
- The app repo builds and pushes `docker.io/<user>/demo-app:latest` with GitHub Actions.
- This config repo supplies `deployment.yaml` and `service.yaml`. The app repo’s workflow checks out this repo and applies these files with `kubectl`.
- Update the `image:` field in `deployment.yaml` whenever the pushed image name or tag changes so deployments pull the correct version.

## Repository layout
- `deployment.yaml`: Deployment for the app (`hello-kubernetes`) with 3 replicas and port 3000. Default image set to `docker.io/arshattech/hello-k8s:latest` (change to match your pushed image, e.g., `docker.io/<user>/demo-app:latest`).
- `service.yaml`: LoadBalancer Service exposing the Deployment on port 80 → 3000.
- `.github/workflows/ci-deploy.yaml`: Optional workflow to stand up a temporary Kind cluster, load a prebuilt image, and apply the manifests (useful for smoke testing changes to the manifests).

## Using the manifests
- Apply to any kube-context that points to your cluster:
  ```bash
  kubectl apply -f deployment.yaml
  kubectl apply -f service.yaml
  ```
- Confirm rollout:
  ```bash
  kubectl get deploy hello-kubernetes
  kubectl get svc hello-kubernetes-service
  ```

## GitHub Actions workflow (ci-deploy.yaml)
- Trigger: push to `main`.
- Steps: checkout repo → install Kind → `docker pull` image → `kind load docker-image` → `kubectl apply` manifests.
- Adjust the image in both `deployment.yaml` and the `docker pull` step to the tag produced by your app repo’s pipeline (currently `docker.io/arshattech/demo-app:test` is pulled; align this with the Deployment image).

## New joiner checklist
- Get read access to both repos; ensure you know the image name/tag produced by the app pipeline.
- Keep image references in `deployment.yaml` and `.github/workflows/ci-deploy.yaml` in sync with what the app repo builds.
- Use `kubectl` against the intended cluster or Kind (for local) before applying.
- For production, supply cluster credentials to the runner that executes the app repo workflow; no secrets are required in this repo itself unless you add private registry pulls.

## References
- Kubernetes Deployments: https://kubernetes.io/docs/concepts/workloads/controllers/deployment/
- Kubernetes Services: https://kubernetes.io/docs/concepts/services-networking/service/
- Kind: https://kind.sigs.k8s.io
- GitHub Actions: https://docs.github.com/actions
