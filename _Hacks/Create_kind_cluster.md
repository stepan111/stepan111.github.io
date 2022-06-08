---
layout: post
title:  "Create kind cluster with insecure registry inside"
categories: Hacks
---

> Quite often it is usful to have local development k8s cluster with anonimous insecure registry. 

## Steps:
### 0. Optiional step - create `.gitignore` file:
```
.terraform*
terraform.tfstate*
!.terraform.lock.hcl
ssh-key*
kubeconfig
istio-*
tmp/
tmp-*/
.kube/
.cache/
.ssh/
.ansible/
.passwd
__pycache__
```

### 1.  In the root directory create `kind.yaml` with next content:

```
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
containerdConfigPatches:
- |-
  [plugins."io.containerd.grpc.v1.cri".registry.mirrors."registry-docker-registry.registry:5000"]
    endpoint = ["http://localhost:30000"]
```

### 2. Create cluster

```
kind create cluster --config ./kind.yaml --name cluster

kubectl cluster-info --context kind-cluster # if required to switch context

```
### 3. Create terraform `provider.tf`:
```
provider "helm" {
  kubernetes {
    config_context = "kind-cluster"
    config_path    = "~/.kube/config"
  }

}


provider "kubernetes" {
  config_context = "kind-cluster"
  config_path    = "~/.kube/config"
}

```
### and `registry.tf` :
```
resource "helm_release" "registry" {
  name              = "registry"
  chart             = "docker-registry"
  repository        = "https://helm.twun.io"
  namespace         = "registry"
  dependency_update = "true"
  create_namespace  = "true"
  set {
    name  = "service.type"
    value = "NodePort"
  }
  set {
    name  = "service.nodePort"
    value = "30000"
  }

}

```

### 4. Apply terraform:
```
alias tf=terraform
tf init
tf apply
```
### 5. Update podman config (Using podman because it is default for ubuntu nowadays). Create `$HOME/.config/containers/registries.conf` with next content:
```
[[registry]]
location="registry-docker-registry.registry:5000"
insecure=truepodman 
```


### 6. Verify it's working:

```
alias k=kubectl

k get po -n registry
sudo kubefwd svc -c ~/.kube/config -n registry

```
```
# In another terminal
podman pull nginx
podman tag nginx registry-docker-registry.registry:5000/nginx:latest
podman push registry-docker-registry.registry:5000/nginx:latest

k run local-nginx --image=registry-docker-registry.registry:5000/nginx:latest -n default
k get pods -n default  -o=custom-columns='NAME:metadata.name,STATUS:status.phase,IMAGE:spec.containers[*].image'

```


# Profit !