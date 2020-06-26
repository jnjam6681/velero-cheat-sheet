# Velero

Backup and migrate Kubernetes resources and persistent volumes

--------------------------------------------------------------------------------

## Requirement

hostname | IP Address
-------- | -------------
microk8s | 192.168.33.30
minio    | 192.168.33.20

## Install Minio object storage

```
docker run --name minio -d -p 9000:9000 \
  -v data:/data \
  -e "MINIO_ACCESS_KEY=AKIAIOSFODNN7EXAMPLE" \
  -e "MINIO_SECRET_KEY=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY" \
  minio/minio server /data
```

If you can't user docker, just create the docker group and add your user.

```
sudo groupadd docker
sudo usermod -aG docker $USER
```

--------------------------------------------------------------------------------

## Install Velero client

```
wget https://github.com/vmware-tanzu/velero/releases/download/v1.4.0/velero-v1.4.0-linux-amd64.tar.gz
tar zxf  velero-v1.4.0-linux-amd64.tar.gz
sudo mv velero-v1.4.0-linux-amd64/velero /usr/local/bin/
```

#### Configuring Secrets

```
cat <<EOF>> minio.credentials
[default]
aws_access_key_id=AKIAIOSFODNN7EXAMPLE
aws_secret_access_key=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
EOF
`
```

If you can't use kubectl.

```
kubectl config view --raw > $HOME/.kube/config
```

#### Install Velero server and connect to Minio object storage

```
velero install \
  --provider aws \
  --plugins velero/velero-plugin-for-aws:v1.0.0 \
  --bucket kubedemo\
  --secret-file ./minio.credentials \
  --backup-location-config s3Url=http://192.168.33.20:9000,region=minio,s3ForcePathStyle=true
```

--------------------------------------------------------------------------------

## Basic commands for backup and restore

#### backup resources

```
velero backup create system-backup --include-namespaces kube-system
```

#### get all backup

```
velero backup get
```

#### restore resources from backup

```
velero restore create restore-system --from-backup system-backup
```

#### get all restore

```
velero restore get
```
