---
title: 在 StreamNative Platform 组件上启用 TLS
id: tls-proxy
category: operator-guides
---

StreamNative Platform 支持传输层安全 (TLS)，一种行业标准的加密协议，以保护 StreamNative Platform 组件的网络通信。

TLS 依赖于 key 和证书来建立可信连接。可以在 Pulsar 代理、控制中心和 KoP broker 处执行 TLS 终止。本节介绍如何为 StreamNative Platform 组件启用 TLS 加密。

为了启用 TLS 加密，StreamNative Platform 支持以下机制：

- 自动生成证书：[cert-manager](https://cert-manager.io/docs/) 自动生成证书。
- 手动生成证书：用户生成私钥、公钥和证书授权。

对于不需要使用自己的服务器证书的场景，我们建议使用自动生成证书功能。

# 使用 cert-manager 启用 TLS

Cert-manager 将证书和证书颁发者作为资源类型添加到 Kubernetes 集群中，并简化了获取、更新和使用这些证书的过程。

StreamNative 平台中，cert-manager 支持从内部发布者和公共发布者授权证书。以下部分介绍如何使用内部发布者和公共发布者生成证书，然后启用 TLS 加密。

::: tabs

 @@@ 内部发布者

使用内部发布者生成证书（自签名），请按如下步骤操作。

1. 为 JKS  cert 创建密码密钥。 

	```
	kubectl create secret generic cert-jks-passwd --from-literal=password=passwd -n KUBERNETES_NAMESPACE
	```

2. 在 Pulsar 代理上启用 TLS。可以在 YAML 文件中将 `tls`、`proxy`、`internal_issuer` 和相关参数设置为 `true`，如下所示。

	```
	tls:

 	 enabled: true
 	 proxy:
 	   enabled: true
 	
 	certs:
 	 internal_issuer:
 	   enabled: true
 	
 	broker:
 	  # set the domain for accessing KoP 
 	  advertisedDomain: "messaging.pulsar.example.local"
 	  kop:
 	    enabled: true
 	    tls:
 		  enabled: true
 	
 	domain:
 	  enabled: true
 	  suffix: pulsar.example.local
 	
 	ingress:
 	  proxy:
 	  enabled: true
 	  tls:
 	    enabled: true
 	control_center:
 	  enabled: true
 	  tls:
 	    enabled: true
 	```
 	
 	Cert-manager automatically configures the “certName” for the self-signed certificate. 

3. 重新启动 Pulsar 代理来应用更改。

    ```
    helm upgrade -f /path/to/your/file.yaml CLUSTER_NAME $PULSAR_CHART/
    ```
    
    Cert-manager 自动生成所有证书并将它们附加到 Kubernetes pod。

4. 将 TLS CA 证书下载到本地目录。 `ca.crt` 用于连接 Pulsar 代理，`truststore.jks` 用于 Kafka 客户端连接 KoP。

    ```
    kubectl get secret CLUSTER_NAME-tls-proxy -o=jsonpath='{.data.ca\.crt}' -n KUBERNETES_NAMESPACE | base64 --decode -o ca.crt
    kubectl get secret CLUSTER_NAME-tls-proxy -o=jsonpath='{.data.truststore\.jks}' -n KUBERNETES_NAMESPACE | base64 --decode -o truststore.jks
    ```

@@@

@@@ 公共发布者

要使用公共发布者（公共签名供加密）生成证书，请按如下步骤操作。 

1. 为 JKS cert 创建密码密钥。 

	```
	kubectl create secret generic cert-jks-passwd --from-literal=password=passwd -n KUBERNETES_NAMESPACE
	```

2. 在 [Amazon Route 53](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/Welcome.html) 中创建托管区域并获取托管 ID。

3. 在 Pulsar 代理上启用 TLS。可以在 YAML 文件中将 `tls`、`proxy`、`public_issuer` 和相关参数设置为 `true`，如图所示。

    ```
    tls:
      enabled: true
      proxy:
        enabled: true
        untrustedCa: false

    certs:
      internal_issuer:
        enabled: false
      public_issuer:
        enabled: true
      issuers:
        acme:
          email: certs-support@streamnative.io
          server: https://acme-v02.api.letsencrypt.org/directory
          solver: route53
          solvers:
            route53:
              region: us-west-2
              hostedZoneID: [YOUR_HOSTED_ZONE_ID]

    ## Domain requested from External DNS
    domain:
      enabled: true
      ## Cert-manager generates a wildcard certificate for this domain.
      suffix: pulsar.example.local

    broker:
     # set the domain for accessing KoP 
     # Use messaging as a prefix
     # Use the domain to access kop
     advertisedDomain: "messaging.pulsar.example.local"
     kop:
       enabled: true
       tls:
         enabled: true

    ingress:
      proxy:
        enabled: true
     # Use the domain to access proxy
        external_domain: data.pulsar.example.local
        type: LoadBalancer
        tls:
          enabled: true
        annotations:
          service.beta.kubernetes.io/aws-load-balancer-type: nlb
      controller:
        enabled: true
        annotations: 
          service.beta.kubernetes.io/aws-load-balancer-type: nlb
      control_center:
        enabled: true
        tls:
          enabled: true
        # Use the domain to access control center
        external_domain: admin.pulsar.example.local
        external_domain_scheme: https://
    ```
	Cert-manager 自动为自签名证书配置 `certName`。

4. 重新启动 Pulsar 代理来应用更改。

    ```
    helm upgrade -f /path/to/your/file.yaml CLUSTER_NAME $PULSAR_CHART/
    ```
    
    Cert-manager 自动生成所有证书并将它们追加到 Kubernetes pod。 

@@@

:::

# 在使用手动生成证书的 Pulsar 代理上启用 TLS 

在不使用 cert-manager 的情况下在 Pulsar 代理上启用 TLS 时，需要先手动生成证书。

要在 Pulsar 代理上启用 TLS，请完成以下步骤。

1. 创建 TLS 认证文件。详情参见[说明](#create-tls-certificates)。

2. 创建密文。 

    ```
    kubectl create secret generic RELEASE_NAME-sn-platform-proxy-tls -n KUBERNETES_NAMESPACE \
      --from-file=tls.crt=$(PWD)/cert/broker.cert.pem \
      --from-file=tls.key=$(PWD)/cert/broker.key-pk8.pem \
      --from-file=ca.crt=$(PWD)/cert/ca.cert.pem
    ```

3. 在 Pulsar 代理上启用 TLS。可以把 `tls` 和 `proxy` 都设置为 `true`，并设置 `tlsSecretName`。 

    ```
    tls:
      enabled: true
      proxy:
       enabled: true
    proxy:
      tlsSecretName: "RELEASE_NAME-sn-platform-proxy-tls"
    ```

4. 重启 Pulsar 代理。

    ```
    helm upgrade -f /path/to/your/file.yaml CLUSTER_NAME $PULSAR_CHART/
    ```

5. 从代理入口服务获取代理 URL。运行如下命令来显示外部 IP 地址。 

    ```
    kubectl get svc/RELEASE_NAME-sn-platform-proxy-ingress -n KUBERNETES_NAMESPACE
    ```
    
6. 在 `config/client.conf` 文件中更改客户端配置。

    启用 TLS 后，连接协议会相应变化，例如 `http://` 变成 `https://`，`pulsar://` 变成 `pulsar+ssl://`。

    ```
    webServiceUrl=https://[PROXY_INGRESS_ADDRESS]/
    brokerServiceUrl=pulsar+ssl://[PROXY_INGRESS_ADDRESS]:6651/
    tlsAllowInsecureConnection=false
    tlsEnableHostnameVerification=false
    tlsTrustCertsFilePath=[PATH_TO]/ca.cert.pem
    ```
7. 通过使用 pulsar-client 生成和使用数据来验证配置。

## 手动创建 TLS 证书

为  Pulsar  创建 TLS 证书涉及创建证书授权（CA）、服务器证书和客户端证书。

您可以按照以下步骤设置证书授权，或参阅[本指南](https://jamielinux.com/docs/openssl-certificate-authority/index.html)了解详细信息。

### 创建证书授权

1. 为证书授权（CA）创建证书。 

    > **注**      
    > 可以使用 CA 来签署 broker 和客户端证书。这确保了每一方都信任另一方。应该将 CA 存储在一个安全的位置（理想情况下与网络完全断开、与空气隔绝（air gapped）并完全加密）。

2. 输入以下命令，为 CA 创建一个目录，并将 OpenSSL 配置文件放在该目录中。   

    > **提示**      
    > 如果需要修改配置文件中公司名称和部门的默认值，可以将 CA 目录的位置导出到环境变量 CA_HOME 中。配置文件使用此环境变量来查找 CA 的文件和目录。

    ```bash
    mkdir my-ca
    cd my-ca
    wget https://raw.githubusercontent.com/apache/pulsar/master/site2/website/static/examples/openssl.cnf
    export CA_HOME=$(pwd)
    ```

3. 输入以下命令，来创建必要的目录、key 和证书。 

    ```bash
    mkdir certs crl newcerts private
    chmod 700 private/
    touch index.txt
    echo 1000 > serial
    openssl genrsa -aes256 -out private/ca.key.pem 4096
    chmod 400 private/ca.key.pem
    openssl req -config openssl.cnf -key private/ca.key.pem \
        -new -x509 -days 7300 -sha256 -extensions v3_ca \
        -out certs/ca.cert.pem
    chmod 444 certs/ca.cert.pem
    ```

4. 回答提示。         
   CA 相关文件存储在 `./my-ca` 目录中。 

   * `certs/ca.cert.pem` 是分发给所有相关方的公共证书。 
   * `private/ca.key.pem` 是私钥。只有在为 broker 或客户签署新证书时才需要这个秘钥。必须安全地保护此私钥。

### 生成服务器证书 

服务器证书是使用相同的 CA 证书后，可以创建证书请求并使用 CA 对其进行签名。

> **提示**     
> 通用名称应与 broker 相匹配。 还可以使用通配符来匹配代理主机名，例如，`*.broker.usw.example.com`。这样确保了多台机器可以重复使用相同的证书。但是当无法匹配主机名时，你可以禁用客户端的 TLS 主机名验证。

要创建证书，请按如下步骤操作。 

1. 生成 key。

    ```bash
    openssl genrsa -out broker.key.pem 2048
    ```

    该 key 为 [PKCS 8 ](https://en.wikipedia.org/wiki/PKCS_8)格式。可以用下面的命令对其进行转换。

    ```bash
    openssl pkcs8 -topk8 -inform PEM -outform PEM \
          -in broker.key.pem -out broker.key-pk8.pem -nocrypt
    ```

2. 生成证书申请。

    ```bash
    openssl req -config openssl.cnf \
        -key broker.key.pem -new -sha256 -out broker.csr.pem
    ```

3. 用 CA 进行签字。

    ```bash
    openssl ca -config openssl.cnf -extensions server_cert \
       -days 1000 -notext -md sha256 \
        -in broker.csr.pem -out broker.cert.pem
    ```

此时你已经有证书，`broker.cert.pem`，和 key，`broker.key-pk8.pem`。你可以使用证书和 key 以及 `ca.cert.pem` 来为代理和代理节点配置 TLS 传输加密。

# 使用手动生成的证书在 KoP 上启用TLS 

当不使用 cert-manager 在 KoP 上启用 TLS 时，首先要为 KoP 生成 SSL key 和证书，然后为 KoP 证书和 key 创建 Kubernetes 密文（Kubernetes Secret）。 

## 为 KoP 生成 SSL key 和证书 

按如下步骤操作，通过 SSL 连接到 KoP。

1. 生成 key 和证书。

    ```
    keytool -keystore server.keystore.jks -alias localhost -validity <validity> -keyalg RSA -genkey
    ```

    上述命令中，需要指明两个参数： 

    - `keystore`：存储证书的文件。Keystore 包含证书的私钥，应该保存在一个安全的地方。
    - `validity`：证书到期剩余的天数。

2. 生成证书授权（CA）。

   1. 生成 CA。

      CA 是指用于签署其他证书的公-私密钥对和证书。

      ```
      openssl req -new -x509 -keyout ca-key -out ca-cert -days <validity>
      ```

   2. 将生成的 CA 添加到 broker 的 truststore（信任库）中，这样 broker 就可以信任这个 CA。 

      ```
      keytool -keystore server.truststore.jks -alias CARoot -import -file ca-cert
      ```

   3. 将生成的 CA 添加到客户的 truststore（信任库）中。 

      ```
      keytool -keystore client.truststore.jks -alias CARoot -import -file ca-cert
      ```

3. 证书签字。

    1. 将证书从 keystore 中导出。

        ```
        keytool -keystore server.keystore.jks -alias localhost -certreq -file cert-file
        ```

    2. 用 CA 对证书签字。 

        ```
        openssl x509 -req -CA ca-cert -CAkey ca-key -in cert-file -out cert-signed -days <validity> -CAcreateserial -passin pass:<ca-password>
        ```

    3. 将 CA 的证书和已签字的证书都导入到 broker 的 keystore 中。 

        ```
        keytool -keystore server.keystore.jks -alias CARoot -import -file ca-cert
        keytool -keystore server.keystore.jks -alias localhost -import -file cert-signed
        ```

参数的定义如下： 

- keystore：证书中储存的文件
- ca-cert：CA 的证书
- ca-key：CA 的私钥
- ca-password：CA 的口令
- cert-file：导出的未签字的服务器证书
- cert-signed：已签字的服务器证书

## 为 KoP 证书和密码创建 Kubernetes 密文 

Kubernetes  密文（Kubernetes Secret）供你存储和管理敏感信息，如密码、token 和 key。在密文中存储机密信息比在 Pod 定义或容器镜像中逐字存储更加安全灵活。

1. 为 KoP 证书创建 Kubernetes  密文（Kubernetes Secret）。

    ```
    kubectl create secret generic kop-secret -n KUBERNETES_NAMESPACE --from-file=keystore.jks=$(PWD)/cert/server.keystore.jks --from-file=truststore.jks=$(PWD)/cert/server.truststore.jks
    ```

2. 为 KoP 证书密码创建 Kubernetes  密文（Kubernetes Secret）。

    ```
    kubectl create secret generic kop-keystore-password --from-literal=password=<ca-password> -n KUBERNETES_NAMESPACE
    
    ```

## 配置 YAML 文件

按如下步骤操作启用 KoP 和 TLS。

1. 在 Pulsar 配置 YAML 文件中设置 KoP 参数。 

    ```
    # The domain name for accessing KoP outside from the Kuberbetes cluster.
    advertisedDomain: "kop.sn.dev"
    kop:
      enabled: true
      tls:
        enabled: true
        # create a secret with keystore.jks and truststore.jks for KoP TLS security
        certSecretName: "kop-secret"
        # create a secret for keystore and truststore cert
        passwordSecretRef:
          key: password
          name: kop-keystore-password
    ```

2. 应用新的配置。

    ```
    helm upgrade -f /path/to/your/file.yaml CLUSTER_NAME $PULSAR_CHART/
    ```
