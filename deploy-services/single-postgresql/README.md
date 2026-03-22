# 安裝單節點 PostgreSQL

<!-- markdownlint-disable MD028 -->
<!-- markdownlint-disable MD033 -->

PostgreSQL 資料庫的生態與社群相對比較完整，是需要穩定服務的最佳選擇。

下面會介紹如何安裝 Keycloak 到 k0s 叢集中

## 安裝方法

1. 打開同資料夾下的 `deployment.yaml`
2. 檢視所有設定是否符合需求
    > 特別是 Secret 的設定，請確認是否已經修改為自己想要的密碼
3. 執行指令 `kubectl apply -f deployment.yaml` 將 Namespace、StatefulSet、Service、ConfigMap、Secret 部署到 k0s 叢集中
    > 若 Namespace 已存在，請將 Namespace 整個註解掉，否則部署會失敗

    > ConfigMap 目前不太需要，但若有特殊需求需要設定，也可以解開註解撰寫
4. 完成

## 常見問題

目前暫無遇到問題

## 參考資料

- [postgres - Official Image | Docker Hub](https://hub.docker.com/_/postgres)
- 親自測試
