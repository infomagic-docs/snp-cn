---
title: 在负载平衡器上启用 TLS 
id: tls-load-balancer
category: operator-guides
---

StreamNative Platform 支持用 [AWS Certificate Manager (ACM)](https://docs.aws.amazon.com/acm/latest/userguide/acm-overview.html) 启用 TLS。当你想在负载均衡器上执行 TLS 终止时，可以使用 ACM 证书。ACM 处理创建、存储和更新公共和私人 SSL/TLS X.509 证书和密钥的复杂性，以保护 AWS 网站和应用程序。

负载平衡器通过 TCP 协议将流量卸载到后端服务。 

* 一个用于 Pulsar 代理的负载平衡器，端口为 `6651/443`。 
  * DNS 名为 `data.pulsar.example.local`
* 一个用于 nginx 入口控制器的负载平衡器，端口为 `443`。 
  * DNS 名为 `admin.pulsar.example.local`
* 一个用于 istio 入口的负载平衡器（连接到 KoP broker），端口为 `9093`。 
  * DNS 名为 `messaging.pulsar.example.local`

> **注**     
> 使用 ACM 启用 TLS 不适用于 KoP，因为 KoP 需要 TLS 服务器名称指示 (SNI) 来路由 broker 端而不是负载均衡器端的 TLF 来终止流量。

要在 ACM 中使用证书，请完成以下步骤。

1. [从 ACM 申请公共证书](https://docs.aws.amazon.com/acm/latest/userguide/gs-acm-request-public.html) 获取以下域名，并获取证书的亚马逊资源名称 (Amazon Resource Name, ARN)。

	```
	*.pulsar.example.local
	```

2. 在 YAML 文件中启用域，配置和注释，并使用如上获得的 ARN。

   ```
   tls:
     enabled: false
     proxy:
       enabled: false

   ## Domain requested from External DNS
   domain:
     enabled: true
     suffix: pulsar.example.local

   ## Ingresses for exposing Pulsar services
   ingress:
     proxy:
       enabled: true
       external_domain: data.pulsar.example.local
       type: LoadBalancer
       tls:
         enabled: true
       annotations:
         external-dns.alpha.kubernetes.io/hostname: "data.pulsar.example.local"
         service.beta.kubernetes.io/aws-load-balancer-cross-zone-load-balancing-enabled: "true"
         service.beta.kubernetes.io/aws-load-balancer-type: nlb
         service.beta.kubernetes.io/aws-load-balancer-ssl-cert: "[ARN OF THE CERT]"
         # Enable this if the service is only accessed from the Amazon Virtual Private Cloud (Amazon VPC).
         #service.beta.kubernetes.io/aws-load-balancer-internal: 0.0.0.0/0
       extraSpec:
         externalTrafficPolicy: Local
   ## Ingresses for exposing monitoring/management services publicly
   controller:
     enabled: true
     annotations:
       external-dns.alpha.kubernetes.io/hostname: "admin.pulsar.example.local"
       service.beta.kubernetes.io/aws-load-balancer-ssl-cert: "[ARN OF THE CERT]"
       service.beta.kubernetes.io/aws-load-balancer-cross-zone-load-balancing-enabled: "true"
     extraSpec:
       externalTrafficPolicy: Local
     # flag whether to terminate the tls at the loadbalancer level
     tls:
       termination: true
   control_center:
     enabled: true
     tls:
       enabled: true
     # Set external domain for the load balancer of the ingress controller
     external_domain: admin.pulsar.example.local
     external_domain_scheme: https://
   ```

3. 通过重新启动 Pulsar 代理来应用更改。 

    ```
    helm upgrade -f /path/to/your/file.yaml CLUSTER_NAME $PULSAR_CHART/
    ```
