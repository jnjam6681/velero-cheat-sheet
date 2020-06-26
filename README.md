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

--------------------------------------------------------------------------------

## Velero cheat sheet

Command                                                                                                 | Description
------------------------------------------------------------------------------------------------------- | -----------
kubectl get ns                                                                                          | Get all namespaces
kubectl get all -n velero                                                                               | Get all resources by namespace velero
kubectl create namespace testing                                                                        | Create a new namespace
kubectl create deployment nginx --image=nginx -n testing                                                | Create a new deployment by namespace
kubectl scale deploy nginx -n testing --replicas=2                                                      | Scaling a deployment
kubectl get backup -n velero                                                                            | Get backup in kubernetes
kubectl get crds -n velero                                                                              | Get crds in kubernetes
velero help                                                                                             | Get help 
velero help bucket                                                                                      | Get help for bucket
velero backup-location get                                                                              | Get a backup storage locations 
velero backup get                                                                                       | Get all backup
velero backup create nginx-backup --include-namespaces testing                                          | Create a backup with specify namespaces
velero backup create nginx-backup --exclude-resources pods                                              | Create a backup with exclude resources
velero backup create nginx-backup --include-namespaces testing --include-resources pods, deployment     | Create a backup with specify namespaces and resources
velero backup create nginx-backup --include-namespaces testing --exclude-resources pods                 | Create a backup with specify namespaces and exclude resources
velero backup create nginx-backup --include-namespaces testing --ttl 2h                                 | Create a backup and set expire time
velero backup describe nginx-backup | less                                                              | Describe a backup
velero restore get                                                                                      | Get all restore
velero restore create nginx-backup-restore --from-backup nginx-backup                                   | Create a restore from backup
velero restore describe nginx-backup-restore | less                                                     | Describe a restore
velero restore delete nginx-backup-restore                                                              | Delete a restore
velero restore delete --all                                                                             | Delete a restore
velero backup delete nginx-backup                                                                       | Delete a backup
velero backup delete --all                                                                              | Delete a backup
velero schedule get                                                                                     | Get all schedule
velero schedule create nginx-schedule --schedule="* * * * *"                                            | Create a backup with schedule
velero schedule create nginx-schedule --schedule="@every 1m" --include-namespaces testing               | Create a backup with schedule and include namespaces
velero schedule delete --all                                                                            | Delete all schedules
