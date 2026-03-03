# 在 Windows 11 中執行 k0s

<!-- markdownlint-disable MD028 -->
<!-- markdownlint-disable MD033 -->

本文章會說明如何在 WSL2 中執行 RockyLinux 9，並在其中安裝 k0s

> 請注意，WSL 其用意仍在提供開發上的方便，正式環境的投入還是建議使用更完整的 Linux 作業系統來執行會比較安全

## Table of Contents

- [必要條件](#必要條件)
- [在 WSL 中安裝 RockyLinux](#在-wsl-中安裝-rockylinux)
- [安裝 k0s](#安裝-k0s)
- [透過 Windows 應用程式來存取 WSL 中的 k0s](#透過-windows-應用程式來存取-wsl-中的-k0s)
- [維持 WSL 背景執行](#維持-wsl-背景執行)
- [參考資料](#參考資料)

## 必要條件

為了讓 Windows 可以執行 WSL 與 k0s，必須符合以下條件

- Windows 功能中必須啟用 `Windows 子系統 Linux 版` 與 `虛擬機器平台`
- WSL 核心版本須高於 5.11
  > 必須支援 systemd
- 終端機

## 在 WSL 中安裝 RockyLinux

1. 依據[官方文件](https://docs.rockylinux.org/9/guides/interoperability/import_rocky_to_wsl/)的說明，先下載 WSL 映像檔
2. 透過以下指令安裝:

    > 執行前請先修改以下變數值:
    >
    > - `PATH_TO_IMAGE_FILE` 請取代為下載下來的 WSL 映像檔完整路徑
    > - `DISTRO_NAME`: 請為這個 WSL 命名，當此 WSL 不是預設的機器時，可以透過指令 `wsl -d DISTRO_NAME` 啟動

    ```Powershell
    wsl --install --from-file PATH_TO_IMAGE_FILE --name DISTRO_NAME
    ```

3. 等待其解壓縮完後，系統會自動啟動並登入該機器，此時請為使用者設定一個名稱
4. 為了增強安全性，請透過指令 `sudo passwd $USER` 為此使用者設定密碼，若無此需求，請直接跳至第 7. 點
    > 為了增強安全性還是建議為使用者設定密碼，多一道保護總比都沒有來的好
5. 如有設定 root 使用者密碼的需求，請先執行 `exit` 指令登出 WSL 機器，並改用 `wsl -u root` 登入 WSL 機器
    > 或是 `wsl -d DISTRO_NAME -u root` 如果這個 WSL 機器不是你的預設機器

    > 一般使用者其實也可以透過 `sudo su -` 來切換使用者為 root 使用者

    > RockyLinux 官方有為 Root 使用者設定預設密碼，但還是推薦將 root 使用者的密碼改掉，畢竟管理員掌握最高權限使用者密碼是在正常不過的事
6. 登入 root 後一樣執行 `passwd` 指令，完成密碼設定
7. 完成後就可以執行指令 `exit` 登出 root 使用者，並透過正常方式登入 WSL 機器
8. 執行指令 `sudo dnf update --refresh -y` 來將套件更新到最新
9. 若有任何其它套件安裝需求如 tmux 等，也可以在這步驟順便安裝
10. 完成

## 安裝 k0s

這邊請透過以下步驟安裝 k0s 到系統上:
> 若有離線安裝的需求，請參考官方文件的說明，本文件僅會針對線上安裝做說明

1. 執行指令 `curl --proto '=https' --tlsv1.2 -sSf https://get.k0s.sh | sudo K0S_VERSION=v1.34.4+k0s.0 sh` 安裝 k0s
    > 若有變更版本號碼需求，請修改 `K0S_VERSION=v1.34.4+k0s.0` 為正確的版本號碼
2. 執行指令 `echo "export PATH=\$PATH:/usr/local/bin" | sudo tee -a /etc/profile` 把路徑加到 PATH 環境變數中
    > 可以透過指令 `echo $PATH` 來確認是否有成功加入
3. 執行指令 `sudo su -` 切換到 root 使用者
    > 以下指令皆須 root 權限才能正常執行
4. 執行指令 `k0s install controller --enable-worker` 安裝 k0s Controller，並啟用 Worker 功能
5. 執行指令 `systemctl start k0scontroller && systemctl enable k0scontroller` 啟用 k0scontroller 服務
6. 執行完上述指令後，等待約一分鐘後，就可以透過 `k0s kubectl get nodes` 或 `k0s kubectl get pods --all-namespaces` 來確認啟動狀態
7. 為了避免未來執行 `k0s` 指令出現權限問題，請以 root 使用者執行 `k0s kubeconfig admin > ~/.kube/config` 指令
    > 透過 `sudo su -` 來切換為 root 使用者
8. 回到原本的使用者，執行以下指令:
    > 透過 `logout` 或 `exit` 來登出 root 使用者

    ```bash
    sudo chown $USER:$USER ~/.kube/config
    export KUBECONFIG=~/.kube/config
    ```

9. 執行指令 `vi ~/.bash_profile`，並將以下內容貼到文件最後一行

    ```bash
    export KUBECONFIG=~/.kube/config
    ```

## 透過 Windows 應用程式來存取 WSL 中的 k0s

若有此需求，請依據以下步驟來設定:

1. 透過指令 `wsl` 登入 WSL 機器
    > 或 `wsl -d DISTRO_NAME` 如果這個 WSL 機器不是你的預設機器
2. 執行指令 `sudo cp ~/.kube/config /mnt/c/Users/USER_NAME/Downloads` 來將 config 檔案複製到 Windows 的下載資料夾中
    > `USER_NAME` 為 Windows 的使用者名稱，請自行取代為正確的名稱
3. 透過文字編輯器打開這個 config 檔案，並依據以下說明修改:
    - 將 `server` 修改為 `https://localhost:6443`
      > 這邊預設 WSL 網路是走 NAT 模式，在此模式下 Windows 會自動將 WSL 中暴露的連接埠轉發到 Windows 主機上，因此可以透過 localhost 來存取
    - 在 `server` 下新增 `insecure-skip-tls-verify: true`
      > 由於 localhost 並不支援 TLS 驗證，且 k0s 的憑證為自簽憑證，本機管理的話直接這樣設定就可以了，若會透過遠端管理，建議還是透過 ssh 等工具加強遠端連線與資料傳輸的安全性

      > 請注意縮排必須與 `server` 行相同，其是隸屬於 `cluster` 的設定項目
    - 將 `certificate-authority-data` 整行註解掉或刪掉
4. 將此 `config` 檔案複製或移動到 `C:\Users\USER_NAME\.kube` 資料夾下
    > `USER_NAME` 為 Windows 的使用者名稱，請自行取代為正確的名稱
5. 此時透過第三方開源工具就可以存取 WSL 機器中的 k0s 服務囉

## 維持 WSL 背景執行

會需要在 WSL 中安裝 k0s 想必也是想在裡面架設相關服務，但 WSL 的設計是當使用者完全退出 WSL 機器後，該機器會開始執行 Graceful Shutdown，除非真的是單純開發測試用，就可以不須理會此段說明，否則請依以下步驟來設定讓 WSL 不要自動關閉機器

> 講白了就是透過**一般使用者**在背景執行**不會終止的程序**來達成目的

> 請注意，若執行過 `wsl -t DISTRO_NAME` 或 `wsl --shutdown` 關閉過 WSL 機器，就一定要重新執行一次第 3. 點的指令，否則 WSL 機器還是會被自動停止

1. 先透過指令 `wsl` 登入 WSL 機器
    > 或 `wsl -d DISTRO_NAME` 如果這個 WSL 機器不是你的預設機器
2. 執行指令 `sudo dnf install dbus-x11 -y` 安裝 dbus-x11 套件
3. 退出 WSL 機器，並執行 `wsl -u root --exec bash -c "dbus-launch true; echo hello"` 指令
    > 或 `wsl -d DISTRO_NAME -u root --exec bash -c "dbus-launch true; echo hello"` 如果這個 WSL 機器不是你的預設機器
4. 過五分鐘後執行指令 `wsl -l -v` 來確認該 WSL 機器有沒有被自動停止，若沒有就表示設定成功
5. 為了避免未來每次重啟 Windows 機器都還要手動執行此指令，可以透過 Windows 排程管理器來自動執行此專案下提供的 keep-wsl-running.bat 檔案
6. 完成

## 備份 WSL 機器

若有備份或匯出 WSL 機器的需求，請透過以下方式進行:

1. 執行指令 `wsl -t DISTRO_NAME` 或 `wsl --shutdown` 停止 WSL 機器
    > `DISTRO_NAME` 為 WSL 機器的名稱，請自行取代為正確的名稱
2. 執行指令 `wsl --export DISTRO_NAME C:\Users\USER_NAME\Downloads\rl9-k0s-wsl.tar` 來匯出 WSL 機器
    > `USER_NAME` 為 Windows 的使用者名稱，請自行取代為正確的名稱
3. 完成

## 參考資料

- [Import Rocky Linux to WSL or WSL2 - RockyLinux 9](https://docs.rockylinux.org/9/guides/interoperability/import_rocky_to_wsl/)
- [Install Kubernetes Cluster using k0s on Rocky Linux 9 - CloudSpnix](https://cloudspinx.com/install-kubernetes-cluster-using-k0s-on-rocky-linux/)
- [K0s kubeconfig admin - k0s Documentation](https://docs.k0sproject.io/head/cli/k0s_kubeconfig_admin/)
- [Simple WSL Keep-Alive that actually works keeping WSL Ubuntu running in background - r/bashonubuntuonwindows - reddit](https://www.reddit.com/r/bashonubuntuonwindows/comments/1n06q2k/simple_wsl_keepalive_that_actually_works_keeping/)
- [How to keep WSL running in the background](https://blog.lecoteauverdoyant.co.uk/articles/wsl-keep-alive.html)
- [dbus-launch command not found on alpine linux running wayland #939 - Spotifyd/spotifyd - GitHub](https://github.com/Spotifyd/spotifyd/issues/939#issuecomment-859192430)
- [匯出發行版本 - WSL 的基本命令 - Microsoft Learn](https://learn.microsoft.com/zh-tw/windows/wsl/basic-commands#export-a-distribution)
