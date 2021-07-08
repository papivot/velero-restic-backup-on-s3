# Leveraging Restic backup and Velero on a S3 compatiable storage
---
### Setting up Minio as a S3 ompatiable storage
1. Download and configure Minio

```shell
$ wget https://dl.min.io/client/mc/release/linux-amd64/mc
$ wget https://dl.min.io/server/minio/release/linux-amd64/minio
$ chmod +x mc
$ chmod +x minio
$ sudo mv mc /usr/local/bin
$ sudo mv minio /usr/local/bin
```

2. Create a certificate/key pair for the server/VM that will be running Minio and set up TLS communications

```shell
$ mkdir -p ${HOME}/.minio/certs
$ cp SERVER-NAME.crt ${HOME}/.minio/certs/public.crt
$ cp SERVER-NAME.key ${HOME}/.minio/certs/private.key
```

3. Start the Minio server  

```shell
$ mkdir -p ${HOME}/data
$ minio server ${HOME}/data

No credential environment variables defined. Going with the defaults.
It is strongly recommended to define your own credentials via environment variables MINIO_ROOT_USER and MINIO_ROOT_PASSWORD instead of using default values
...
Endpoint: https://10.197.107.61:9000  
RootUser: minioadmin
RootPass: minioadmin

Browser Access:
   https://10.197.107.61:9000

Command-line Access: https://docs.min.io/docs/minio-client-quickstart-guide
   $ mc alias set myminio https://10.197.107.61:9000 minioadmin minioadmin
...
Certificate:
    Signature Algorithm: SHA512-RSA
    Issuer: C=US, ST=VA, UnknownOID=2.5.4.7, O=SE, OU=Personal, CN=ubuntu-nv-062.env1.lab.test
    Validity
        Not Before: Thu, 08 Jul 2021 05:56:06 GMT
        Not After : Sun, 06 Jul 2031 05:56:06 GMT

Detected default credentials 'minioadmin:minioadmin', please change the credentials immediately by setting 'MINIO_ROOT_USER' and 'MINIO_ROOT_PASSWORD' environment values
IAM initialization complete
```

4. Create a bucket called `test`
```shell 
$ mc alias set local https://10.197.107.61:9000 minioadmin minioadmin
$ mc mb local/test
```

### Enable VELERO on the Kubernetes cluster

1. The easiest way to achieve this is to attach the cluster to TMC and enable data protection. This installed the necessary velero components on the cluster. 
```shell
$ kubectl get all -n velero

NAME                         READY   STATUS    RESTARTS   AGE
pod/restic-qdbsh             1/1     Running   0          13h
pod/restic-r6lfl             1/1     Running   0          13h
pod/velero-67554fd54-sk8ml   1/1     Running   0          13h

NAME                    DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
daemonset.apps/restic   2         2         2       2            2           <none>          13h

NAME                     READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/velero   1/1     1            1           13h

NAME                               DESIRED   CURRENT   READY   AGE
replicaset.apps/velero-67554fd54   1         1         1       13h
```

2. Fix velero install to trust root cert of Minio server. 
```shell
$ cat MINIO_SERVER-ROOT-CA.crt|base64 -w0;echo
$ kubectl get backupstoragelocations -n velero
$ kubectl edit backupstoragelocations -n velero LOCATION_DETAIL
#### Edit the spec section and add a field caCert with the BASE64 encoded value of the ROOT CA CERT of MINIO server

spec:
  config:
    bucket: test
    profile: nverma-minio1
    publicUrl: https://10.197.107.61:9000
    region: minio
    s3ForcePathStyle: "true"
    s3Url: https://10.197.107.61:9000
  objectStorage:
    bucket: test
    caCert: LS0tLS1CRUdJTiB........S9XM3NmQVU9Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K
    prefix: 01F9FNY05068P4KSCM15M26S9G/
  provider: aws
```

3. For an internet restricted env, copy the restic image to a local registry and create a `restic-restore-action-config` ConfigMap for the velero install
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: restic-restore-action-config
  namespace: velero
  labels:
    velero.io/plugin-config: ""
    velero.io/restic: RestoreItemAction
data:
  image: harbor-repo.vmware.com/navneetv/velero-restic-restore-helper:v1.4.2
  cpuRequest: 200m
  memRequest: 128Mi
  cpuLimit: 200m
  memLimit: 128Mi
```



