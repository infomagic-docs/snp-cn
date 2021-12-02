---
title: 使用元数据工具备份和恢复
id: backup-restore-metadata-tool
category: operator-guides
---

StreamNative Platform 是具有多层架构的分布式消息系统，使用 Apache ZooKeeper 作为元数据服务。StreamNative Platform 存储层和 ZooKeeper 都提供了多个数据副本。为了进一步提高数据容错性，防止数据丢失，可以用元数据工具进行备份和恢复。目前该工具可以将 Pulsar 集群的元数据备份到云存储或从云存储中恢复元数据。

 

# 备份 ZooKeeper 元数据

默认情况下，备份服务是禁用的。要启用备份服务，需要将 `backup` 设置为  `true`，在需要部署集群的 `values.yaml` 文件中对 bucket 和 secret 进行相应的设置。

按照如下步骤备份集群的 ZooKeeper 元数据：

1. 在 AWS S3 中创建一个 bucket 以保存备份文件。
2. 新建访问 AWS S3 的凭证。
3. 用以下命令创建一个 secret 来保存 AWS 凭证。

	```bash
	kubectl create secret generic aws-secret          --from-literal=AWS_ACCESS_KEY_ID=<YOUR_ACCESS_KEY_ID>    --from-literal=AWS_SECRET_ACCESS_KEY=<YOUR_AWS_SECRET_ACCESS_KEY>
	```

4. Enable backup service in the ZooKeeper customTools section in the `values.yaml` file.

	```yaml

 	backup:
 	  component: "backup"
 	  enable: true
 	  webServerPort: "8088"
 	  backupInterval: "600"
 	  bucket: "s3a://<your-bucket-name>"
 	  backupPrefix: "pulsar-backup"
 	  managedLedgerPath: "/managed-ledgers"
 	  configData:
 	  secrets:
 	   	use: true
 	     aws:
 	       secretName: "aws-secret"
 	```

5. Create your cluster with the `values.yaml` file.

	```bash
	helm install  -f values.yaml snpe $PULSAR_CHART/  --set initialize=true --set namespace=snpe
	```

6. Check whether the backup service is running.

	```bash
	kubectl get pods snpe-restore-pulsar-backup-0
	```
	
	If the status is `running`, the backup service starts. You can also check the backup files in your bucket.

# Restore metadata from an old cluster

The restore service works only when you start a new cluster by using the old backup metadata, and it only restores data from tiered storage. If a cluster is running, you cannot restore it.

By default, the restore service is disabled. To restore your cluster to a backup point, complete the following steps:

1. Decide the restore point by finding a restore file from your bucket, for example: `s3a://my-backuprestore/pulsar-backup-1623721488825.tar.gz`.

2. Enable the restore service in the ZooKeeper customTools section in the `values.yaml` file that you use to deploy the cluster.

	```yaml

 	restore:
 	   component: "restore"
 	   enable: true
 	   restorePoint: "s3a://<your-bucket-name>/<backup-file>"
 	   restoreVersion: "1"
 	   bucket: "s3a://<your-bucket-name>"
 	   configData:
 	   secrets:
 	   	use: true
 	     aws:
 	       secretName: "aws-secret"
 	```

3. Restore your cluster with the `values.yaml`.

	```bash
	helm install  -f values.yaml snpe $PULSAR_CHART/  --set initialize=true --set namespace=snpe
	```

4. Check the restore service and make sure that ZooKeeper pods start.

	```bash
	kubectl get pods | grep zookeeper
	```
	You can also check the topics and namespaces created in the new cluster.

# Configuration

The following table lists the properties, description and default values of the backup and restore services.

| Property | Description | Default value |
| -------- | ----------- | ------------- |
| webServerPort | The backup HTTP service port for metrics and administration. | 8088 |
| backupInterval | The backup service running interval. The time unit is measured in seconds. | 600 |
| bucket | The bucket for saving backup files. | s3a://metadata-backup |
| backupPrefix | The prefix of the backup file name. | metadata-backup |
| managedLedgerPath | The Pulsar managed ledger path in ZooKeeper.  | /managed-ledgers |
| restorePoint | The specific time point that you want to restore your cluster. You can set the `restorePoint` to a cloud storage file path to restore the cluster with a specified restore point. | |
| restoreVersion | A unique number for restoration. If you want to restore the ZooKeeper multiple times with the same restore point, you need to specify a different restore version number for each restore. | 1 |

# Administration

The backup service provides some REST APIs for administration.

| Endpoint | Description | Example |
| -------- | ----------- | ------- |
| /backup/latest | Get the latest backup file | curl localhost:8088/backup/latest |
| /backup/list | Get all the backup files | curl localhost:8088/backup/list |

# Monitor

The backup service provides the metrics endpoint to monitor the following metrics. 

| Name | Type | Description |
| ---- | ---- | ----------- |
| backup_count | Counter | The count of running backups. |
| backup_failure_count | Counter | The count of failed backups. |
| backup_size_non_compressed | Gauge | The size of the backup files. |
| backup_size_compressed | Gauge | The size of the backup compressed file. |
| backup_size_total | Gauge | The size of all the backup compressed files. |
| backup_execution_time_total | Gauge | The time of the backup execution process. |
| backup_execution_time_prepare | Gauge | The time of the backup preparation process. |
| backup_execution_time_compression | Gauge | The time of compressing the backup files. |
| backup_execution_time_upload | Gauge | The time of uploading the backup files.|
