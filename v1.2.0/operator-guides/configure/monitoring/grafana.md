---
title: 配置 Grafana
id: grafana
category: operator-guides
---

[Grafana ](https://grafana.com/)是 StreamNative Platform 的一个组件。它为 StreamNative 平台提供了可观察性。Grafana 提供的仪表盘包括 broker、bookie、zookeeper、代理（proxy）、主题和消息。

默认情况下，Grafana 在 StreamNative Platform 处在被启用的状态。如果要禁用 Grafana，可以在 Pulsar 集群配置 YAML 文件中设置 `monitoring.grafana: false`。 

# 配置 Grafana

可以在 Pulsar 集群配置 YAML 文件中为 Grafana 配置要求和限制的 CPU 和内存。然后可以使用 `helm upgrade` 命令来重启 StreamNative Platform 使更新生效。

```
grafana:
  resources:
    # keep request = limit to keep this container in guaranteed class
    requests:
      cpu: 
      memory: 
    limits:
      cpu: 
      memory: 
```

# 暴露 Grafana 服务

StreamNative Platform 支持使用 [LoadBalancer](/operator-guides/configure/networking.md#loadbalancer) 或 [NodePort](/operator-guides/configure/networking.md#nodeport) 暴露 Granfana 服务。 

1. 启用 Grafana 并在 Pulsar 集群配置 YAML 文件中配置外部域。 

    ```
    Ingress:
        controller:
        enabled: true
    control_center:
        enabled: true
        # Set external domain of the load balancer of ingress controller
        # external_domain: 
        # external_domain_scheme: https://
    ```

2. 执行以下命令以获得 Grafana 服务的外部 IP 地址。

    ```
    kubectl get svc/RELEASE_NAME-sn-platform-nginx-ingress-controller -n KUBERNETES_NAMESPACE
    ```

3. 在浏览器中访问 `http://EXTERNAL_IP/grafana`，并使用用户名和密码登录。默认情况下，用户名和密码都是 `pulsar`。 

> **注**
> 
> 确保在登录到 Grafana 仪表板后更改密码。