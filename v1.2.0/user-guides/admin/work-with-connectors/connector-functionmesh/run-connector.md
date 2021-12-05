---
title: 运行连接器
id: run-connector
category: user-guides
---

本文介绍如何使用 Function Mesh 创建连接器。

# 前提条件

- 部署并连接到 [Kubernetes 集群](https://kubernetes.io/)。 
- 在 Kubernetes 集群中部署 [Pulsar集群](/operator-guides/deploy/sn-deploy.md#deploy-pulsar-clusters)。 
- 将 Function Mesh Operator 和 CRD 安装到 Kubernetes 集群中。 

# 步骤

根据其打包的方式来创建连接器。

1. 通过使用 YAML 文件定义一个连接器，并保存 YAML 文件。 

   - 本例中，通过指定连接器包的 URL，为一个名为 `my-sink-sample` 的sink 连接器定义 YAML 文件，YAML 文件被保存为 `sink-sample.yaml`。

       ```yaml
      apiVersion: compute.functionmesh.io/v1alpha1
      kind: Sink
      metadata:
        name: my-sink-package-sample
        namespace: default
      spec:
        image: streamnative/pulsar-functions-java-runner:2.7.1 # using java function runner
        className: org.example.MySink
        forwardSourceMessageProperty: true
        MaxPendingAsyncRequests: 1000
        replicas: 1
        maxReplicas: 1
        logTopic: persistent://public/default/logging-connector-logs
        input:
          topics:
          - persistent://public/default/input
          typeClassName: java.lang.String
        sinkConfig:
          myconfig: "test-config"
        pulsar:
          pulsarConfig: "test-pulsar"
        java:
          extraDependenciesDir: random-dir/
          jar: /pulsar/connectors/pulsar-io-my-sink.nar # the package will download as this filename.
          jarLocation: sink://public/default/my-sink@1.0 # connector package URL
      ```

   - 本例通过指定连接器的 Docker 镜像 URL，为名为 `my-sink-package-sample` 的 sink 连接器定义了 YAML 文件，YAML 文件被保存为`sink-sample.yaml`。

     ```yaml
      apiVersion: compute.functionmesh.io/v1alpha1
      kind: Sink
      metadata:
        name: my-sink-sample
        namespace: default
      spec:
        image: myorg/pulsar-io-my-sink:2.7.1 # using self built image
        forwardSourceMessageProperty: true
        MaxPendingAsyncRequests: 1000
        replicas: 1
        maxReplicas: 1
        logTopic: persistent://public/default/logging-connector-logs
        input:
          topics:
          - persistent://public/default/input
          typeClassName: java.lang.String
        sinkConfig:
          myconfig: "test-config"
        pulsar:
          pulsarConfig: "test-pulsar"
        java:
          extraDependenciesDir: random-dir/
          jar: /pulsar/connectors/pulsar-io-my-sink.nar # the NAR location in image.
          jarLocation: "" # leave empty since we will not download package from Pulsar Packages
     ```

2. 应用 YAML 文件来新建 sink 连接器。 

	```bash
	kubectl apply -f /path/to/sink-sample.yaml
	```

3. 验证 sink 连接器是否被成功创建。 

	```bash
	kubectl get all
	```

	**输出**
	
	```
	NAME                                READY   STATUS      RESTARTS   AGE
	pod/my-sink-sample              1/1     Running     0          77s
	```
	
	从输出中可以看到，sink 排连接器处于 `Running` 状态，表明 sink 连接器被成功创建。
