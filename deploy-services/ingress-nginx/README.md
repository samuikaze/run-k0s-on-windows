# 安裝 nginx-ingress

<!-- markdownlint-disable MD028 -->
<!-- markdownlint-disable MD033 -->

ingress-nginx 在 Kubernetes 被廣泛的被使用，以下會說明如何部署 ingress-nginx 到 k0s 中。

> ⚠️ [ingress-nginx 已經被標示為棄用](https://kubernetes.io/blog/2025/11/11/ingress-nginx-retirement/)，且於 2026/03 後就完全不進行任何更新，請使用 Gateway 實作

## 安裝方法

基本上直接按照[官方提供的安裝方式](https://kubernetes.github.io/ingress-nginx/deploy/)進行安裝就可以了

## 使用自簽憑證取代預設的 Fake Ingress 憑證

以下會說明如何透過 cert-manager 簽一張 IP 的自簽憑證並套用到 ingress-nginx

> 請先安裝 cert-manager 到 k0s 中，或是也可以自行簽一張憑證並放到 `ingress-nginx` 命名空間中的 Secret 中

1. 首先先部署 `self-signed-certificate-for-ingress.yaml` 到 k0s 中

    > 請記得先修改檔案中 `# external-certificate.yaml` 部分的 IP 區塊

2. 編輯 ingress-nginx-controller 的 Deployment，在 `spec.template.spec.containers[0].args` 中加入以下參數

    > 如果 `self-signed-certificate-for-ingress.yaml` 的 `external-ingress-certificate` 其 `secretName` 不是 `external-ingress-tls` 的話，請改成自己設定的名稱

    ```yaml
    - "--default-ssl-certificate=ingress-nginx/external-ingress-tls"
    ```

3. 完成，現在透過瀏覽器存取 k0s 中任意服務，就會看到憑證已經變成自謙的憑證了

## 常見問題

目前暫無遇到問題

## 參考資料

- [ingress-nginx](https://kubernetes.github.io/ingress-nginx/)
