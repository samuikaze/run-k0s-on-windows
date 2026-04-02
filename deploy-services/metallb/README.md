# 安裝 MetalLB

裸機 (Bare-Metal) 安裝 Kubernetes 後 Service type 為 LoadBalancer 拿不到外部 IP，導致僅可使用 NodePort 提供服務是很不方便的，透過 MetalLB 後可以讓這類服務自動吃到外部 IP

## 安裝方法

依據[官方文件](https://metallb.io/installation/)說明進行安裝就可以了，其中因為我們的 KubeProxy 是走 iptables 模式，所以修改 KubeProxy 設定的部分可以直接跳過

## 常見問題

目前暫無遇到問題

## 參考資料

- [MetalLB](https://metallb.io/)
