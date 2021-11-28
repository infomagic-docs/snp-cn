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

StreamNative 平台中，cert-manager 支持从内部发布者和公共发布者发布证书。以下部分介绍如何使用内部发布者和公共发布者生成证书，然后启用 TLS 加密。

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
    
    Cert-manager 自动生成所有证书并将它们附加到 Kubernetes pod。 

@@@

:::

# 在使用手动生成证书的 Pulsar 代理上启用 TLS 

在不使用 cert-manager 的情况下在 Pulsar 代理上启用 TLS 时，需要先手动生成证书。

To enable TLS on the Pulsar proxy, complete the following steps.

1. Create TLS certification files. For details, see [instructions](#create-tls-certificates).

2. Create secrets. 

    ```
    kubectl create secret generic RELEASE_NAME-sn-platform-proxy-tls -n KUBERNETES_NAMESPACE \
      --from-file=tls.crt=$(PWD)/cert/broker.cert.pem \
      --from-file=tls.key=$(PWD)/cert/broker.key-pk8.pem \
      --from-file=ca.crt=$(PWD)/cert/ca.cert.pem
    ```

3. Enable TLS on the Pulsar proxy. You can set both `tls` and `proxy` to `true`, and set `tlsSecretName`.

    ```
    tls:
      enabled: true
      proxy:
       enabled: true
    proxy:
      tlsSecretName: "RELEASE_NAME-sn-platform-proxy-tls"
    ```

4. Restart the Pulsar proxy.

    ```
    helm upgrade -f /path/to/your/file.yaml CLUSTER_NAME $PULSAR_CHART/
    ```

5. Get the proxy URL from the proxy-ingress service. To display the external IP address run the following command. 

    ```
    kubectl get svc/RELEASE_NAME-sn-platform-proxy-ingress -n KUBERNETES_NAMESPACE
    ```
    
6. Change the client configuration in the `config/client.conf` file. 

    When TLS is enabled, the connection protocol changes accordingly, for example, `http://` changes into `https://`, `pulsar://` changes into `pulsar+ssl://`.

    ```
    webServiceUrl=https://[PROXY_INGRESS_ADDRESS]/
    brokerServiceUrl=pulsar+ssl://[PROXY_INGRESS_ADDRESS]:6651/
    tlsAllowInsecureConnection=false
    tlsEnableHostnameVerification=false
    tlsTrustCertsFilePath=[PATH_TO]/ca.cert.pem
    ```
7. Validate the configuration by producing and consuming data with pulsar-client.

## Create TLS certificates manually

Creating TLS certificates for Pulsar involves creating a certificate authority (CA), server certificate, and client certificate.

You can follow these steps to set up a certificate authority or refer to [this guide](https://jamielinux.com/docs/openssl-certificate-authority/index.html) for details.

### Create certificate authority

1. Create the certificate for the certificate authority (CA). 

    > **NOTE**      
    > You can use a CA to sign both the broker and client certificates. This ensures that each party trusts the other. You should store the CA in a secure location (ideally completely disconnected from networks, air gapped, and fully encrypted).

2. Enter the following command to create a directory for the CA, and place the OpenSSL configuration file in the directory. 

    > **TIP**      
    > If you need to modify the default values for the company name and department in the configuration file, you can export the location of the CA directory to the environment variable, CA_HOME. The configuration file uses this environment variable to find the files and directories for the CA.

    ```bash
    mkdir my-ca
    cd my-ca
    wget https://raw.githubusercontent.com/apache/pulsar/master/site2/website/static/examples/openssl.cnf
    export CA_HOME=$(pwd)
    ```

3. Enter the following commands to create the necessary directories, keys and certs.

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

4. Answer the prompts.         
   CA-related files are stored in the `./my-ca` directory. 

   * `certs/ca.cert.pem` is the public certificate that is distributed to all parties involved.
   * `private/ca.key.pem` is the private key. You only need the key when you are signing a new certificate for either a broker or client. You must safely guard this private key.

### Generate server certificate

Server certificates are generated with the same certificate authority. Once you have created a CA certificate, you can create certificate requests and sign them with the CA.

> **Tips**     
> The common name should match the broker. You can also use a wildcard to match a group of broker hostnames, for example, `*.broker.usw.example.com`. This ensures that multiple machines can reuse the same certificate. However, when it is impossible to match the hostname, you can disable TLS hostname verification for the client. 

To create the certificates, complete the following steps:

1. Generate the key.

    ```bash
    openssl genrsa -out broker.key.pem 2048
    ```

    The key is in [PKCS 8](https://en.wikipedia.org/wiki/PKCS_8) format. You can convert it with the following command.

    ```bash
    openssl pkcs8 -topk8 -inform PEM -outform PEM \
          -in broker.key.pem -out broker.key-pk8.pem -nocrypt
    ```

2. Generate the certificate request.

    ```bash
    openssl req -config openssl.cnf \
        -key broker.key.pem -new -sha256 -out broker.csr.pem
    ```

3. Sign it with the certificate authority.

    ```bash
    openssl ca -config openssl.cnf -extensions server_cert \
       -days 1000 -notext -md sha256 \
        -in broker.csr.pem -out broker.cert.pem
    ```

At this point, you have a certificate, `broker.cert.pem`, and a key, `broker.key-pk8.pem`. You can use the cert and key along with `ca.cert.pem` to configure TLS transport encryption for the broker and proxy nodes.

# Enable TLS on KoP with manually-generated certificates

When you enable TLS on KoP without using cert-manager, you need to first generate the SSL key and certificates for KoP, and then create Kubernetes Secret for KoP certificates and passwords.  

## Generate SSL key and certificates for KoP

To connect to KoP through SSL, follow these steps.

1. Generate the keys and certificates.

    ```
    keytool -keystore server.keystore.jks -alias localhost -validity <validity> -keyalg RSA -genkey
    ```

    You need to specify two parameters in the above command:

    - `keystore`: the file that stores the certificate. The keystore contains the private key of the certificate and should be kept in a secure place.
    - `validity`: Days before the certificate expires. 

2. Generate the Certificate Authority (CA).

   1. Generate a CA.

      A CA is a public-private key pair and certificate used to sign other certificates.

      ```
      openssl req -new -x509 -keyout ca-key -out ca-cert -days <validity>
      ```

   2. Add the generated CA to the broker's truststore so that the brokers can trust this CA.

      ```
      keytool -keystore server.truststore.jks -alias CARoot -import -file ca-cert
      ```

   3. Add the generated CA to the client's truststore.

      ```
      keytool -keystore client.truststore.jks -alias CARoot -import -file ca-cert
      ```

3. Sign the certificate.

    1. Export the certificate from the keystore.

        ```
        keytool -keystore server.keystore.jks -alias localhost -certreq -file cert-file
        ```

    2. Sign the certificate with the CA.

        ```
        openssl x509 -req -CA ca-cert -CAkey ca-key -in cert-file -out cert-signed -days <validity> -CAcreateserial -passin pass:<ca-password>
        ```

    3. Import both the certificate of the CA and the signed certificate into the broker keystore.

        ```
        keytool -keystore server.keystore.jks -alias CARoot -import -file ca-cert
        keytool -keystore server.keystore.jks -alias localhost -import -file cert-signed
        ```

The definitions of the parameters are the following:

- keystore: the file that stores the certificate
- ca-cert: the certificate of the CA
- ca-key: the private key of the CA
- ca-password: the passphrase of the CA
- cert-file: the exported, unsigned certificate of the server
- cert-signed: the signed certificate of the server

## Create Kubernetes Secret for KoP certificates and passwords

Kubernetes Secrets let you store and manage sensitive information, such as passwords, tokens, and keys. Storing confidential information in a Secret is safer and more flexible than putting it verbatim in a Pod definition or in a container image.

1. Create the Kubernetes Secret for KoP certificates.

    ```
    kubectl create secret generic kop-secret -n KUBERNETES_NAMESPACE --from-file=keystore.jks=$(PWD)/cert/server.keystore.jks --from-file=truststore.jks=$(PWD)/cert/server.truststore.jks
    ```

2. Create the Kubernetes Secret for KoP certification passwords.

    ```
    kubectl create secret generic kop-keystore-password --from-literal=password=<ca-password> -n KUBERNETES_NAMESPACE
    
    ```

## Configure YAML file

To enable KoP and TLS, follow these steps.

1. Set the KoP parameters in the Pulsar configuration YAML file.

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

2. Apply the new configuration.

    ```
    helm upgrade -f /path/to/your/file.yaml CLUSTER_NAME $PULSAR_CHART/
    ```
