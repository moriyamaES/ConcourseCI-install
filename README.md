# ConcourseCI-install
Concourse CI のインストール手順

## 参考
- ConcourseCI は v3.19以上のカーネルが必要。カーネル更新手順は以下を参考にする

    https://qiita.com/ryoctrl/items/8f786d3ccb2333ffc4d0


-  Concourse CI のインストール手順

    https://note.com/shift_tech/n/nead03c02b095


- Concourse CI のチュートリアル

    https://concoursetutorial-ja.site.lkj.io/


## 前提

- 以下の資料に従って harborをインストール済

    https://github.com/moriyamaES/harbor-install#readme


## 環境

```
# uname -r
3.10.0-1160.el7.x86_64
```

```
# minikube version 
minikube version: v1.21.0
commit: 76d74191d82c47883dc7e1319ef7cebd3e00ee11
```

```
# docker version 
Client: Docker Engine - Community
 Version:           20.10.7
 API version:       1.41
 Go version:        go1.13.15
 Git commit:        f0df350
 Built:             Wed Jun  2 11:58:10 2021
 OS/Arch:           linux/amd64
 Context:           default
 Experimental:      true

Server: Docker Engine - Community
 Engine:
  Version:          20.10.7
  API version:      1.41 (minimum version 1.12)
  Go version:       go1.13.15
  Git commit:       b0f5bc3
  Built:            Wed Jun  2 11:56:35 2021
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.4.6
  GitCommit:        d71fcd7d8303cbf684402823e425e9dd2e99285d
 runc:
  Version:          1.0.0-rc95
  GitCommit:        b9ee9c6314599f1b4a7f497e1f1f856fe433d3b7
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0
```
```
# docker-compose version
Docker Compose version v2.20.3
```

## カーネルアップデート

- ConcourseCI は v3.19以上のカーネルが必要。
- カーネルのバージョンをチェック

```
# uname -r
3.10.0-1160.el7.x86_64
```
```
# rpm -qa | grep "^kernel" | sort
kernel-3.10.0-1160.el7.x86_64
kernel-tools-3.10.0-1160.el7.x86_64
kernel-tools-libs-3.10.0-1160.el7.x86_64
```

- 3.10で要求を満たしていないのでカーネルをアップデートします。

- ELRepoを追加します。
- ELRepoにもバージョンがあるので[公式ページ](http://elrepo.org/tiki/tiki-index.php)を参考にしながらコマンドを実行していきます。
 ※ ここでは、[参考資料](https://qiita.com/ryoctrl/items/8f786d3ccb2333ffc4d0)の手順に従った

```
# rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
```
```
# rpm -Uvh https://www.elrepo.org/elrepo-release-7.0-3.el7.elrepo.noarch.rpm
```
- 結果
```
https://www.elrepo.org/elrepo-release-7.0-3.el7.elrepo.noarch.rpm を取得中
準備しています...                                                (################################# [100%]
更新中 / インストール中...
   1:elrepo-release-7.0-3.el7.elrepo                              ################################# [100%]
```
```
# cat /etc/yum.repos.d/elrepo.repo
```
- 結果
```
### Name: ELRepo.org Community Enterprise Linux Repository for el7
### URL: http://elrepo.org/

[elrepo]
name=ELRepo.org Community Enterprise Linux Repository - el7
baseurl=http://elrepo.org/linux/elrepo/el7/$basearch/
        http://mirrors.coreix.net/elrepo/elrepo/el7/$basearch/
        http://mirror.rackspace.com/elrepo/elrepo/el7/$basearch/
        http://repos.lax-noc.com/elrepo/elrepo/el7/$basearch/
        http://mirror.ventraip.net.au/elrepo/elrepo/el7/$basearch/
mirrorlist=http://mirrors.elrepo.org/mirrors-elrepo.el7
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-elrepo.org
protect=0

[elrepo-testing]
name=ELRepo.org Community Enterprise Linux Testing Repository - el7
baseurl=http://elrepo.org/linux/testing/el7/$basearch/
        http://mirrors.coreix.net/elrepo/testing/el7/$basearch/
        http://mirror.rackspace.com/elrepo/testing/el7/$basearch/
        http://repos.lax-noc.com/elrepo/testing/el7/$basearch/
        http://mirror.ventraip.net.au/elrepo/testing/el7/$basearch/
mirrorlist=http://mirrors.elrepo.org/mirrors-elrepo-testing.el7
enabled=0
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-elrepo.org
protect=0

[elrepo-kernel]
name=ELRepo.org Community Enterprise Linux Kernel Repository - el7
baseurl=http://elrepo.org/linux/kernel/el7/$basearch/
        http://mirrors.coreix.net/elrepo/kernel/el7/$basearch/
        http://mirror.rackspace.com/elrepo/kernel/el7/$basearch/
        http://repos.lax-noc.com/elrepo/kernel/el7/$basearch/
        http://mirror.ventraip.net.au/elrepo/kernel/el7/$basearch/
mirrorlist=http://mirrors.elrepo.org/mirrors-elrepo-kernel.el7
enabled=0
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-elrepo.org
protect=0

[elrepo-extras]
name=ELRepo.org Community Enterprise Linux Extras Repository - el7
baseurl=http://elrepo.org/linux/extras/el7/$basearch/
        http://mirrors.coreix.net/elrepo/extras/el7/$basearch/
        http://mirror.rackspace.com/elrepo/extras/el7/$basearch/
        http://repos.lax-noc.com/elrepo/extras/el7/$basearch/
        http://mirror.ventraip.net.au/elrepo/extras/el7/$basearch/
mirrorlist=http://mirrors.elrepo.org/mirrors-elrepo-extras.el7
enabled=0
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-elrepo.org
protect=0
```

- ELRepoが追加出来たら、以下を実行

```
# yum --enablerepo=elrepo-kernel -y install kernel-ml
```
- 結果
```
読み込んだプラグイン:fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: ftp.tsukuba.wide.ad.jp
 * elrepo: ftp.ne.jp
 * elrepo-kernel: ftp.ne.jp
 * extras: ftp.tsukuba.wide.ad.jp
 * updates: ftp.tsukuba.wide.ad.jp
base                                       | 3.6 kB     00:00     
docker-ce-stable                           | 3.5 kB     00:00     
elrepo                                     | 3.0 kB     00:00     
elrepo-kernel                              | 3.0 kB     00:00     
extras                                     | 2.9 kB     00:00     
ius                                        | 1.3 kB     00:00     
updates                                    | 2.9 kB     00:00     
(1/2): elrepo/primary_db                     | 357 kB   00:00     
(2/2): elrepo-kernel/primary_db              | 2.1 MB   00:00     
依存性の解決をしています
--> トランザクションの確認を実行しています。
---> パッケージ kernel-ml.x86_64 0:6.4.10-1.el7.elrepo を インストール
--> 依存性解決を終了しました。

依存性を解決しました

==================================================================
 Package     アーキテクチャー
                      バージョン            リポジトリー     容量
==================================================================
インストール中:
 kernel-ml   x86_64   6.4.10-1.el7.elrepo   elrepo-kernel    66 M

トランザクションの要約
==================================================================
インストール  1 パッケージ

総ダウンロード容量: 66 M
インストール容量: 339 M
Downloading packages:
kernel-ml-6.4.10-1.el7.elrepo.x86_64.rpm     |  66 MB   00:07     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
警告: RPMDB は yum 以外で変更されました。
  インストール中          : kernel-ml-6.4.10-1.el7.elrepo.x   1/1 
  検証中                  : kernel-ml-6.4.10-1.el7.elrepo.x   1/1 

インストール:
  kernel-ml.x86_64 0:6.4.10-1.el7.elrepo                          

完了しました!
```
- 更新内容を確認
```
# rpm -qa | grep "^kernel" | sort
kernel-3.10.0-1160.el7.x86_64
kernel-ml-6.4.10-1.el7.elrepo.x86_64
kernel-tools-3.10.0-1160.el7.x86_64
kernel-tools-libs-3.10.0-1160.el7.x86_64
```
- kernel-mlが更新されていればOKです。

- このままだと次回の再起動時は元の古いカーネルで起動されてしまうため起動設定を変えます。

```
# grub2-editenv list
saved_entry=CentOS Linux (3.10.0-1160.el7.x86_64) 7 (Core)
```

```
# awk -F\' '$1=="menuentry " {print i++ " : " $2}' /etc/grub2.cfg
0 : CentOS Linux (6.4.10-1.el7.elrepo.x86_64) 7 (Core)
1 : CentOS Linux (3.10.0-1160.el7.x86_64) 7 (Core)
2 : CentOS Linux (0-rescue-b89cc28406e54592a5591681ba669192) 7 (Core)
```

- 今回更新した6.4.10で起動したいので0をsetします。

```
# grub2-set-default 0
```

- カーネルヘッダやツールを一つずつ入れ替えていきます

    ```
    # yum --enablerepo=elrepo-kernel -y swap kernel-headers -- kernel-ml-headers
    ```


    - 結果

        ```
        読み込んだプラグイン:fastestmirror, langpacks
        引数に一致しません: kernel-headers
        swap remove kernel-headers
        ```


    ```
    # yum --enablerepo=elrepo-kernel -y swap kernel-tools-libs -- kernel-ml-tools-libs
    ```

    - 結果

        ```
        読み込んだプラグイン:fastestmirror, langpacks
        Loading mirror speeds from cached hostfile
        * base: ftp.tsukuba.wide.ad.jp
        * elrepo: ftp.ne.jp
        * elrepo-kernel: ftp.ne.jp
        * extras: ftp.tsukuba.wide.ad.jp
        * updates: ftp.tsukuba.wide.ad.jp
        依存性の解決をしています
        --> トランザクションの確認を実行しています。
        ---> パッケージ kernel-ml-tools-libs.x86_64 0:6.4.10-1.el7.elrepo を インストール
        ---> パッケージ kernel-tools-libs.x86_64 0:3.10.0-1160.el7 を 削除
        --> 依存性の処理をしています: kernel-tools-libs = 3.10.0-1160.el7 のパッケージ: kernel-tools-3.10.0-1160.el7.x86_64
        --> トランザクションの確認を実行しています。
        ---> パッケージ kernel-tools.x86_64 0:3.10.0-1160.el7 を 削除
        --> 依存性解決を終了しました。

        依存性を解決しました

        ==================================================================
        Package           アーキテクチャー
                                バージョン          リポジトリー   容量
        ==================================================================
        インストール中:
        kernel-ml-tools-libs
                        x86_64 6.4.10-1.el7.elrepo elrepo-kernel 183 k
        削除中:
        kernel-tools-libs x86_64 3.10.0-1160.el7     @anaconda      18 k
        依存性関連での削除をします:
        kernel-tools      x86_64 3.10.0-1160.el7     @anaconda     337 k

        トランザクションの要約
        ==================================================================
        インストール  1 パッケージ
        削除          1 パッケージ (+1 個の依存関係のパッケージ)

        総ダウンロード容量: 183 k
        Downloading packages:
        kernel-ml-tools-libs-6.4.10-1.el7.elrepo.x86 | 183 kB   00:00     
        Running transaction check
        Running transaction test
        Transaction test succeeded
        Running transaction
        インストール中          : kernel-ml-tools-libs-6.4.10-1.e   1/3 
        削除中                  : kernel-tools-3.10.0-1160.el7.x8   2/3 
        削除中                  : kernel-tools-libs-3.10.0-1160.e   3/3 
        検証中                  : kernel-ml-tools-libs-6.4.10-1.e   1/3 
        検証中                  : kernel-tools-3.10.0-1160.el7.x8   2/3 
        検証中                  : kernel-tools-libs-3.10.0-1160.e   3/3 

        削除しました:
        kernel-tools-libs.x86_64 0:3.10.0-1160.el7                      

        依存性の削除をしました:
        kernel-tools.x86_64 0:3.10.0-1160.el7                           

        インストール:
        kernel-ml-tools-libs.x86_64 0:6.4.10-1.el7.elrepo               

        完了しました!
        ```

    ```
    # yum --enablerepo=elrepo-kernel -y install kernel-ml-tools
    ```

    - 結果
        
        ```
        読み込んだプラグイン:fastestmirror, langpacks
        Loading mirror speeds from cached hostfile
        * base: ftp.tsukuba.wide.ad.jp
        * elrepo: ftp.ne.jp
        * elrepo-kernel: ftp.ne.jp
        * extras: ftp.tsukuba.wide.ad.jp
        * updates: ftp.tsukuba.wide.ad.jp
        依存性の解決をしています
        --> トランザクションの確認を実行しています。
        ---> パッケージ kernel-ml-tools.x86_64 0:6.4.10-1.el7.elrepo を インストール
        --> 依存性解決を終了しました。

        依存性を解決しました

        =====================================================================================================================================
        Package                          アーキテクチャー        バージョン                            リポジトリー                    容量
        =====================================================================================================================================
        インストール中:
        kernel-ml-tools                  x86_64                  6.4.10-1.el7.elrepo                   elrepo-kernel                  296 k

        トランザクションの要約
        =====================================================================================================================================
        インストール  1 パッケージ

        総ダウンロード容量: 296 k
        インストール容量: 435 k
        Downloading packages:
        kernel-ml-tools-6.4.10-1.el7.elrepo.x86_64.rpm                                                                | 296 kB  00:00:00     
        Running transaction check
        Running transaction test
        Transaction test succeeded
        Running transaction
        インストール中          : kernel-ml-tools-6.4.10-1.el7.elrepo.x86_64                                                           1/1 
        検証中                  : kernel-ml-tools-6.4.10-1.el7.elrepo.x86_64                                                           1/1 

        インストール:
        kernel-ml-tools.x86_64 0:6.4.10-1.el7.elrepo                                                                                       

        完了しました!
        ```

    ```
    # yum --enablerepo=elrepo-kernel -y swap kernel-devel -- kernel-ml-devel
    ```

    - 結果

        ```
        読み込んだプラグイン:fastestmirror, langpacks
        引数に一致しません: kernel-devel
        swap remove kernel-devel
        ```

- 最後に古いカーネルを削除します。

    ```
    # yum -y remove kernel
    ```

    - 結果

        ```
        読み込んだプラグイン:fastestmirror, langpacks
        実行中のカーネルを飛ばします: kernel-3.10.0-1160.el7.x86_64
        削除対象とマークされたパッケージはありません。
        ```

    ```
    # ls -l /lib/modules
    ```

    - 結果

        ```
        合計 8
        drwxr-xr-x. 7 root root 4096  7月  2 06:25 3.10.0-1160.el7.x86_64
        drwxr-xr-x. 7 root root 4096  8月 15 11:59 6.4.10-1.el7.elrepo.x86_64
        ```

- 自動起動が新しいカーネルになっていることを確認して再起動

    ```
    # grub2-editenv list
    ```
    - 結果

        ```
        saved_entry=0
        ```
    ```
     # shutdown -r now
    ```

- 再起動後、カーネルがバージョンアップされていることを確認

    ```
    # uname -r
    6.4.10-1.el7.elrepo.x86_64
    ```

- minikube の起動を確認

    ```
    # minikube start --vm-driver=none
    ```

    - 結果

        ```
        😄  Centos 7.9.2009 上の minikube v1.21.0
        ✨  プロフィールを元に、 none ドライバを使用します

        🧯  The requested memory allocation of 2200MiB does not leave room for system overhead (total system memory: 2909MiB). You may face stability issues.
        💡  提案: Start minikube with less memory allocated: 'minikube start --memory=2200mb'

        👍  コントロールプレーンのノード minikube を minikube 上で起動しています
        🔄  既存の none bare metal machine を "minikube" のために再起動しています...
        🎉  minikube 1.31.1 が利用可能です! 以下のURLでダウンロードできます。 https://github.com/kubernetes/minikube/releases/tag/v1.31.1
        💡  To disable this notice, run: 'minikube config set WantUpdateNotification false'

        ℹ️  OS は CentOS Linux 7 (Core) です。
        🐳  Docker 20.10.7 で Kubernetes v1.20.7 を準備しています...
        🤦  Unable to restart cluster, will reset it: apiserver healthz: apiserver process never appeared
            ▪ 証明書と鍵を作成しています...
            ▪ Control Plane を起動しています...
        💢  初期化が失敗しました。再施行します。 wait: /bin/bash -c "sudo env PATH=/var/lib/minikube/binaries/v1.20.7:$PATH kubeadm init --config /var/tmp/minikube/kubeadm.yaml  --ignore-preflight-errors=DirAvailable--etc-kubernetes-manifests,DirAvailable--var-lib-minikube,DirAvailable--var-lib-minikube-etcd,FileAvailable--etc-kubernetes-manifests-kube-scheduler.yaml,FileAvailable--etc-kubernetes-manifests-kube-apiserver.yaml,FileAvailable--etc-kubernetes-manifests-kube-controller-manager.yaml,FileAvailable--etc-kubernetes-manifests-etcd.yaml,Port-10250,Swap,Mem": exit status 1
        stdout:
        [init] Using Kubernetes version: v1.20.7
        [preflight] Running pre-flight checks
        [preflight] Pulling images required for setting up a Kubernetes cluster
        [preflight] This might take a minute or two, depending on the speed of your internet connection
        [preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
        [certs] Using certificateDir folder "/var/lib/minikube/certs"
        [certs] Using existing ca certificate authority
        [certs] Using existing apiserver certificate and key on disk
        [certs] Using existing apiserver-kubelet-client certificate and key on disk
        [certs] Using existing front-proxy-ca certificate authority
        [certs] Using existing front-proxy-client certificate and key on disk
        [certs] Using existing etcd/ca certificate authority
        [certs] Using existing etcd/server certificate and key on disk
        [certs] Using existing etcd/peer certificate and key on disk
        [certs] Using existing etcd/healthcheck-client certificate and key on disk
        [certs] Using existing apiserver-etcd-client certificate and key on disk
        [certs] Using the existing "sa" key
        [kubeconfig] Using kubeconfig folder "/etc/kubernetes"
        [kubeconfig] Writing "admin.conf" kubeconfig file
        [kubeconfig] Writing "kubelet.conf" kubeconfig file
        [kubeconfig] Writing "controller-manager.conf" kubeconfig file
        [kubeconfig] Writing "scheduler.conf" kubeconfig file
        [kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
        [kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
        [kubelet-start] Starting the kubelet
        [control-plane] Using manifest folder "/etc/kubernetes/manifests"
        [control-plane] Creating static Pod manifest for "kube-apiserver"
        [control-plane] Creating static Pod manifest for "kube-controller-manager"
        [control-plane] Creating static Pod manifest for "kube-scheduler"
        [etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
        [wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
        [kubelet-check] Initial timeout of 40s passed.
        [kubelet-check] It seems like the kubelet isn't running or healthy.
        [kubelet-check] The HTTP call equal to 'curl -sSL http://localhost:10248/healthz' failed with error: Get "http://localhost:10248/healthz": dial tcp [::1]:10248: connect: connection refused.
        [kubelet-check] It seems like the kubelet isn't running or healthy.
        [kubelet-check] The HTTP call equal to 'curl -sSL http://localhost:10248/healthz' failed with error: Get "http://localhost:10248/healthz": dial tcp [::1]:10248: connect: connection refused.
        [kubelet-check] It seems like the kubelet isn't running or healthy.
        [kubelet-check] The HTTP call equal to 'curl -sSL http://localhost:10248/healthz' failed with error: Get "http://localhost:10248/healthz": dial tcp [::1]:10248: connect: connection refused.
        [kubelet-check] It seems like the kubelet isn't running or healthy.
        [kubelet-check] The HTTP call equal to 'curl -sSL http://localhost:10248/healthz' failed with error: Get "http://localhost:10248/healthz": dial tcp [::1]:10248: connect: connection refused.
        [kubelet-check] It seems like the kubelet isn't running or healthy.
        [kubelet-check] The HTTP call equal to 'curl -sSL http://localhost:10248/healthz' failed with error: Get "http://localhost:10248/healthz": dial tcp [::1]:10248: connect: connection refused.

                Unfortunately, an error has occurred:
                        timed out waiting for the condition

                This error is likely caused by:
                        - The kubelet is not running
                        - The kubelet is unhealthy due to a misconfiguration of the node in some way (required cgroups disabled)

                If you are on a systemd-powered system, you can try to troubleshoot the error with the following commands:
                        - 'systemctl status kubelet'
                        - 'journalctl -xeu kubelet'

                Additionally, a control plane component may have crashed or exited when started by the container runtime.
                To troubleshoot, list all containers using your preferred container runtimes CLI.

                Here is one example how you may list all Kubernetes containers running in docker:
                        - 'docker ps -a | grep kube | grep -v pause'
                        Once you have found the failing container, you can inspect its logs with:
                        - 'docker logs CONTAINERID'


        stderr:
                [WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". Please follow the guide at https://kubernetes.io/docs/setup/cri/
                [WARNING Swap]: running with swap on is not supported. Please disable swap
                [WARNING SystemVerification]: this Docker version is not on the list of validated versions: 20.10.7. Latest validated version: 19.03
                [WARNING Service-Kubelet]: kubelet service is not enabled, please run 'systemctl enable kubelet.service'
        error execution phase wait-control-plane: couldn't initialize a Kubernetes cluster
        To see the stack trace of this error execute with --v=5 or higher

            ▪ 証明書と鍵を作成しています...
            ▪ Control Plane を起動しています...

        💣  クラスタを起動中にエラーが発生しました: wait: /bin/bash -c "sudo env PATH=/var/lib/minikube/binaries/v1.20.7:$PATH kubeadm init --config /var/tmp/minikube/kubeadm.yaml  --ignore-preflight-errors=DirAvailable--etc-kubernetes-manifests,DirAvailable--var-lib-minikube,DirAvailable--var-lib-minikube-etcd,FileAvailable--etc-kubernetes-manifests-kube-scheduler.yaml,FileAvailable--etc-kubernetes-manifests-kube-apiserver.yaml,FileAvailable--etc-kubernetes-manifests-kube-controller-manager.yaml,FileAvailable--etc-kubernetes-manifests-etcd.yaml,Port-10250,Swap,Mem": exit status 1
        stdout:
        [init] Using Kubernetes version: v1.20.7
        [preflight] Running pre-flight checks
        [preflight] Pulling images required for setting up a Kubernetes cluster
        [preflight] This might take a minute or two, depending on the speed of your internet connection
        [preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
        [certs] Using certificateDir folder "/var/lib/minikube/certs"
        [certs] Using existing ca certificate authority
        [certs] Using existing apiserver certificate and key on disk
        [certs] Using existing apiserver-kubelet-client certificate and key on disk
        [certs] Using existing front-proxy-ca certificate authority
        [certs] Using existing front-proxy-client certificate and key on disk
        [certs] Using existing etcd/ca certificate authority
        [certs] Using existing etcd/server certificate and key on disk
        [certs] Using existing etcd/peer certificate and key on disk
        [certs] Using existing etcd/healthcheck-client certificate and key on disk
        [certs] Using existing apiserver-etcd-client certificate and key on disk
        [certs] Using the existing "sa" key
        [kubeconfig] Using kubeconfig folder "/etc/kubernetes"
        [kubeconfig] Writing "admin.conf" kubeconfig file
        [kubeconfig] Writing "kubelet.conf" kubeconfig file
        [kubeconfig] Writing "controller-manager.conf" kubeconfig file
        [kubeconfig] Writing "scheduler.conf" kubeconfig file
        [kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
        [kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
        [kubelet-start] Starting the kubelet
        [control-plane] Using manifest folder "/etc/kubernetes/manifests"
        [control-plane] Creating static Pod manifest for "kube-apiserver"
        [control-plane] Creating static Pod manifest for "kube-controller-manager"
        [control-plane] Creating static Pod manifest for "kube-scheduler"
        [etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
        [wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
        [kubelet-check] Initial timeout of 40s passed.
        [kubelet-check] It seems like the kubelet isn't running or healthy.
        [kubelet-check] The HTTP call equal to 'curl -sSL http://localhost:10248/healthz' failed with error: Get "http://localhost:10248/healthz": dial tcp [::1]:10248: connect: connection refused.
        [kubelet-check] It seems like the kubelet isn't running or healthy.
        [kubelet-check] The HTTP call equal to 'curl -sSL http://localhost:10248/healthz' failed with error: Get "http://localhost:10248/healthz": dial tcp [::1]:10248: connect: connection refused.

                Unfortunately, an error has occurred:
                        timed out waiting for the condition

                This error is likely caused by:
                        - The kubelet is not running
                        - The kubelet is unhealthy due to a misconfiguration of the node in some way (required cgroups disabled)

                If you are on a systemd-powered system, you can try to troubleshoot the error with the following commands:
                        - 'systemctl status kubelet'
                        - 'journalctl -xeu kubelet'

                Additionally, a control plane component may have crashed or exited when started by the container runtime.
                To troubleshoot, list all containers using your preferred container runtimes CLI.

                Here is one example how you may list all Kubernetes containers running in docker:
                        - 'docker ps -a | grep kube | grep -v pause'
                        Once you have found the failing container, you can inspect its logs with:
                        - 'docker logs CONTAINERID'


        stderr:
                [WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". Please follow the guide at https://kubernetes.io/docs/setup/cri/
                [WARNING Swap]: running with swap on is not supported. Please disable swap
                [WARNING SystemVerification]: this Docker version is not on the list of validated versions: 20.10.7. Latest validated version: 19.03
                [WARNING Service-Kubelet]: kubelet service is not enabled, please run 'systemctl enable kubelet.service'
        error execution phase wait-control-plane: couldn't initialize a Kubernetes cluster
        To see the stack trace of this error execute with --v=5 or higher


        ╭────────────────────────────────────────────────────────────────────╮
        │                                                                    │
        │    😿  If the above advice does not help, please let us know:      │
        │    👉  https://github.com/kubernetes/minikube/issues/new/choose    │
        │                                                                    │
        │    Please attach the following file to the GitHub issue:           │
        │    - /root/.minikube/logs/lastStart.txt                            │
        │                                                                    │
        ╰────────────────────────────────────────────────────────────────────╯


        ❌  Exiting due to K8S_KUBELET_NOT_RUNNING: wait: /bin/bash -c "sudo env PATH=/var/lib/minikube/binaries/v1.20.7:$PATH kubeadm init --config /var/tmp/minikube/kubeadm.yaml  --ignore-preflight-errors=DirAvailable--etc-kubernetes-manifests,DirAvailable--var-lib-minikube,DirAvailable--var-lib-minikube-etcd,FileAvailable--etc-kubernetes-manifests-kube-scheduler.yaml,FileAvailable--etc-kubernetes-manifests-kube-apiserver.yaml,FileAvailable--etc-kubernetes-manifests-kube-controller-manager.yaml,FileAvailable--etc-kubernetes-manifests-etcd.yaml,Port-10250,Swap,Mem": exit status 1
        stdout:
        [init] Using Kubernetes version: v1.20.7
        [preflight] Running pre-flight checks
        [preflight] Pulling images required for setting up a Kubernetes cluster
        [preflight] This might take a minute or two, depending on the speed of your internet connection
        [preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
        [certs] Using certificateDir folder "/var/lib/minikube/certs"
        [certs] Using existing ca certificate authority
        [certs] Using existing apiserver certificate and key on disk
        [certs] Using existing apiserver-kubelet-client certificate and key on disk
        [certs] Using existing front-proxy-ca certificate authority
        [certs] Using existing front-proxy-client certificate and key on disk
        [certs] Using existing etcd/ca certificate authority
        [certs] Using existing etcd/server certificate and key on disk
        [certs] Using existing etcd/peer certificate and key on disk
        [certs] Using existing etcd/healthcheck-client certificate and key on disk
        [certs] Using existing apiserver-etcd-client certificate and key on disk
        [certs] Using the existing "sa" key
        [kubeconfig] Using kubeconfig folder "/etc/kubernetes"
        [kubeconfig] Writing "admin.conf" kubeconfig file
        [kubeconfig] Writing "kubelet.conf" kubeconfig file
        [kubeconfig] Writing "controller-manager.conf" kubeconfig file
        [kubeconfig] Writing "scheduler.conf" kubeconfig file
        [kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
        [kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
        [kubelet-start] Starting the kubelet
        [control-plane] Using manifest folder "/etc/kubernetes/manifests"
        [control-plane] Creating static Pod manifest for "kube-apiserver"
        [control-plane] Creating static Pod manifest for "kube-controller-manager"
        [control-plane] Creating static Pod manifest for "kube-scheduler"
        [etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
        [wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
        [kubelet-check] Initial timeout of 40s passed.
        [kubelet-check] It seems like the kubelet isn't running or healthy.
        [kubelet-check] The HTTP call equal to 'curl -sSL http://localhost:10248/healthz' failed with error: Get "http://localhost:10248/healthz": dial tcp [::1]:10248: connect: connection refused.
        [kubelet-check] It seems like the kubelet isn't running or healthy.
        [kubelet-check] The HTTP call equal to 'curl -sSL http://localhost:10248/healthz' failed with error: Get "http://localhost:10248/healthz": dial tcp [::1]:10248: connect: connection refused.

                Unfortunately, an error has occurred:
                        timed out waiting for the condition

                This error is likely caused by:
                        - The kubelet is not running
                        - The kubelet is unhealthy due to a misconfiguration of the node in some way (required cgroups disabled)

                If you are on a systemd-powered system, you can try to troubleshoot the error with the following commands:
                        - 'systemctl status kubelet'
                        - 'journalctl -xeu kubelet'

                Additionally, a control plane component may have crashed or exited when started by the container runtime.
                To troubleshoot, list all containers using your preferred container runtimes CLI.

                Here is one example how you may list all Kubernetes containers running in docker:
                        - 'docker ps -a | grep kube | grep -v pause'
                        Once you have found the failing container, you can inspect its logs with:
                        - 'docker logs CONTAINERID'


        stderr:
                [WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". Please follow the guide at https://kubernetes.io/docs/setup/cri/
                [WARNING Swap]: running with swap on is not supported. Please disable swap
                [WARNING SystemVerification]: this Docker version is not on the list of validated versions: 20.10.7. Latest validated version: 19.03
                [WARNING Service-Kubelet]: kubelet service is not enabled, please run 'systemctl enable kubelet.service'
        error execution phase wait-control-plane: couldn't initialize a Kubernetes cluster
        To see the stack trace of this error execute with --v=5 or higher

        💡  提案: Check output of 'journalctl -xeu kubelet', try passing --extra-config=kubelet.cgroup-driver=systemd to minikube start
        🍿  Related issue: https://github.com/kubernetes/minikube/issues/4172

        ```

- minikube の起動が起動できなくなった

## Dockerをバージョンアップする

- 現在の環境は、Dockerをyumでインストールした。

    ```
    yum install -y \
    docker-ce-20.10.7 \
    docker-ce-cli-20.10.7 \
    containerd.io-1.4.6
    ```

- 上記を考慮して、Dockerをバージョンアップする

- インストールされているDockerのバージョンを確認する。

    ```
    # yum list installed | grep -e "docker" -e "container"
    container-selinux.noarch                    2:2.119.2-1.911c772.el7_8  @extras  
    containerd.io.x86_64                        1.4.6-3.1.el7              @docker-ce-stable
    docker-ce.x86_64                            3:20.10.7-3.el7            @docker-ce-stable
    docker-ce-cli.x86_64                        1:20.10.7-3.el7            @docker-ce-stable
    docker-ce-rootless-extras.x86_64            24.0.4-1.el7               @docker-ce-stable
    docker-scan-plugin.x86_64                   0.23.0-3.el7               @docker-ce-stable
    ```

- インストールしたパッケージをアンインストール

    ```
    # yum remove docker-ce.x86_64
    ```

    - 結果

        ```
            読み込んだプラグイン:fastestmirror, langpacks
        依存性の解決をしています
        --> トランザクションの確認を実行しています。
        ---> パッケージ docker-ce.x86_64 3:20.10.7-3.el7 を 削除
        --> 依存性の処理をしています: docker-ce のパッケージ: docker-ce-rootless-extras-24.0.4-1.el7.x86_64
        --> トランザクションの確認を実行しています。
        ---> パッケージ docker-ce-rootless-extras.x86_64 0:24.0.4-1.el7 を 削除
        --> 依存性解決を終了しました。

        依存性を解決しました

        ================================================================================================================================================================================================================
        Package                                                    アーキテクチャー                        バージョン                                         リポジトリー                                        容量
        ================================================================================================================================================================================================================
        削除中:
        docker-ce                                                  x86_64                                  3:20.10.7-3.el7                                    @docker-ce-stable                                  115 M
        依存性関連での削除をします:
        docker-ce-rootless-extras                                  x86_64                                  24.0.4-1.el7                                       @docker-ce-stable                                   19 M

        トランザクションの要約
        ================================================================================================================================================================================================================
        削除  1 パッケージ (+1 個の依存関係のパッケージ)

        インストール容量: 135 M
        上記の処理を行います。よろしいでしょうか？ [y/N] y
        Downloading packages:
        Running transaction check
        Running transaction test
        Transaction test succeeded
        Running transaction
        削除中                  : docker-ce-rootless-extras-24.0.4-1.el7.x86_64                                                                                                                                   1/2 
        削除中                  : 3:docker-ce-20.10.7-3.el7.x86_64                                                                                                                                                2/2 
        検証中                  : 3:docker-ce-20.10.7-3.el7.x86_64                                                                                                                                                1/2 
        検証中                  : docker-ce-rootless-extras-24.0.4-1.el7.x86_64                                                                                                                                   2/2 

        削除しました:
        docker-ce.x86_64 3:20.10.7-3.el7                                                                                                                                                                              

        依存性の削除をしました:
        docker-ce-rootless-extras.x86_64 0:24.0.4-1.el7                                                                                                                                                               

        完了しました!
        ```

    ```
    # yum remove docker-ce-cli.x86_64
    ```
    - 結果

        ```
        読み込んだプラグイン:fastestmirror, langpacks
        依存性の解決をしています
        --> トランザクションの確認を実行しています。
        ---> パッケージ docker-ce-cli.x86_64 1:20.10.7-3.el7 を 削除
        --> 依存性解決を終了しました。

        依存性を解決しました

        ================================================================================================================================================================================================================
        Package                                           アーキテクチャー                           バージョン                                            リポジトリー                                           容量
        ================================================================================================================================================================================================================
        削除中:
        docker-ce-cli                                     x86_64                                     1:20.10.7-3.el7                                       @docker-ce-stable                                     156 M

        トランザクションの要約
        ================================================================================================================================================================================================================
        削除  1 パッケージ

        インストール容量: 156 M
        上記の処理を行います。よろしいでしょうか？ [y/N]y
        Downloading packages:
        Running transaction check
        Running transaction test
        Transaction test succeeded
        Running transaction
        削除中                  : 1:docker-ce-cli-20.10.7-3.el7.x86_64                                                                                                                                            1/1 
        検証中                  : 1:docker-ce-cli-20.10.7-3.el7.x86_64                                                                                                                                            1/1 

        削除しました:
        docker-ce-cli.x86_64 1:20.10.7-3.el7                                                                                                                                                                          

        完了しました!
        ```

    ```
    # yum remove containerd.io.x86_64
    ```
    - 結果

    ```
    読み込んだプラグイン:fastestmirror, langpacks
    依存性の解決をしています
    --> トランザクションの確認を実行しています。
    ---> パッケージ containerd.io.x86_64 0:1.4.6-3.1.el7 を 削除
    --> 依存性解決を終了しました。

    依存性を解決しました

    ================================================================================================================================================================================================================
    Package                                            アーキテクチャー                            バージョン                                         リポジトリー                                            容量
    ================================================================================================================================================================================================================
    削除中:
    containerd.io                                      x86_64                                      1.4.6-3.1.el7                                      @docker-ce-stable                                      129 M

    トランザクションの要約
    ================================================================================================================================================================================================================
    削除  1 パッケージ

    インストール容量: 129 M
    上記の処理を行います。よろしいでしょうか？ [y/N]y
    Downloading packages:
    Running transaction check
    Running transaction test
    Transaction test succeeded
    Running transaction
    削除中                  : containerd.io-1.4.6-3.1.el7.x86_64                                                                                                                                              1/1 
    検証中                  : containerd.io-1.4.6-3.1.el7.x86_64                                                                                                                                              1/1 

    削除しました:
    containerd.io.x86_64 0:1.4.6-3.1.el7                                                                                                                                                                          

    完了しました!
    ```


- 不要な依存関係や設定ファイルが残っている可能性があるため、アンインストール後に不要なファイルや設定を削除する

    ```
    # yum autoremove
    ```

    - 結果

        ```
        読み込んだプラグイン:fastestmirror, langpacks
        依存性の解決をしています
        --> トランザクションの確認を実行しています。
        ---> パッケージ container-selinux.noarch 2:2.119.2-1.911c772.el7_8 を 削除
        ---> パッケージ docker-scan-plugin.x86_64 0:0.23.0-3.el7 を 削除
        ---> パッケージ fuse-overlayfs.x86_64 0:0.7.2-6.el7_8 を 削除
        ---> パッケージ slirp4netns.x86_64 0:0.4.3-4.el7_8 を 削除
        --> 依存性解決を終了しました。
        --> Finding unneeded leftover dependencies
        ---> Marking fuse3-libs to be removed - no longer needed by fuse-overlayfs
        Found and removing 1 unneeded dependencies
        --> トランザクションの確認を実行しています。
        ---> パッケージ fuse3-libs.x86_64 0:3.6.1-4.el7 を 削除
        --> 依存性解決を終了しました。

        依存性を解決しました

        ================================================================================================================================================================================================================
        Package                                            アーキテクチャー                       バージョン                                                   リポジトリー                                       容量
        ================================================================================================================================================================================================================
        削除中:
        container-selinux                                  noarch                                 2:2.119.2-1.911c772.el7_8                                    @extras                                            41 k
        docker-scan-plugin                                 x86_64                                 0.23.0-3.el7                                                 @docker-ce-stable                                  12 M
        fuse-overlayfs                                     x86_64                                 0.7.2-6.el7_8                                                @extras                                           116 k
        slirp4netns                                        x86_64                                 0.4.3-4.el7_8                                                @extras                                           169 k
        依存性関連での削除をします:
        fuse3-libs                                         x86_64                                 3.6.1-4.el7                                                  @extras                                           270 k

        トランザクションの要約
        ================================================================================================================================================================================================================
        削除  4 パッケージ (+1 個の依存関係のパッケージ)

        インストール容量: 13 M
        上記の処理を行います。よろしいでしょうか？ [y/N]y
        Downloading packages:
        Running transaction check
        Running transaction test
        Transaction test succeeded
        Running transaction
        削除中                  : fuse-overlayfs-0.7.2-6.el7_8.x86_64                                                                                                                                             1/5 
        削除中                  : 2:container-selinux-2.119.2-1.911c772.el7_8.noarch                                                                                                                              2/5 
        削除中                  : docker-scan-plugin-0.23.0-3.el7.x86_64                                                                                                                                          3/5 
        削除中                  : fuse3-libs-3.6.1-4.el7.x86_64                                                                                                                                                   4/5 
        削除中                  : slirp4netns-0.4.3-4.el7_8.x86_64                                                                                                                                                5/5 
        検証中                  : fuse3-libs-3.6.1-4.el7.x86_64                                                                                                                                                   1/5 
        検証中                  : fuse-overlayfs-0.7.2-6.el7_8.x86_64                                                                                                                                             2/5 
        検証中                  : docker-scan-plugin-0.23.0-3.el7.x86_64                                                                                                                                          3/5 
        検証中                  : slirp4netns-0.4.3-4.el7_8.x86_64                                                                                                                                                4/5 
        検証中                  : 2:container-selinux-2.119.2-1.911c772.el7_8.noarch                                                                                                                              5/5 

        削除しました:
        container-selinux.noarch 2:2.119.2-1.911c772.el7_8           docker-scan-plugin.x86_64 0:0.23.0-3.el7           fuse-overlayfs.x86_64 0:0.7.2-6.el7_8           slirp4netns.x86_64 0:0.4.3-4.el7_8          

        依存性の削除をしました:
        fuse3-libs.x86_64 0:3.6.1-4.el7                                                                                                                                                                               

        完了しました!
    
        ```



- アンインストールを確認する。

    ```
    # yum list installed | grep -e "docker" -e "container"
    ```
    
    - 結果 → 表示なし（正常）
        ```
        ```


- インストールするソフトの最新をバージョンを確認する

    ```
    # yum --showduplicates list | grep \
    -e "^docker-ce.x86_64" \
    -e "^docker-ce-cli.x86_64" \
    -e "^containerd.io.x86_64"
    ```

    -  結果、それぞれの最新バージョンは以下と判明

        ```
        docker-ce.x86_64                         3:24.0.5-1.el7                docker-ce-stable
        ```

        ```
        docker-ce-cli.x86_64                     1:24.0.5-1.el7                docker-ce-stable
        ```

        ```
        containerd.io.x86_64                     1.6.22-3.1.el7                docker-ce-stable
        ```

- 上記結果から、Dockerの最新版のインストールコマンドは以下となる

    ```
    # yum install -y \
    docker-ce-24.0.5 \
    docker-ce-cli-24.0.5 \
    containerd.io-1.6.22
    ```

    - 結果

        ```
        読み込んだプラグイン:fastestmirror, langpacks
        Loading mirror speeds from cached hostfile
        * base: ftp.tsukuba.wide.ad.jp
        * elrepo: ftp.ne.jp
        * extras: ftp.tsukuba.wide.ad.jp
        * updates: ftp.tsukuba.wide.ad.jp
        依存性の解決をしています
        --> トランザクションの確認を実行しています。
        ---> パッケージ containerd.io.x86_64 0:1.6.22-3.1.el7 を インストール
        --> 依存性の処理をしています: container-selinux >= 2:2.74 のパッケージ: containerd.io-1.6.22-3.1.el7.x86_64
        ---> パッケージ docker-ce.x86_64 3:24.0.5-1.el7 を インストール
        --> 依存性の処理をしています: docker-ce-rootless-extras のパッケージ: 3:docker-ce-24.0.5-1.el7.x86_64
        ---> パッケージ docker-ce-cli.x86_64 1:24.0.5-1.el7 を インストール
        --> 依存性の処理をしています: docker-buildx-plugin のパッケージ: 1:docker-ce-cli-24.0.5-1.el7.x86_64
        --> 依存性の処理をしています: docker-compose-plugin のパッケージ: 1:docker-ce-cli-24.0.5-1.el7.x86_64
        --> トランザクションの確認を実行しています。
        ---> パッケージ container-selinux.noarch 2:2.119.2-1.911c772.el7_8 を インストール
        ---> パッケージ docker-buildx-plugin.x86_64 0:0.11.2-1.el7 を インストール
        ---> パッケージ docker-ce-rootless-extras.x86_64 0:24.0.5-1.el7 を インストール
        --> 依存性の処理をしています: fuse-overlayfs >= 0.7 のパッケージ: docker-ce-rootless-extras-24.0.5-1.el7.x86_64
        --> 依存性の処理をしています: slirp4netns >= 0.4 のパッケージ: docker-ce-rootless-extras-24.0.5-1.el7.x86_64
        ---> パッケージ docker-compose-plugin.x86_64 0:2.20.2-1.el7 を インストール
        --> トランザクションの確認を実行しています。
        ---> パッケージ fuse-overlayfs.x86_64 0:0.7.2-6.el7_8 を インストール
        --> 依存性の処理をしています: libfuse3.so.3(FUSE_3.2)(64bit) のパッケージ: fuse-overlayfs-0.7.2-6.el7_8.x86_64
        --> 依存性の処理をしています: libfuse3.so.3(FUSE_3.0)(64bit) のパッケージ: fuse-overlayfs-0.7.2-6.el7_8.x86_64
        --> 依存性の処理をしています: libfuse3.so.3()(64bit) のパッケージ: fuse-overlayfs-0.7.2-6.el7_8.x86_64
        ---> パッケージ slirp4netns.x86_64 0:0.4.3-4.el7_8 を インストール
        --> トランザクションの確認を実行しています。
        ---> パッケージ fuse3-libs.x86_64 0:3.6.1-4.el7 を インストール
        --> 依存性解決を終了しました。

        依存性を解決しました

        ================================================================================================================================================================================================================
        Package                                                  アーキテクチャー                      バージョン                                                リポジトリー                                     容量
        ================================================================================================================================================================================================================
        インストール中:
        containerd.io                                            x86_64                                1.6.22-3.1.el7                                            docker-ce-stable                                 34 M
        docker-ce                                                x86_64                                3:24.0.5-1.el7                                            docker-ce-stable                                 24 M
        docker-ce-cli                                            x86_64                                1:24.0.5-1.el7                                            docker-ce-stable                                 13 M
        依存性関連でのインストールをします:
        container-selinux                                        noarch                                2:2.119.2-1.911c772.el7_8                                 extras                                           40 k
        docker-buildx-plugin                                     x86_64                                0.11.2-1.el7                                              docker-ce-stable                                 13 M
        docker-ce-rootless-extras                                x86_64                                24.0.5-1.el7                                              docker-ce-stable                                9.1 M
        docker-compose-plugin                                    x86_64                                2.20.2-1.el7                                              docker-ce-stable                                 13 M
        fuse-overlayfs                                           x86_64                                0.7.2-6.el7_8                                             extras                                           54 k
        fuse3-libs                                               x86_64                                3.6.1-4.el7                                               extras                                           82 k
        slirp4netns                                              x86_64                                0.4.3-4.el7_8                                             extras                                           81 k

        トランザクションの要約
        ================================================================================================================================================================================================================
        インストール  3 パッケージ (+7 個の依存関係のパッケージ)

        合計容量: 107 M
        総ダウンロード容量: 13 M
        インストール容量: 383 M
        Downloading packages:
        No Presto metadata available for docker-ce-stable
        (1/5): container-selinux-2.119.2-1.911c772.el7_8.noarch.rpm                                                                                                                              |  40 kB  00:00:00     
        (2/5): fuse-overlayfs-0.7.2-6.el7_8.x86_64.rpm                                                                                                                                           |  54 kB  00:00:00     
        (3/5): fuse3-libs-3.6.1-4.el7.x86_64.rpm                                                                                                                                                 |  82 kB  00:00:00     
        (4/5): slirp4netns-0.4.3-4.el7_8.x86_64.rpm                                                                                                                                              |  81 kB  00:00:00     
        (5/5): docker-compose-plugin-2.20.2-1.el7.x86_64.rpm                                                                                                                                     |  13 MB  00:00:00     
        ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
        合計                                                                                                                                                                             15 MB/s |  13 MB  00:00:00     
        Running transaction check
        Running transaction test
        Transaction test succeeded
        Running transaction
        インストール中          : 2:container-selinux-2.119.2-1.911c772.el7_8.noarch                                                                                                                             1/10 
        インストール中          : containerd.io-1.6.22-3.1.el7.x86_64                                                                                                                                            2/10 
        インストール中          : docker-buildx-plugin-0.11.2-1.el7.x86_64                                                                                                                                       3/10 
        インストール中          : docker-compose-plugin-2.20.2-1.el7.x86_64                                                                                                                                      4/10 
        インストール中          : 1:docker-ce-cli-24.0.5-1.el7.x86_64                                                                                                                                            5/10 
        インストール中          : slirp4netns-0.4.3-4.el7_8.x86_64                                                                                                                                               6/10 
        インストール中          : fuse3-libs-3.6.1-4.el7.x86_64                                                                                                                                                  7/10 
        インストール中          : fuse-overlayfs-0.7.2-6.el7_8.x86_64                                                                                                                                            8/10 
        インストール中          : docker-ce-rootless-extras-24.0.5-1.el7.x86_64                                                                                                                                  9/10 
        インストール中          : 3:docker-ce-24.0.5-1.el7.x86_64                                                                                                                                               10/10 
        検証中                  : 3:docker-ce-24.0.5-1.el7.x86_64                                                                                                                                                1/10 
        検証中                  : fuse3-libs-3.6.1-4.el7.x86_64                                                                                                                                                  2/10 
        検証中                  : fuse-overlayfs-0.7.2-6.el7_8.x86_64                                                                                                                                            3/10 
        検証中                  : slirp4netns-0.4.3-4.el7_8.x86_64                                                                                                                                               4/10 
        検証中                  : 2:container-selinux-2.119.2-1.911c772.el7_8.noarch                                                                                                                             5/10 
        検証中                  : docker-compose-plugin-2.20.2-1.el7.x86_64                                                                                                                                      6/10 
        検証中                  : 1:docker-ce-cli-24.0.5-1.el7.x86_64                                                                                                                                            7/10 
        検証中                  : containerd.io-1.6.22-3.1.el7.x86_64                                                                                                                                            8/10 
        検証中                  : docker-buildx-plugin-0.11.2-1.el7.x86_64                                                                                                                                       9/10 
        検証中                  : docker-ce-rootless-extras-24.0.5-1.el7.x86_64                                                                                                                                 10/10 

        インストール:
        containerd.io.x86_64 0:1.6.22-3.1.el7                                  docker-ce.x86_64 3:24.0.5-1.el7                                  docker-ce-cli.x86_64 1:24.0.5-1.el7                                 

        依存性関連をインストールしました:
        container-selinux.noarch 2:2.119.2-1.911c772.el7_8      docker-buildx-plugin.x86_64 0:0.11.2-1.el7      docker-ce-rootless-extras.x86_64 0:24.0.5-1.el7      docker-compose-plugin.x86_64 0:2.20.2-1.el7     
        fuse-overlayfs.x86_64 0:0.7.2-6.el7_8                   fuse3-libs.x86_64 0:3.6.1-4.el7                 slirp4netns.x86_64 0:0.4.3-4.el7_8                  

        完了しました!                
        ```

- インストール完了の確認。

    ```
    # yum list installed | grep -e "docker" -e "container"
    ```

    - 結果

        ```
        container-selinux.noarch                    2:2.119.2-1.911c772.el7_8  @extras  
        containerd.io.x86_64                        1.6.22-3.1.el7             @docker-ce-stable
        docker-buildx-plugin.x86_64                 0.11.2-1.el7               @docker-ce-stable
        docker-ce.x86_64                            3:24.0.5-1.el7             @docker-ce-stable
        docker-ce-cli.x86_64                        1:24.0.5-1.el7             @docker-ce-stable
        docker-ce-rootless-extras.x86_64            24.0.5-1.el7               @docker-ce-stable
        docker-compose-plugin.x86_64                2.20.2-1.el7               @docker-ce-stable
        ```

- バージョンの確認


- Dockerを起動

    ```
    # systemctl enable docker
    ```
    
    - 結果

        ```
        Created symlink from /etc/systemd/system/multi-user.target.wants/docker.service to /usr/lib/systemd/system/docker.service.
        ```
    
    ```
    # systemctl start docker
    ```

    - 結果 → なし

        ```
        ```

    ```
    # systemctl status docker    
    ```

    - 結果 → なし

        ```
        ● docker.service - Docker Application Container Engine
        Loaded: loaded (/usr/lib/systemd/system/docker.service; enabled; vendor preset: disabled)
        Active: active (running) since 火 2023-08-15 15:53:59 JST; 22s ago
            Docs: https://docs.docker.com
        Main PID: 9662 (dockerd)
            Tasks: 27
        Memory: 51.8M
        CGroup: /system.slice/docker.service
                ├─9662 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
                └─9855 /usr/bin/docker-proxy -proto tcp -host-ip 127.0.0.1 -host-port 1514 -container-ip 172.18.0.2 -container-port 10514

        8月 15 15:54:00 control-plane.minikube.internal dockerd[9662]: time="2023-08-15T15:54:00.246152651+09:00" level=error msg="Handler for GET /v1.40/images/json returned error: context canceled"
        8月 15 15:54:00 control-plane.minikube.internal dockerd[9662]: time="2023-08-15T15:54:00.298848720+09:00" level=error msg="Handler for GET /v1.40/images/json returned error: context canceled"
        8月 15 15:54:00 control-plane.minikube.internal dockerd[9662]: time="2023-08-15T15:54:00.316241741+09:00" level=error msg="Handler for GET /v1.40/images/json returned error: context canceled"
        8月 15 15:54:00 control-plane.minikube.internal dockerd[9662]: time="2023-08-15T15:54:00.849098968+09:00" level=info msg="ignoring event" container=bb352ac48a4df31f9cf304407810d3ec45c92fb462f....TaskDelete"
        8月 15 15:54:01 control-plane.minikube.internal dockerd[9662]: time="2023-08-15T15:54:01.528737820+09:00" level=info msg="ignoring event" container=bb352ac48a4df31f9cf304407810d3ec45c92fb462f....TaskDelete"
        8月 15 15:54:02 control-plane.minikube.internal dockerd[9662]: time="2023-08-15T15:54:02.321787066+09:00" level=info msg="ignoring event" container=bb352ac48a4df31f9cf304407810d3ec45c92fb462f....TaskDelete"
        8月 15 15:54:03 control-plane.minikube.internal dockerd[9662]: time="2023-08-15T15:54:03.524621477+09:00" level=info msg="ignoring event" container=bb352ac48a4df31f9cf304407810d3ec45c92fb462f....TaskDelete"
        8月 15 15:54:05 control-plane.minikube.internal dockerd[9662]: time="2023-08-15T15:54:05.541911783+09:00" level=info msg="ignoring event" container=bb352ac48a4df31f9cf304407810d3ec45c92fb462f....TaskDelete"
        8月 15 15:54:09 control-plane.minikube.internal dockerd[9662]: time="2023-08-15T15:54:09.398180749+09:00" level=info msg="ignoring event" container=bb352ac48a4df31f9cf304407810d3ec45c92fb462f....TaskDelete"
        8月 15 15:54:16 control-plane.minikube.internal dockerd[9662]: time="2023-08-15T15:54:16.229850221+09:00" level=info msg="ignoring event" container=bb352ac48a4df31f9cf304407810d3ec45c92fb462f....TaskDelete"
        Hint: Some lines were ellipsized, use -l to show in full.
        ```

- バージョンを確認

    ```
    # docker version
    Client: Docker Engine - Community
    Version:           24.0.5
    API version:       1.43
    Go version:        go1.20.6
    Git commit:        ced0996
    Built:             Fri Jul 21 20:39:02 2023
    OS/Arch:           linux/amd64
    Context:           default

    Server: Docker Engine - Community
    Engine:
    Version:          24.0.5
    API version:      1.43 (minimum version 1.12)
    Go version:       go1.20.6
    Git commit:       a61e2b4
    Built:            Fri Jul 21 20:38:05 2023
    OS/Arch:          linux/amd64
    Experimental:     false
    containerd:
    Version:          1.6.22
    GitCommit:        8165feabfdfe38c65b599c4993d227328c231fca
    runc:
    Version:          1.1.8
    GitCommit:        v1.1.8-0-g82f18fe
    docker-init:
    Version:          0.19.0
    GitCommit:        de40ad0
    ```



## kubectl をバージョンアップする

- [公式ドキュメント](https://kubernetes.io/ja/docs/tasks/tools/install-kubectl/)を参考にして、kubctlをバージョンアップする。

- kubectl の実行ファイルの保存先を確認

    ```
    # ll /usr/local/bin/
    合計 210980
    -rwxr-xr-x. 1 root root 59383631  8月 13 23:21 docker-compose
    -rwxr-xr-x. 1 root root 46182400  8月 13 07:47 helm
    -rwxr-xr-x. 1 root root 46419968  7月 17 12:13 kubectl
    -rwxr-xr-x. 1 root root 64057293  7月 17 12:13 minikube    
    ```

- 以下のコマンドを実行し、kubctlをバージョンアップする。

    ```
    cd ~
    ```

    ```
    rm -f  /usr/local/bin/kubectl
    ```

    ```
    # ll /usr/local/bin/kubectl
    ls: /usr/local/bin/kubectl にアクセスできません: そのようなファイルやディレクトリはありません
    ```

    - [公式ドキュメント](https://kubernetes.io/ja/docs/tasks/tools/install-kubectl/)より、最新のバージョンのダウンロートコマンドは以下


    ```
    curl -LO "https://dl.k8s.io/release/$(curl -LS https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
    ```

    ```
    # ls ./kubectl
    ./kubectl
    ```
        
    ```
    # chmod +x ./kubectl
    ```

    ```
    # # ll ./kubectl
    -rwxr-xr-x. 1 root root 49262592  8月 15 13:39 ./kubectl
    ```

    ```
    mv -f ./kubectl /usr/local/bin
    ```

    ```
    # ll /usr/local/bin/kubectl
    -rwxr-xr-x. 1 root root 49262592  8月 15 13:39 /usr/local/bin/kubectl
    ```

    ```
    # kubectl version --client
    WARNING: This version information is deprecated and will be replaced with the output from kubectl version --short.  Use --output=yaml|json to get the full version.
    Client Version: version.Info{Major:"1", Minor:"27", GitVersion:"v1.27.4", GitCommit:"fa3d7990104d7c1f16943a67f11b154b71f6a132", GitTreeState:"clean", BuildDate:"2023-07-19T12:20:54Z", GoVersion:"go1.20.6", Compiler:"gc", Platform:"linux/amd64"}
    Kustomize Version: v5.0.1
    ```

## minikubeをバージョンアップする

- [公式ドキュメント](https://minikube.sigs.k8s.io/docs/start/)を参考にして、minikubeをインストールする。

- minikube の実行ファイルの保存先を確認

    ```
    # ll /usr/local/bin/
    合計 210980
    -rwxr-xr-x. 1 root root 59383631  8月 13 23:21 docker-compose
    -rwxr-xr-x. 1 root root 46182400  8月 13 07:47 helm
    -rwxr-xr-x. 1 root root 46419968  7月 17 12:13 kubectl
    -rwxr-xr-x. 1 root root 64057293  7月 17 12:13 minikube    
    ```

- 以下のコマンドを実行し、minikubeをバージョンアップする。

    ```
    # cd ~
    ```

    ```
    # rm -f  /usr/local/bin/minikube
    ```

    ```
    # ls /usr/local/bin/minikube
    ls: /usr/local/bin/minikube にアクセスできません: そのようなファイルやディレクトリはありません
    ```

    - [公式ドキュメント](https://minikube.sigs.k8s.io/docs/start/)をより、minikubeの最新バージョンのダウンロードコマンドは以下。

    ```
    # curl -Lo minikube  https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
    ```

    - 結果

        ```
        % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                        Dload  Upload   Total   Spent    Left  Speed
        100 82.4M  100 82.4M    0     0  16.4M      0  0:00:05  0:00:05 --:--:-- 20.0M
        ```

    ```
    # ls ./minikube
    ./minikube
    ```
    
    ```
    # chmod +x ./minikube
    ```

    ```
    # ll ./minikube
    -rwxr-xr-x. 1 root root 86430510  8月 15 17:18 ./minikube
    ```

    ```
    # install ./minikube /usr/local/bin
    ```
 
    ```
    # ll /usr/local/bin/minikube
    -rwxr-xr-x. 1 root root 86430510  8月 15 17:24 /usr/local/bin/minikube
    ```


    ```
    # rm -f ./minikube
    ```

    ```
    # ls ./minikube
    ls: ./minikube にアクセスできません: そのようなファイルやディレクトリはありません
    ```

- バージョンを確認

    ```
    # minikube version 
    minikube version: v1.31.1
    commit: fd3f3801765d093a485d255043149f92ec0a695f
    ```

- minikube を再起動

    - ステータスを確認

        ```
        # minikube version 
        minikube version: v1.31.1
        commit: fd3f3801765d093a485d255043149f92ec0a695f
        ```

    - minikube を ストップ

        ```
        # minikube stop 
        ✋  「minikube」ノードを停止しています...
        🛑  1 台のノードが停止しました。        
        ```

    - minikube を 削除

        ```
        # minikube delete 
        🔄  kubeadm を使用して Kubernetes v1.20.7 をアンインストールしています...
        🔥  none の「minikube」を削除しています...
        💀  クラスター「minikube」の全てのトレースを削除しました。        
        ```

    - minikube を起動

    ```
    # minikube start --vm-driver=none
    😄  Centos 7.9.2009 (hyperv/amd64) 上の minikube v1.31.1
    ✨  ユーザーの設定に基づいて none ドライバーを使用します

    ❌  GUEST_MISSING_CONNTRACK が原因で終了します: 申し訳ありませんが、Kubernetes 1.27.3 は root アカウントのパス中にインストールされた crictl が必要です
    ```

## crictl のインストール

- [GitHub](https://github.com/kubernetes-sigs/cri-tools)を参考にした

-以下のコマンドを実行

```
# cd ~
```

```
# VERSION="v1.28.0"
```

```
# wget https://github.com/kubernetes-sigs/cri-tools/releases/download/$VERSION/crictl-$VERSION-linux-amd64.tar.gz
```

```
# tar zxvf crictl-$VERSION-linux-amd64.tar.gz -C /usr/local/bin
```

```
# ll /usr/local/bin/crictl 
-rwxr-xr-x. 1 kazuhiro users 54939628  8月 14 16:10 /usr/local/bin/crictl
```

```
# rm -f crictl-$VERSION-linux-amd64.tar.g
```

```
# crictl -v
crictl version v1.28.0
```

## minikubeを起動

```
# minikube start --vm-driver=none
😄  Centos 7.9.2009 (hyperv/amd64) 上の minikube v1.31.1
✨  ユーザーの設定に基づいて none ドライバーを使用します

🧯  要求された 2200MiB のメモリー割当は、システムのオーバーヘッド (合計システムメモリー: 2909MiB) に十分な空きを残しません。安定性の問題に直面するかも知れません。
💡  提案: Start minikube with less memory allocated: 'minikube start --memory=2200mb'

👍  minikube クラスター中のコントロールプレーンの minikube ノードを起動しています
🤹  localhost (CPU=2、メモリー=2909MB、ディスク=48522MB) 上で実行しています...

🐳  NOT_FOUND_CRI_DOCKERD が原因で終了します: 

💡  提案: 

    Kubernetes v1.24+ の none ドライバーと docker container-runtime は cri-dockerd を要求します。
    
    これらの手順を参照して cri-dockerd をインストールしてください:
    
    https://github.com/Mirantis/cri-dockerd

```

## cri-dockerd をインストール

- 以下のサイトに従ってcri-dockerd をインストール

    https://github.com/Mirantis/cri-dockerd


-- 以下のコマンドを実行

    ```
    # cd ~
    ```

    ```
    # git clone https://github.com/Mirantis/cri-dockerd.git
    ```

### ビルド環境の構築

- 以下にサイトの手順に従って、ビルド環境を構築

    https://go.dev/doc/install


- 以下のサイトに従って go1.21.0.linux-amd64.tar.gz をダウンロードする

    https://go.dev/doc/install


- linux サーバの 「~」 にアップ


- 以下のコマンドを実行


    1. Remove any previous Go installation

        ```
        # rm -rf /usr/local/go && tar -C /usr/local -xzf go1.21.0.linux-amd64.tar.gz
        ```

        ```
        # ll /usr/local/go
        ```

    1. Add /usr/local/go/bin to the PATH environment variable

        ```
        # export PATH=$PATH:/usr/local/go/bin
        ```

        ```
        # echo $PATH
        /root/.krew/bin:/root/.vscode-server/bin/6c3e3dba23e8fadc360aed75ce363ba185c49794/bin/remote-cli:/root/.krew/bin:/root/.krew/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin:/usr/local/go/bin
        ```

    3. Verify that you've installed Go by opening a command prompt and typing the following command

        ```
        # go version
        go version go1.21.0 linux/amd64
        ```

### cri-dockerd のインストール

- cri-dockerd を ビルド

    ```
    # cd cri-dockerd
    ```
    ```
    # make cri-dockerd
    ```
    
    - 結果

        ```
        GOARCH= go build -trimpath -ldflags "-s -w -buildid=`git log -1 --pretty='%h'` -X github.com/Mirantis/cri-dockerd/cmd/version.Version=0.3.4 -X github.com/Mirantis/cri-dockerd/cmd/version.PreRelease=`grep -q dev <<< "0.3.4" && echo "pre" || echo ""` -X github.com/Mirantis/cri-dockerd/cmd/version.GitCommit=`git log -1 --pretty='%h'`" -o cri-dockerd
        ```

- cri-dockerd を インストール

    ```
    # cd cri-dockerd
    ```

~~mkdir -p /usr/local/bin~~

- cri-dockerd を インストール

q
    ```
    # install -o root -g root -m 0755 cri-dockerd /usr/local/bin/cri-dockerd
    ```

    ```
    # install packaging/systemd/* /etc/systemd/system
    ```

    ```
    # sed -i -e 's,/usr/bin/cri-dockerd,/usr/local/bin/cri-dockerd,' /etc/systemd/system/cri-docker.service
    ```

    ```
    # systemctl daemon-reload
    ```

    ```
    # systemctl enable --now cri-docker.socket
    ```

    ```
    # systemctl status cri-docker.socket
    ● cri-docker.socket - CRI Docker Socket for the API
    Loaded: loaded (/etc/systemd/system/cri-docker.socket; enabled; vendor preset: disabled)
    Active: active (listening) since 火 2023-08-15 18:54:57 JST; 33s ago
    Listen: /run/cri-dockerd.sock (Stream)

    8月 15 18:54:57 control-plane.minikube.internal systemd[1]: Starting CRI Docker Socket for the API.
    8月 15 18:54:57 control-plane.minikube.internal systemd[1]: Listening on CRI Docker Socket for the API.
    ```

## minikubeを起動

```
# minikube start --vm-driver=none
😄  Centos 7.9.2009 (hyperv/amd64) 上の minikube v1.31.1
✨  既存のプロファイルを元に、none ドライバーを使用します

🧯  要求された 2200MiB のメモリー割当は、システムのオーバーヘッド (合計システムメモリー: 2909MiB) に十分な空きを残しません。安定性の問題に直面するかも知れません。
💡  提案: Start minikube with less memory allocated: 'minikube start --memory=2200mb'

👍  minikube クラスター中のコントロールプレーンの minikube ノードを起動しています
🔄  「minikube」のために既存の none bare metal machine を再起動しています...

🔗  NOT_FOUND_CNI_PLUGINS が原因で終了します: 


💡  提案: 

    The none driver with Kubernetes v1.24+ requires containernetworking-plugins.
    
    Please install containernetworking-plugins using these instructions:
    
    https://minikube.sigs.k8s.io/docs/faq/#how-do-i-install-containernetworking-plugins-for-none-driver
```

## containernetworking-plugins のインストール

- 以下のサイトに従って実施、

    https://minikube.sigs.k8s.io/docs/faq/#how-do-i-install-containernetworking-plugins-for-none-driver


- 以下のサイトより
    
    https://github.com/containernetworking/plugins/releases

 version_here = 1.3.0   

~~CNI_PLUGIN_VERSION="<version_here>"~~

```
# CNI_PLUGIN_VERSION="v1.3.0"
```

~~CNI_PLUGIN_TAR="cni-plugins-linux-amd64-$CNI_PLUGIN_VERSION.tgz" # change arch if not on amd64~~

```
# CNI_PLUGIN_TAR="cni-plugins-linux-amd64-$CNI_PLUGIN_VERSION.tgz"
```

```
# CNI_PLUGIN_INSTALL_DIR="/opt/cni/bin"
```

```
# curl -LO "https://github.com/containernetworking/plugins/releases/download/$CNI_PLUGIN_VERSION/$CNI_PLUGIN_TAR"
```
```
# mkdir -p "$CNI_PLUGIN_INSTALL_DIR"
```
```
# tar -xf "$CNI_PLUGIN_TAR" -C "$CNI_PLUGIN_INSTALL_DIR"
```
```
# rm "$CNI_PLUGIN_TAR"
```



    - [公式ドキュメント](https://minikube.sigs.k8s.io/docs/start/) のダウンロードリンクは正常動作しないので、以下のコマンドでダウンロードを行った。(保存先が /root であるため？)

    ```
    # curl -SL https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 -o ~
    ```

    ```
    # ls ~/minikube-linux-amd64
    ```

    ```
    # chmod +x ~/minikube-linux-amd64
    ```

    ```
    # install ~/minikube-linux-amd64 /usr/local/bin
    ```

    ```
    # rm -f ~/minikube-linux-amd64
    ```


## minikubeを起動

- 1回目の起動では以下のような結果となった

    ```
    # minikube start --vm-driver=none
    😄  Centos 7.9.2009 (hyperv/amd64) 上の minikube v1.31.1
    ✨  既存のプロファイルを元に、none ドライバーを使用します

    🧯  要求された 2200MiB のメモリー割当は、システムのオーバーヘッド (合計システムメモリー: 2909MiB) に十分な空きを残しません。安定性の問題に直面するかも知れません。
    💡  提案: Start minikube with less memory allocated: 'minikube start --memory=2200mb'

    👍  minikube クラスター中のコントロールプレーンの minikube ノードを起動しています
    🔄  「minikube」のために既存の none bare metal machine を再起動しています...
    ℹ️  OS リリースは CentOS Linux 7 (Core) です
    E0815 19:32:02.104858    4580 start.go:415] unable to disable preinstalled bridge CNI(s): failed to disable all bridge cni configs in "/etc/cni/net.d": sudo find /etc/cni/net.d -maxdepth 1 -type f ( ( -name *bridge* -or -name *podman* ) -and -not -name *.mk_disabled ) -printf "%p, " -exec sh -c "sudo mv {} {}.mk_disabled" ;: exit status 1
    stdout:

    stderr:
    find: ‘/etc/cni/net.d’: そのようなファイルやディレクトリはありません
        > kubelet.sha256:  64 B / 64 B [-------------------------] 100.00% ? p/s 0s
        > kubectl.sha256:  64 B / 64 B [-------------------------] 100.00% ? p/s 0s
        > kubeadm.sha256:  64 B / 64 B [-------------------------] 100.00% ? p/s 0s
        > kubectl:  46.98 MiB / 46.98 MiB [-------------] 100.00% 9.60 MiB p/s 5.1s
        > kubeadm:  45.93 MiB / 45.93 MiB [--------------] 100.00% 2.55 MiB p/s 18s
        > kubelet:  101.24 MiB / 101.24 MiB [------------] 100.00% 2.16 MiB p/s 47s

        ▪ 証明書と鍵を作成しています...
        ▪ コントロールプレーンを起動しています...
        ▪ RBAC のルールを設定中です...
    🔗  bridge CNI (コンテナーネットワークインターフェース) を設定中です...
    🤹  ローカルホスト環境を設定中です...

    ❗  'none' ドライバーは既存 VM の統合が必要なエキスパートに向けて設計されています。
    💡  多くのユーザーはより新しい 'docker' ドライバーを代わりに使用すべきです (root 権限が必要ありません！)
    📘  追加の詳細情報はこちらを参照してください: https://minikube.sigs.k8s.io/docs/reference/drivers/none/

    ❗  kubectl と minikube の構成は /root に保存されます
    ❗  kubectl か minikube コマンドを独自のユーザーとして使用するためには、そのコマンドの再配置が必要な場合があります。たとえば、独自の設定を上書きするためには、以下を実行します

        ▪ sudo mv /root/.kube /root/.minikube $HOME
        ▪ sudo chown -R $USER $HOME/.kube $HOME/.minikube

    💡  これは環境変数 CHANGE_MINIKUBE_NONE_USER=true を設定して自動的に行うこともできます
    🔎  Kubernetes コンポーネントを検証しています...
        ▪ gcr.io/k8s-minikube/storage-provisioner:v5 イメージを使用しています
    🌟  有効なアドオン: default-storageclass, storage-provisioner
    🏄  終了しました！kubectl がデフォルトで「minikube」クラスターと「default」ネームスペースを使用するよう設定されました``````
    ```


- 以下のエラーの解消を試みたが、対策が分からなかった。

    ```
    E0815 19:32:02.104858    4580 start.go:415] unable to disable preinstalled bridge CNI(s): failed to disable all bridge cni configs in "/etc/cni/net.d": sudo find /etc/cni/net.d -maxdepth 1 -type f ( ( -name *bridge* -or -name *podman* ) -and -not -name *.mk_disabled ) -printf "%p, " -exec sh -c "sudo mv {} {}.mk_disabled" ;: exit status 1
     ```

- 仕方なく、minikube を一旦削除し、再度起動したとところ、`E0815 19:32:02.104858` のエラーは以下のように消えた。

    ```
    # minikube delete 
    🔄  kubeadm を使用して Kubernetes v1.27.3 をアンインストールしています...
    🔥  none の「minikube」を削除しています...
    💀  クラスター「minikube」の全てのトレースを削除しました。
    ```

    2回目の起動結果は、以下
    ```
    # minikube start --vm-driver=none
    😄  Centos 7.9.2009 (hyperv/amd64) 上の minikube v1.31.1
    ✨  ユーザーの設定に基づいて none ドライバーを使用します

    🧯  要求された 2200MiB のメモリー割当は、システムのオーバーヘッド (合計システムメモリー: 2909MiB) に十分な空きを残しません。安定性の問題に直面するかも知れません。
    💡  提案: Start minikube with less memory allocated: 'minikube start --memory=2200mb'

    👍  minikube クラスター中のコントロールプレーンの minikube ノードを起動しています
    🤹  localhost (CPU=2、メモリー=2909MB、ディスク=48522MB) 上で実行しています...
    ℹ️  OS リリースは CentOS Linux 7 (Core) です
    🐳  Docker 24.0.5 で Kubernetes v1.27.3 を準備しています...
        ▪ 証明書と鍵を作成しています...
        ▪ コントロールプレーンを起動しています...
        ▪ RBAC のルールを設定中です...
    🔗  bridge CNI (コンテナーネットワークインターフェース) を設定中です...
    🤹  ローカルホスト環境を設定中です...

    ❗  'none' ドライバーは既存 VM の統合が必要なエキスパートに向けて設計されています。
    💡  多くのユーザーはより新しい 'docker' ドライバーを代わりに使用すべきです (root 権限が必要ありません！)
    📘  追加の詳細情報はこちらを参照してください: https://minikube.sigs.k8s.io/docs/reference/drivers/none/

    ❗  kubectl と minikube の構成は /root に保存されます
    ❗  kubectl か minikube コマンドを独自のユーザーとして使用するためには、そのコマンドの再配置が必要な場合があります。たとえば、独自の設定を上書きするためには、以下を実行します

        ▪ sudo mv /root/.kube /root/.minikube $HOME
        ▪ sudo chown -R $USER $HOME/.kube $HOME/.minikube

    💡  これは環境変数 CHANGE_MINIKUBE_NONE_USER=true を設定して自動的に行うこともできます
    🔎  Kubernetes コンポーネントを検証しています...
        ▪ gcr.io/k8s-minikube/storage-provisioner:v5 イメージを使用しています
    🌟  有効なアドオン: default-storageclass, storage-provisioner
    🏄  終了しました！kubectl がデフォルトで「minikube」クラスターと「default」ネームスペースを使用するよう設定されました
    ```

- とりあえずこれで使ってみる。

## docker-compose のバージョン

docker-compose は 2023-08-15現在の最新のため、このままとする

```
#  docker-compose version
Docker Compose version v2.20.3
```

## Concourse CI のインストール

-  Concourse CI のインストール手順に従い、インストールする

    https://note.com/shift_tech/n/nead03c02b095


- 以下を実行

    ```
    # cd ~
    ```

    ```
    # git clone https://github.com/concourse/concourse-docker
    ```

    - docker-compose.yml をコピー

        ```
        # ll ~/ConcourseCI-install
        ```

        ```
        # cp ../concourse-docker/docker-compose.yml .
        ```
    
    - keyフォルダをdocker-compose.yml と同じフォルダにコピー

        ```
        # cp -r  ~/concourse-docker/keys/    ~/ConcourseCI-install/
        ```
        ```
        # ll ~/ConcourseCI-install
        合計 96
        -rw-r--r--. 1 root root 92699  8月 15 20:33 README.md
        -rw-r--r--. 1 root root  1006  8月 15 20:26 docker-compose.yml
        drwxr-xr-x. 4 root root    47  8月 15 20:32 keys
        ```

        ```
        # ll ~/ConcourseCI-install/keys/
        合計 4
        -rwxr-xr-x. 1 root root 617  8月 15 20:32 generate
        drwxr-xr-x. 2 root root  22  8月 15 20:32 web
        drwxr-xr-x. 2 root root  22  8月 15 20:32 worker
        ```

    -  generate を実行

        ``` 
        #  ~/ConcourseCI-install/keys/generate 
        Unable to find image 'concourse/concourse:latest' locally
        latest: Pulling from concourse/concourse
        f7cdc50d9449: Pull complete 
        f976864fe298: Pull complete 
        ea8f02e90e81: Pull complete 
        64409b1e865b: Pull complete 
        4a13e817f670: Pull complete 
        a809f4c15750: Pull complete 
        c62956abf66d: Pull complete 
        585225b84b15: Pull complete 
        d9f6653b1de9: Pull complete 
        Digest: sha256:d35fea8f50f6b1400a1e39e23701582c18f84d558f5b631d33c4e0932d03755d
        Status: Downloaded newer image for concourse/concourse:latest
        wrote private key to /keys/session_signing_key
        wrote private key to /keys/tsa_host_key
        wrote ssh public key to /keys/tsa_host_key.pub
        wrote private key to /keys/worker_key
        wrote ssh public key to /keys/worker_key.pub
        ```

    - ここまででConcourse CIのインストールの準備は完了です。

##  Concourse CI を起動

1. 以下のコマンドを実行

    ```
    # cd ~/ConcourseCI-install
    ```
    ```
    # docker-compose up -d
    ```
    - 結果

        ```
        [+] Running 14/14
        ✔ db 13 layers [⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿]      0B/0B      Pulled                                                                                                                                                                                           22.2s 
        ✔ 648e0aadf75a Already exists                                                                                                                                                                                                                  0.0s 
        ✔ f715c8c55756 Pull complete                                                                                                                                                                                                                   0.7s 
        ✔ b11a1dc32c8c Pull complete                                                                                                                                                                                                                   0.9s 
        ✔ f29e8ba9d17c Pull complete                                                                                                                                                                                                                   0.8s 
        ✔ 78af88a8afb0 Pull complete                                                                                                                                                                                                                   1.8s 
        ✔ b74279c188d9 Pull complete                                                                                                                                                                                                                   1.8s 
        ✔ 6e3e5bf64fd2 Pull complete                                                                                                                                                                                                                   1.6s 
        ✔ b62a2c2d2ce5 Pull complete                                                                                                                                                                                                                   2.2s 
        ✔ 8fd97c27b3fa Pull complete                                                                                                                                                                                                                   6.0s 
        ✔ cb70616b7657 Pull complete                                                                                                                                                                                                                   2.5s 
        ✔ d8ada539301f Pull complete                                                                                                                                                                                                                   3.1s 
        ✔ c60b6f73552c Pull complete                                                                                                                                                                                                                   3.4s 
        ✔ 665d514d2b02 Pull complete                                                                                                                                                                                                                   3.9s 
        [+] Running 4/4
        ✔ Network concourseci-install_default     Created                                                                                                                                                                                                0.2s 
        ✔ Container concourseci-install-db-1      Started                                                                                                                                                                                                0.4s 
        ✔ Container concourseci-install-web-1     Started                                                                                                                                                                                                0.0s 
        ✔ Container concourseci-install-worker-1  Started          ```
        ```

1. ブラウザでhttp://localhost:8080にアクセスする


-  Concourse CI のインストール手順(簡単な操作)

    https://note.com/shift_tech/n/nead03c02b095


- Concourse CI のチュートリアル

    https://concoursetutorial-ja.site.lkj.io/


## flyコマンドのインストール

- 以下のサイトのAssetsにて

    https://github.com/concourse/concourse/releases

-  2023-08-15 時点の最新版である fly-7.10.0-linux-amd64.tgz を選択

    https://github.com/concourse/concourse/releases/download/v7.10.0/fly-7.10.0-linux-amd64.tgz

- 以下のコマンドを実行


    ```
    # curl -L https://github.com/concourse/concourse/releases/download/v7.10.0/fly-7.10.0-linux-amd64.tgz > /tmp/fly
    ```
    ```
    # mv /tmp/fly /usr/local/bin/fly
    ```
    ```
    # chmod 775 /usr/local/bin/fly
    ```

    ```
    # chmod 755 /usr/local/bin/fly
    ```
