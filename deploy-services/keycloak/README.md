# 安裝 Keycloak

<!-- markdownlint-disable MD028 -->
<!-- markdownlint-disable MD033 -->

Keycloak 在提供身分驗證有非常豐富的介面與方式可以介接，甚至連 Kubenetes 的 kube-apiserver 都可以用它來驗證使用者身分。

下面會介紹如何安裝 Keycloak 到 k0s 叢集中

> 建議安裝前先準備好 PostgreSQL 或是其它資料庫來存放設定，否則所有的設定會在服務重啟時遺失

## 安裝方法

請依據以下方式進行安裝

1. 打開同資料夾下的 `deployment.yaml`
2. 檢視所有設定是否符合需求
3. 執行指令 `kubectl apply -f deployment.yaml` 將 Namespace、Deployment、Service、ConfigMap、Secret 部署到 k0s 叢集中
    > 若 Namespace 已存在，請將 Namespace 整個註解掉，否則部署會失敗

    > 部署前特別需要檢查 ConfigMap 與 Secret 是否都已經修改完成
4. 若有對外開放的需求，請先打開 `ingress.yaml` 確定檔案中的宣告是否都符合需求
    > 部署前特別需要檢查 `spec.tls.hosts` 與 `rules` 中的設定
5. 執行指令 `kubectl apply -f ingress.yaml` 將 Ingress 部署到 k0s 叢集中
6. 完成

## 常見問題

### 我沒有網域的話可以透過 IP 簽發憑證嗎

可以，cert-manager 支援透過 IP 簽發憑證，唯一要注意簽發憑證時使用的 IP 必須是「外部」看到的 IP，否則即便將該憑證安裝到客戶端上，驗證仍會失敗

### Ingress 設定好後可以正常存取，但 Keycloak 只要重新導向就掛掉

這個問題出在於 ingress 將請求的網址 Rewrite 掉了，對於 Keycloak 來說只要有設定 `http-relative-path`，除非 Keycloak 收到的網址中包含該路徑，否則會一直重新導向

這個問題兩種解法，一種是直接把 Keycloak 放在 `/` 路徑，這樣 Keycloak 也不用設定 `http-relative-path`，~~但都用 Ingress 了怎麼可能還這樣玩~~

第二種解法就是 ingress 不要 Rewrite 路徑，Keycloak 只要設定 `http-relative-path` 後，就會正常運作了

### Keycloak 啟不來或一直重啟

請先檢查 Log 看是否有哪邊沒有設定好，但大部份的問題都是 CPU 和記憶體給太低，目前 Deployment 中的設定是官方文件要求的最低設定，若要調整設定建議是只能高不能低

### Keycloak 所在的主機不能對外連線

請透過可以拉取 Image 的裝置將所需的 Image 拉下來後丟到主機中就可以了，啟動過程不需要對外連線

## 參考資料

- [Get started with Keycloak on Docker](https://www.keycloak.org/getting-started/getting-started-docker)
- [All configuration - Keycloak](https://www.keycloak.org/server/all-config)
- [Configuring a Docker registry](https://www.keycloak.org/securing-apps/docker-registry)
- [設定存活、就緒和啟動探針 - Kubernetes](https://kubernetes.io/zh-cn/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/#define-startup-probes)
- [Annotations - kubernetes/ingress-nginx - GitHub](https://github.com/kubernetes/ingress-nginx/blob/main/docs/user-guide/nginx-configuration/annotations.md)
