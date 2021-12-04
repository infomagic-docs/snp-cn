---
title: 更新证书
id: license-update
category: operator-guides
---

StreamNative Platform 提供 30 天试用证书让用户体验。30 天后，StreamNative Platform 将要求用户购买证书。联系 StreamNative Platform 的销售人员，向StreamNative 提交购买证书申请。从 StreamNative 收到文件后就可以上传证书。

按照如下步骤更新 StreamNative Platform 证书。

1. 进入 `toolset` Pod。

    要使用官方 Pulsar CLI 工具，例如 pulsar-client，StreamNative Platform 提供了 `toolset `Pod。你可以使用 `kubectl exec` 命令来连接到 Pod。

    ```
    kubectl exec -n KUBERNETES_NAMESPACE -it RELEASE_NAME-sn-platform-toolset-0 -- bash
    ```

2. 为认证 token 添加环境变量。 

    ```
    export AUTH_TOKEN=`cat /pulsar/tokens/client/token`
    ```

3. 根据从 StreamNative 收到的证书文件，更新证书信息。 

    ```
    curl -XPOST -H "Authorization:Bearer $AUTH_TOKEN" http://RELEASE_NAME-sn-platform-broker-headless:8080/snp/admin/v1/license -d '{
    "license": {
        "uid": "f8b79e9e-fb15-11eb-9412-bd766dc06cb8",
        "type": "enterprise",
        "issue_date_in_millis": 1628380800000,
        "expiry_date_in_millis": 1660262400000,
        "start_date_in_millis": 1628726400000,
        "max_cpu_cores": 10,
        "issued_to": "StreamNative Ltd.",
        "issuer": "binw@streamnative.io",
        "signature": "AAAAAQAAAA3QvaexD1qhAxUhWkOoAAAAIKEWNm+fXOi/V7uWBym/nC1KED2o527crKp1yZ5ahNGbAAABAEDcgIjhMfojDcs68Q26FLQ8ilsdiJvqDL2LxC1JUSS1e2NgOuIPhPruKdzJAVYmfXtAGjwVRtcUX45k+TtyI2l+5qlmt+BuS/5RVPQL524XC6Ptygp0N"
    }
    }'
    ```

    **输出**
    
    ```
    active
    ```
    
    输出为 `active` 即表明许证书已成功更新。