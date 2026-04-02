# 安裝 trust-manager

<!-- markdownlint-disable MD028 -->
<!-- markdownlint-disable MD033 -->

trust-manager 適用於當服務需要 ca-bundle 時，但每次 cert-manager 簽發或代理新憑證後，都要手動部署憑證到該服務的命名空間才能使用，且若忘記更新就會出大事，因此才會有 trust-manager 的誕生

> 請注意，trust-manager **只能**代理將 cert-manager 簽出來的憑證轉換成 ca-bundle 並部署到指定的命名空間中，如果服務需要的是 crt + key 格式者，則此套件不適用

由於目前沒有使用情境，因此本文件不會闡述太多注意事項。

## 安裝方法

目前官方僅提供 [Helm 方式進行安裝](https://cert-manager.io/docs/trust/trust-manager/installation/)，請直接遵照官方的說明進行安裝即可

> 如果無法連線外網，請自行下載 Helm Chart 後自行包版部署

## 常見問題

目前暫無遇到問題

## 參考資料

- [trust-manager - cert-manager](https://cert-manager.io/docs/trust/trust-manager/installation/)
