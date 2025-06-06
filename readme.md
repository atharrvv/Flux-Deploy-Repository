# FluxCD-Based Python Application Deployment

This repository demonstrates GitOps-based deployment of a containerized Python application using [FluxCD](https://fluxcd.io/) on Kubernetes. It includes automated image updates using Flux Image Update Automation and Git-based declarative configuration.

## 🔧 Components

### ✅ Kubernetes Manifests
- **Deployment**: Deploys a Python application with 3 replicas using rolling updates.
- **Service**: Exposes the app internally via `ClusterIP` on port 80 targeting container port 5000.

### 📦 Image Update Automation
The container image is managed via FluxCD Image Update Automation using the following annotation in the Deployment YAML:

```yaml
image: docker.io/eatherv/python-application:0.0.5
{"$imagepolicy": "flux-system:python"}
```

This ensures that Flux automatically pulls the latest tag matching the defined image policy (e.g., semver, latest, etc.) and updates the image in the Git repository.

### 📁 Namespace
All resources are deployed in the `app` namespace for isolation and better management.

---

## 🚀 How to Deploy with Flux

1. **Bootstrap Flux** into your Kubernetes cluster:
   ```bash
   flux bootstrap github      --owner=your-github-username      --repository=Flux-Deploy-Repository      --branch=main      --path=./clusters/dev
   ```

2. **Apply your image policy and automation:**
   ```bash
   flux create image repository python      --image=docker.io/eatherv/python-application      --interval=1m      --export > ./clusters/dev/image-repository.yaml

   flux create image policy python      --image-ref=flux-system:python      --select-semver=">=0.0.0"      --export > ./clusters/dev/image-policy.yaml
   ```

3. **Enable automation:**
   ```bash
   flux create image update automation python-update      --git-repo-ref=flux-system      --path="./clusters/dev"      --author-name=fluxcd      --author-email=flux@example.com      --commit-template="{{range .Updated.Images}}{{println .}}{{end}}"      --interval=2m
   ```

4. **Commit and push** the manifests to trigger deployment.

---

## 🧩 Folder Structure

```
Flux-Deploy-Repository/
│
├── clusters/
│   └── dev/
│       ├── deployment.yaml
│       ├── service.yaml
│       ├── image-repository.yaml
│       ├── image-policy.yaml
│       └── image-update-automation.yaml
```

---

## 📌 Notes
- The image policy is defined in the `flux-system` namespace.
- You must configure appropriate write access in your GitHub token for image automation to commit updated image tags.

---

## 🛡️ License

This project is licensed under the MIT License.
