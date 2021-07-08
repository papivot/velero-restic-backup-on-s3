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

2. Create a certificate/key pair for the server that will be running the Minio server and set up TLS communications

```shell
$ mkdir -p ${HOME}/.minio/certs
$ cp SERVER-NAME.crt ${HOME}/.minio/certs/public.crt
$ cp SERVER-NAME.key ${HOME}/.minio/certs/private.key
```

3. Start the Minio server  

$ mkdir -p ${HOME}/data
