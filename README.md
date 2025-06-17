## Requriments - Installation

- Docker
    
    ```bash
    apt update
    apt install docker.io
    ```
    
- Kubectl
    
    Documentation: https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/
    
    ```bash
    curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
    curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"
    echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check
    install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
    chmod +x kubectl
    mkdir -p ~/.local/bin
    mv ./kubectl ~/.local/bin/kubectl
    # and then append (or prepend) ~/.local/bin to $PATH
    ```
    
- Flux CLI
    
    Documentation: https://fluxcd.io/flux/installation/#install-the-flux-cli
    
    ```bash
    curl -s https://fluxcd.io/install.sh | sudo bash
    . <(flux completion bash)
    ```
    
- Kind Cluster
    
    Documentation: https://kind.sigs.k8s.io/docs/user/quick-start/
    
    ```bash
    [ $(uname -m) = x86_64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.27.0/kind-linux-amd64
    chmod +x ./kind
    mv ./kind /usr/local/bin/kind
    ```
    

## kind cluster set-up

- config.yml

```yaml
# a cluster with 3 control-plane nodes and 3 workers
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
- role: worker
- role: worker
```

```yaml
kind create cluster --config=config.yml
```

- output

```bash
Creating cluster "kind" ...
 ✓ Ensuring node image (kindest/node:v1.32.2) 🖼
 ✓ Preparing nodes 📦 📦 📦
 ✓ Writing configuration 📜
 ✓ Starting control-plane 🕹️
 ✓ Installing CNI 🔌
 ✓ Installing StorageClass 💾
 ✓ Joining worker nodes 🚜
Set kubectl context to "kind-kind"
You can now use your cluster with:

kubectl cluster-info --context kind-kind

Have a question, bug, or feature request? Let us know! https://kind.sigs.k8s.io/#community 🙂
```

- Cluster-info

```bash
root@ip:/home/ubuntu# kubectl  cluster-info
Kubernetes control plane is running at https://127.0.0.1:41995
CoreDNS is running at https://127.0.0.1:41995/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.

root@ip:/home/ubuntu# kubectl get nodes
NAME                 STATUS   ROLES           AGE    VERSION
kind-control-plane   Ready    control-plane   2m4s   v1.32.2
kind-worker          Ready    <none>          113s   v1.32.2
kind-worker2         Ready    <none>          113s   v1.32.2
```

## Flux - set-up

- Export GitHub Credentials

```bash
export GITHUB_TOKEN=<your-token>
export GITHUB_USER=<your-username>
```

- Cluster check

```bash
flux check --pre
► checking prerequisites
✔ Kubernetes 1.32.2 >=1.30.0-0
✔ prerequisites checks passed
```

## Flux Way

- GitHub Repository
    - The source for Manifests and source files
- Kustomization
    - By using the git source, it will install, add and configure the cluster

### While installing the Flux, it uses the same concept GitHub Repository and Kustomization

- Bootstrapping
    - It will create the GitHub Repository with all resources that needed for Flux on Cluster
    - And that created GitHub repository will have the Manifest files needed for Flux to install on cluster. Like source-controller, kustomization-controller, helm-controller, notification-controller
        - GitHub Repository
        - Kustomization
    - It will create Flux related resources under the namespace of “flux-system”
- Flux Bootstrapping

```bash
  flux bootstrap github \
  --owner=$GITHUB_USER \
  --repository=flux-fleet-infra \
  --branch=main \
  --path=./clusters/my-cluster \
  --personal
```

- Output

```bash
► connecting to github.com
✔ repository "https://github.com/atharrvv/flux-python-app-infra" created
► cloning branch "main" from Git repository "https://github.com/atharrvv/flux-python-app-infra.git"
✔ cloned repository
► generating component manifests
✔ generated component manifests
✔ committed component manifests to "main" ("cf56fbd649aadfb0cd80d8185165ac7130185d4d")
► pushing component manifests to "https://github.com/atharrvv/flux-python-app-infra.git"
► installing components in "flux-system" namespace
✔ installed components
✔ reconciled components
► determining if source secret "flux-system/flux-system" exists
► generating source secret
✔ public key: ecdsa-sha2-nistp384 AAAAE2VjZHNhLXNoYTItbmlzdHAzODQAAAAIbmlzdHAzODQAAABhBLY/wJjDzfXaONnGsAdWEBU0HjAZqu1MmSD5P5BVMnFSXWnTQROxMd6Yh5ayixHZ5lyAIa1BF3bTlSAw9eYi9j3ngXz0BhSH3zEBQ8EEwLGwZxDmb4vRNb/Yr6BAHnbjWg==
✔ configured deploy key "flux-system-main-flux-system-./clusters/my-cluster" for "https://github.com/atharrvv/flux-python-app-infra"
► applying source secret "flux-system/flux-system"
✔ reconciled source secret
► generating sync manifests
✔ generated sync manifests
✔ committed sync manifests to "main" ("fa78e53459fb661291ca32726c712fb5e310c698")
► pushing sync manifests to "https://github.com/atharrvv/flux-python-app-infra.git"
► applying sync manifests
✔ reconciled sync configuration
◎ waiting for GitRepository "flux-system/flux-system" to be reconciled
✔ GitRepository reconciled successfully
◎ waiting for Kustomization "flux-system/flux-system" to be reconciled
✔ Kustomization reconciled successfully
► confirming components are healthy
✔ helm-controller: deployment ready
✔ kustomize-controller: deployment ready
✔ notification-controller: deployment ready
✔ source-controller: deployment ready
✔ all components are healthy
```

### Everything related to Flux, created in “flux-system” namespace

```bash
root@ip:/home/ubuntu# kubectl get all -n flux-system

NAME                                           READY   STATUS    RESTARTS   AGE
pod/helm-controller-b6767d66-mq2pz             1/1     Running   0          6m44s
pod/kustomize-controller-57c7ff5596-dzhn7      1/1     Running   0          6m44s
pod/notification-controller-58ffd586f7-p9n7s   1/1     Running   0          6m44s
pod/source-controller-6ff87cb475-c78pr         1/1     Running   0          6m44s

NAME                              TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
service/notification-controller   ClusterIP   10.96.157.79   <none>        80/TCP    6m45s
service/source-controller         ClusterIP   10.96.70.200   <none>        80/TCP    6m45s
service/webhook-receiver          ClusterIP   10.96.230.55   <none>        80/TCP    6m44s

NAME                                      READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/helm-controller           1/1     1            1           6m44s
deployment.apps/kustomize-controller      1/1     1            1           6m44s
deployment.apps/notification-controller   1/1     1            1           6m44s
deployment.apps/source-controller         1/1     1            1           6m44s

NAME                                                 DESIRED   CURRENT   READY   AGE
replicaset.apps/helm-controller-b6767d66             1         1         1       6m44s
replicaset.apps/kustomize-controller-57c7ff5596      1         1         1       6m44s
replicaset.apps/notification-controller-58ffd586f7   1         1         1       6m44s
replicaset.apps/source-controller-6ff87cb475         1         1         1       6m44s
```

```bash
root@ip:/home/ubuntu# kubectl get gitrepo -n flux-system
NAME          URL                                                   AGE     READY   STATUS
flux-system   ssh://git@github.com/atharrvv/flux-python-app-infra   2m42s   True    stored artifact for revision 'main@sha1:fa78e53459fb661291ca32726c712fb5e310c698'

root@ip:/home/ubuntu# kubectl get kustomization -n flux-system
NAME          AGE    READY   STATUS
flux-system   3m9s   True    Applied revision: main@sha1:fa78e53459fb661291ca32726c712fb5e310c698
```

### Add the source GitHub Repository to Flux - Imperative Way

- fork this repository: https://github.com/atharrvv/Flux-Deploy-Repository.git

```bash
flux create source git python \
    --url=https://github.com/atharrvv/Flux-Deploy-Repository.git \
    --branch=main
```

- output:

```bash
✚ generating GitRepository source
► applying GitRepository source
✔ GitRepository source created
◎ waiting for GitRepository source reconciliation
✔ GitRepository source reconciliation completed
✔ fetched revision: main@sha1:0cce17779714de6b6311168fe858506db5ba5eea
```

### Add the source GitHub Repository to Flux - Declarative Way

- This will export command to YAML

```bash
flux create source git python \
    --url=https://github.com/atharrvv/Flux-Deploy-Repository.git \
    --branch=main \
    --export > ./python-github-repository
```

- output YAML:  python-github-repository.yml

```yaml
---
apiVersion: source.toolkit.fluxcd.io/v1
kind: GitRepository
metadata:
  name: python
  namespace: flux-system
spec:
  interval: 1m0s
  ref:
    branch: main
  url: https://github.com/atharrvv/Flux-Deploy-Repository.git
```

### Deploy the application

- First create namespace

```bash
kubectl create ns app
```

### Create a Kustomization - Imperative Way

```bash
flux create kustomization python \
  --target-namespace=app \
  --source=python \
  --path="./" \
  --interval=5m
```

- output:

```bash
✚ generating Kustomization
► applying Kustomization
✔ Kustomization created
◎ waiting for Kustomization reconciliation
✔ Kustomization python is ready
✔ applied revision main@sha1:0cce17779714de6b6311168fe858506db5ba5eea
```

### Create a Kustomization - Declarative Way

- This will export command to YAML

```yaml
   flux create kustomization python \
  --target-namespace=app \
  --source=python \
  --path="./" \
  --interval=5m \
  --export > ./python-kustomization.yml
```

- output YAML: python-kustomization.yml

```yaml
---
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: python
  namespace: flux-system
spec:
  interval: 5m0s
  path: ./
  prune: false
  sourceRef:
    kind: GitRepository
    name: python
  targetNamespace: app
```

- Application

```bash
root@ip:/home/ubuntu# kubectl get pods -n app

NAME                                 READY   STATUS    RESTARTS   AGE
python-application-cc9d4f4fc-9kqz4   1/1     Running   0          2m2s
python-application-cc9d4f4fc-xm786   1/1     Running   0          2m2s
```

### **Now, make any change in manifest (/deploy/manifests) file**,

## flux should react! ☸️

# Alerts set-up

- **Provider**
    - This will define which provider will be used. Ex.Slack, Microsoft Teams, Discord and more
- Alerts
    - This will set the alerts parameterts, for more: https://fluxcd.io/flux/monitoring/alerts/

### Creating a Webhook URL

- click on Create New App

![image.png](attachment:12505e25-5e70-4e9c-90a3-cf499b6044a6:422dfc6b-f910-4992-a90b-b7812b5e9372.png)

- Select from scratch

![image.png](attachment:134ea432-57d6-47e0-9e4f-c5917e94f6e7:1ff18eed-ec57-4a02-b04f-2753d19a0577.png)

- Name the app and create

![image.png](attachment:a84fcac7-79a6-44e0-8626-cb3213e5bd70:f5d21a8b-c7fd-4788-bce3-c3b66707037c.png)

- Select Incomming Webhook

![image.png](attachment:0d180c98-079d-45d2-b212-7fffa3091a23:1a40fa6c-50ee-4c0f-8425-5446671ea814.png)

- Enable Wenhook

![image.png](attachment:306a0d38-c6ba-4491-8300-d28c6d5e9d7e:d568ecdf-93dd-4339-aa95-1567ecd8168d.png)

- Select add new webhook to workplace

![image.png](attachment:89731c41-6f5f-4b1e-ba7c-858a3b52f2e6:6d7246c5-5f78-4753-88d1-82956927be80.png)

- Select channel you want have notification on and click allow

![image.png](attachment:55bf0283-dfe0-4471-983a-7f76ead1efe4:50c05093-e33c-435d-b88c-c4d25465227b.png)

- Confirm

![image.png](attachment:30f031ad-c0fd-4168-aff1-4b08fc030028:d2e5e32d-d2a8-4409-899a-22e7e2eab9a5.png)

![image.png](attachment:08428606-1f30-4866-8095-8d7949ffa6c2:2e681ea1-2695-42ae-b0ac-dfd17455cbb1.png)

### Adding webhook URL to K8S secrets

```bash
kubectl -n flux-system create secret generic slack-url \
--from-literal=address={webhook_URL}
```

### Provider.yml

```bash
apiVersion: notification.toolkit.fluxcd.io/v1beta3
kind: Provider
metadata:
  name: slack
  namespace: flux-system
spec:
  type: slack
  channel: general
  secretRef:
    name: slack-url
```

### notification.yml

```bash
apiVersion: notification.toolkit.fluxcd.io/v1beta3
kind: Alert
metadata:
  name: on-call-webapp
  namespace: flux-system
spec:
  providerRef:
    name: slack
  eventSeverity: info
  eventSources:
    - kind: GitRepository
      name: '*'
    - kind: Kustomization
      name: '*'
    - kind: HelmRelease
      name: '*'
```

## Now, Make any change in GitHub Repository, Kustomization and Helm Release

### Got notified!

![image.png](attachment:2829655f-be9f-4c23-a30b-181e446c8c74:79eb152e-0bae-43f2-8713-1a0cb6742397.png)
