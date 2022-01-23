# Build Container Infra
Use container-base application to build cloud infrastructure. In other words, you can use kubernetes to operation Database, Streaming, Object Store, API Gateway ...etc.
```
wudajun@controller:~$ kubectl get no
NAME         STATUS   ROLES                  AGE   VERSION
controller   Ready    control-plane,master   21h   v1.23.2
worker-0     Ready    <none>                 21h   v1.23.2
worker-1     Ready    <none>                 21h   v1.23.2
worker-2     Ready    <none>                 21h   v1.23.2
```

## Pre-require
* Kubenetes Cluster Setup: 1 Controller + 3 Worker
* Compute Resource Recommand
  * CPU: 64 vCPU 
  * Memory: 128 GB
* kubectl Installed

## Object Store
I choose MinIO. MinIO offers high-performance, S3 compatible object storage and native to Kubernetes.

### Install krew first
```
(
  set -x; cd "$(mktemp -d)" &&
  OS="$(uname | tr '[:upper:]' '[:lower:]')" &&
  ARCH="$(uname -m | sed -e 's/x86_64/amd64/' -e 's/\(arm\)\(64\)\?.*/\1\2/' -e 's/aarch64$/arm64/')" &&
  KREW="krew-${OS}_${ARCH}" &&
  curl -fsSLO "https://github.com/kubernetes-sigs/krew/releases/latest/download/${KREW}.tar.gz" &&
  tar zxvf "${KREW}.tar.gz" &&
  ./"${KREW}" install krew
)

export PATH="${KREW_ROOT:-$HOME/.krew}/bin:$PATH"
```
### Install MinIO and initialize
```
kubectl krew install minio
kubectl minio init
```

### Create tenant
Check args before you run this command!
```
kubectl create ns minio-instance

kubectl minio tenant create minio-k8s-lab \
     --servers 4                          \
     --volumes 4                          \
     --capacity 10Gi                      \
     --namespace minio-instance           \
     --storage-class local-storage        \
```
### Verify instance
```
Tenant 'minio-k8s-lab' created in 'minio-instance' Namespace

  Username: admin
  Password: 6c44b148-bf4c-4d4d-b577-c36016fe1520
  Note: Copy the credentials to a secure location. MinIO will not display these again.

+-------------+-----------------------+----------------+--------------+--------------+
| APPLICATION | SERVICE NAME          | NAMESPACE      | SERVICE TYPE | SERVICE PORT |
+-------------+-----------------------+----------------+--------------+--------------+
| MinIO       | minio                 | minio-instance | ClusterIP    | 443          |
| Console     | minio-k8s-lab-console | minio-instance | ClusterIP    | 9443         |
+-------------+-----------------------+----------------+--------------+--------------+
```

## Reference
* [MinIO Kubernetes Install](https://github.com/minio/operator/blob/v4.0.10/README.md)

## Database
I choose CockroachDB, YugabyteDB.

## Streaming/Message Queue
I choose NATS/JetStream


## API Gateway

## Service Mesh
I choose Linkerd, Istio.



