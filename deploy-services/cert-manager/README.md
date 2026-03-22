# 安裝 cert-manager

<!-- markdownlint-disable MD028 -->
<!-- markdownlint-disable MD033 -->

cert-manager 對於自動輪替憑證非常有用，設定得當可以免除許多忘記輪替憑證導致的問題。

本文件會說明如何安裝 cert-manager 到 k0s 叢集中。

> 建議可以連同 trust-manager 一同安裝，讓服務信任憑證可以更加簡便

## 安裝方法

如果是有對外網路的叢集，其實可以直接依據[官方文件](https://cert-manager.io/docs/installation/)的說明來安裝，下面還是會說明如何安裝 cert-manager。

### 透過 kubectl 安裝

依據[官方文件](https://cert-manager.io/docs/installation/kubectl/)的說明，直接執行一下的指令就可以完成安裝了

> 連結可能不是最新版的連結，請到官方網站複製最新的連結

> 如果叢集無法對外連線，請先將 yaml 下載下來後，請透過直接指定 yaml 檔路徑方式來安裝

```bash
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.20.0/cert-manager.yaml
```

等待 cert-manager 命名空間中所有的 Pod 狀態都為 Up 就算完成安裝

### 透過 Helm 安裝

依據[官方文件](https://cert-manager.io/docs/installation/helm/)的說明，直接執行一下的指令就可以完成安裝了

> 版本號碼請改為想要的版本號碼，一般建議直接安裝最新版即可

> 如果叢集無法對外連線，請改以 kubectl 方式來安裝

```bash
helm install \
  cert-manager oci://quay.io/jetstack/charts/cert-manager \
  --version v1.20.0 \
  --namespace cert-manager \
  --create-namespace \
  --set crds.enabled=true
```

等待 cert-manager 命名空間中所有的 Pod 狀態都為 Up 就算完成安裝

## 常見問題

這邊會針對安裝 cert-manager 過程中常見的問題進行解答

### 叢集無法對外連線

若 k0s 所在的網路無法對外連線，安裝方式請改為 kubectl，並依據以下步驟進行安裝:

1. 將 yaml 檔下載下來
2. 打開 yaml 檔搜尋 `image:` 關鍵字，將該 yaml 檔中所有有使用到的 Image 都先透過以下指令拉下來

    > 拉取 Image 時請將完整的 Image 名稱與版本 Tag 都帶上，SHA 值可以不用帶，避免 containerd 找不到 Image

    > yaml 中若看到 Image 名稱和標籤後面還帶了 SHA 值，請將其移除，因為 containerd 只認完全一樣的 Image，透過 tar 檔匯入的方式 SHA 值會不同ㄋ

    ```bash
    # docker
    docker pull IMAGE_NAME:VERSION_TAG

    # podman
    podman pull IMAGE_NAME:VERSION_TAG
    ```

3. 把這些 Image 包成 tar 檔

    > 一樣指定 Image 時請將完整的 Image 名稱與版本 Tag 都帶上，SHA 值可以不用帶，避免 containerd 找不到 Image

    - bash

      ```bash
      # docker
      docker image save -o images.tar \
        IMAGE_NAME_1:VERSION_TAG \
        IMAGE_NAME_2:VERSION_TAG \
        IMAGE_NAME_3:VERSION_TAG

      # podman
      podman save -o images.tar \
        IMAGE_NAME_1:VERSION_TAG \
        IMAGE_NAME_2:VERSION_TAG \
        IMAGE_NAME_3:VERSION_TAG
      ```

4. 手動丟進 containerd 的 Image 本地儲存空間

    ```bash
    # Login as root user
    sudo su -

    # Import images from tar archive
    # Remember to replace /path/to/images.tar with currect path
    k0s ctr images import /path/to/images.tar
    ```

5. 執行 `kubectl apply -f cert-manager.yaml` 完成安裝

## 參考資料

- [cert-manager 官方文件](https://cert-manager.io/docs/)
