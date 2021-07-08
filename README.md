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
