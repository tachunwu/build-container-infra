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
* Compute Resource Recommend
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

### Install Storage Provider
MinIO recommend use their CSI driver, but their driver not work on my kubernetes version. So I use rancher's local path provisioner.
```
kubectl apply -f https://raw.githubusercontent.com/rancher/local-path-provisioner/master/deploy/local-path-storage.yaml
```
### Verify Storage Provider
```
wudajun@controller:~$  kubectl -n local-path-storage get pod
NAME                                      READY   STATUS    RESTARTS   AGE
local-path-provisioner-566b877b9c-j8skv   1/1     Running   0          54s
```

### Create tenant
Check args before you run this command!
```
kubectl minio tenant create minio-k8s-lab \
     --servers 1                          \
     --volumes 4                          \
     --capacity 10Gi                      \
     --namespace default                  \
     --storage-class local-path           \
```
### Verify instance
```
Tenant 'minio-k8s-lab' created in 'default' Namespace

  Username: admin
  Password: c46eeda8-c528-4c7c-9afe-e5d6b18744b1
  Note: Copy the credentials to a secure location. MinIO will not display these again.

+-------------+-----------------------+-----------+--------------+--------------+
| APPLICATION | SERVICE NAME          | NAMESPACE | SERVICE TYPE | SERVICE PORT |
+-------------+-----------------------+-----------+--------------+--------------+
| MinIO       | minio                 | default   | ClusterIP    | 443          |
| Console     | minio-k8s-lab-console | default   | ClusterIP    | 9443         |
+-------------+-----------------------+-----------+--------------+--------------+
```
### Verify pods, pvc, pv
```
wudajun@controller:~$ kubectl get pvc
NAME                     STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
0-minio-k8s-lab-ss-0-0   Bound    pvc-0c3d3bd1-4b28-4f33-a148-e0c166dc1830   2560Mi     RWO            local-path     31s
1-minio-k8s-lab-ss-0-0   Bound    pvc-d2e26f6b-9cc0-40a1-9a87-0301ed6df1bc   2560Mi     RWO            local-path     31s
2-minio-k8s-lab-ss-0-0   Bound    pvc-586e502a-0d74-4c95-94da-79a5ce41c1bf   2560Mi     RWO            local-path     31s
3-minio-k8s-lab-ss-0-0   Bound    pvc-ae51b62e-c95b-42a6-860c-341e7229ea42   2560Mi     RWO            local-path     31s
wudajun@controller:~$ kubectl get po
NAME                   READY   STATUS    RESTARTS   AGE
minio-k8s-lab-ss-0-0   1/1     Running   0          40s
wudajun@controller:~$ kubectl get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                            STORAGECLASS   REASON   AGE
pvc-0c3d3bd1-4b28-4f33-a148-e0c166dc1830   2560Mi     RWO            Delete           Bound    default/0-minio-k8s-lab-ss-0-0   local-path              114s
pvc-586e502a-0d74-4c95-94da-79a5ce41c1bf   2560Mi     RWO            Delete           Bound    default/2-minio-k8s-lab-ss-0-0   local-path              119s
pvc-ae51b62e-c95b-42a6-860c-341e7229ea42   2560Mi     RWO            Delete           Bound    default/3-minio-k8s-lab-ss-0-0   local-path              117s
pvc-d2e26f6b-9cc0-40a1-9a87-0301ed6df1bc   2560Mi     RWO            Delete           Bound    default/1-minio-k8s-lab-ss-0-0   local-path              2m1s
```

## Reference
* [MinIO Kubernetes Install](https://github.com/minio/operator/blob/v4.0.10/README.md)
* [How MinIO Brings Object Storage Service to Kubernetes](https://thenewstack.io/how-minio-brings-object-storage-service-to-kubernetes/)
* [Local Path Provisioner](https://github.com/rancher/local-path-provisioner)

## Database
I choose CockroachDB, YugabyteDB.

### Install CockraochDB CRD
```
kubectl apply -f https://raw.githubusercontent.com/cockroachdb/cockroach-operator/v2.4.0/install/crds.yaml
```
### Install CockraochDB Operator
```
kubectl apply -f https://raw.githubusercontent.com/cockroachdb/cockroach-operator/v2.4.0/install/operator.yaml
```
### Verify operator
```
kubectl get pods
```
### Download cluster yaml
```
curl -O https://raw.githubusercontent.com/cockroachdb/cockroach-operator/v2.4.0/examples/example.yaml
```
### Install CockroachDB Cluster
```
kubectl apply -f example.yaml
```

## Reference
* [Configure Cockroachdb Kubernetes](https://www.cockroachlabs.com/docs/v21.2/configure-cockroachdb-kubernetes?filters=manual)

## Streaming/Message Queue
I choose NATS/JetStream


## API Gateway
I choose Emissary-Ingress.

## Reference
* [Emissary](https://www.getambassador.io/docs/emissary/)

## Service Mesh
I choose Linkerd, Istio.



