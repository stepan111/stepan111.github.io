---
layout: post
title:  "Deploy Minio to local Kind cluster with terraform"
categories: Hacks
---

# TBD


## Steps:
### 0. Create cluster as described [here](/Terraform/Create_kind_cluster.html)
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

resource "random_password" "minio" {
  length = 16
}

resource "kubernetes_namespace" "tenant" {
  metadata {
    name = "block-storage"
  }
}

resource "kubernetes_secret" "minio_secret" {
  metadata {
    name      = "minio-secret"
    namespace = kubernetes_namespace.tenant.metadata.0.name

  }

  data = {
    accesskey = "admin"
    secretkey = random_password.minio.result
  }

  type = "Opaque"
}

resource "kubectl_manifest" "tenant" {
  yaml_body = file("${path.module}/minio-tenant.yaml")
  depends_on = [
    kubernetes_secret.minio_secret
  ]
}

resource "kubectl_manifest" "create_bucket" {
  yaml_body = file("${path.module}/minio-bucket.yaml")
  force_new = true
  wait      = true
  depends_on = [
    kubectl_manifest.tenant
  ]
}
```

and 
```
apiVersion: minio.min.io/v2
kind: Tenant
metadata:
  name: minio
  namespace: block-storage
spec:
  # configuration:
  #   name: storage-env-configuration
  credsSecret:
    name: minio-secret
  image: minio/minio:RELEASE.2022-06-07T00-33-41Z
  imagePullPolicy: IfNotPresent
  pools:
  - name: pool-0
    resources:
      requests:
        cpu: "1"
        memory: 2Gi
    servers: 1
    volumeClaimTemplate:
      metadata:
        name: data
      spec:
        accessModes:
        - ReadWriteOnce
        resources:
          requests:
            storage: "5Gi"
        storageClassName: standard
    volumesPerServer: 4
  requestAutoCert: false
  users:
  - name: storage-user-0

```
and
```
apiVersion: batch/v1
kind: Job
metadata:
  name: create-bucket
  namespace: block-storage
spec:
  template:
    spec:
      containers:
      - name: createbucket
        image: amazon/aws-cli
        command: ["aws"]
        args:
        -  s3api
        - create-bucket
        - --bucket
        - postgres
        - --endpoint-url
        - http://minio:80
        env:
        - name: AWS_ACCESS_KEY_ID
          valueFrom:
             secretKeyRef:
                name: minio-secret
                key: accesskey
          
        - name: AWS_SECRET_ACCESS_KEY
          valueFrom:
             secretKeyRef:
                name: minio-secret
                key: secretkey
          
      restartPolicy: Never
  backoffLimit: 1
```
### 2. Deploy
```
tf apply 

```
### 3. Optional. Install minio kubectl plugin same version as helm chart:

```
wget https://github.com/minio/operator/releases/download/v4.4.22/kubectl-minio_4.4.22_linux_amd64 -O kubectl-minio
chmod +x kubectl-minio
mv kubectl-minio /usr/local/bin/
```

Get operator UI operator ui:
```
kubectl minio proxy -n minio-operator
```

### 4. Optional. Check minio-console:
```
k get secret minio-secret -o jsonpath={.data.accesskey} | base64 -d && echo
k get secret minio-secret -o jsonpath={.data.secretkey} | base64 -d && echo
sudo kubefwd svc -c ~/.kube/config -n block-storage

```
and navigate to https://minio-console.block-storage:9443 
