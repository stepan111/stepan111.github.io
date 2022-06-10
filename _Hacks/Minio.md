---
layout: post
title:  "Deploy <b>Minio</b> to local Kind cluster with terraform"
categories: Hacks
---

# TBD


## Steps:
### 0. Create cluster as described [here](/Hacks/Create_kind_cluster.html)
### 1. Create `minio.tf`:
```
resource "helm_release" "minio_operator" {
  name              = "minio-operator"
  chart             = "operator"
  repository        = "https://operator.min.io/"
  create_namespace  = "true"
  namespace         = "minio-operator"
  dependency_update = "true"
  version           = "4.4.22"
}

```
### 2. Deploy
```
tf apply 
k create ns minio
```
### 3. Install minio kubectl plugin same version as helm chart:

```
wget https://github.com/minio/operator/releases/download/v4.4.22/kubectl-minio_4.4.22_linux_amd64 -O kubectl-minio
chmod +x kubectl-minio
mv kubectl-minio /usr/local/bin/
```

### 4. Create tenant in minio namespace through operator ui:
```
kubectl minio proxy -n minio-operator
```

### 5. Create bucket:
```
sudo kubefwd svc -c ~/.kube/config -n minio

```
and navigate to https://storage-console:9443 
