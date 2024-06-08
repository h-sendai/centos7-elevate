# centos7-elevate

CentOS 7をAlmaLinuxの
[elevate](https://wiki.almalinux.org/elevate/)
を使ってAlmaLinux 8へアップグレードするときのメモ。

## 解説文書

- [Quick Guide](https://wiki.almalinux.org/elevate/ELevating-CentOS7-to-AlmaLinux-9.html)
- [CentOS 7 to AlmaLinux 8](https://wiki.almalinux.org/elevate/ELevating-CentOS7-to-AlmaLinux-9.html#migrate-centos-7-to-almalinux-8)


## 準備

リモートからアップグレードするときはシリアルコンソールがあると
リブート後のアップグレード作業の様子が見ることができて安心できる。

## 手順

バックアップを取っておく。

CentOS 7でアップデートしておく:

```
sudo dnf -y update
```

一旦リブートしてリブート可能かどうか確認しておくのがよい。

```
sudo reboot
```

elevate-releaseのインストール:
```
sudo yum install -y http://repo.almalinux.org/elevate/elevate-release-latest-el$(rpm --eval %rhel).noarch.rpm
```

leapp-upgradeとleapp-data-almalinuxパッケージのインストール:

```
sudo yum install -y leapp-upgrade leapp-data-almalinux
```

アップグレード可能かどうかのテスト:
```
sudo leapp preupgrade
```

画面に赤文字が出たら
/var/log/leapp/leapp-report.txt
を見て対処する。たいていは

```
sudo leapp answer --section remove_pam_pkcs11_module_check.confirm=True
```

でよい。

もう一度``leapp preupgrade``を実行する:
```
sudo leapp preupgrade
```

画面に緑色で表示されればOK。

アップグレードを開始:
```
sudo leapp upgrade
```

終了後、リブートする:
```
sudo reboot
```

リブート後、grub画面で``ELevate-Upgrade-Initramfs``というのが選択されているので
そのままこれを起動する。
作業が終わると自動でリブートされる。

リブート後CentOS 7のパッケージが残っていないかどうか確認する:
```
sudo rpm -qa | grep el7
```

``/etc/dnf/dnf.conf``でelevate関連パッケージがexclude行に指定されているので
コメントアウトしておく:

```
sudo vi /etc/dnf/dnf.conf
```

```
sudo rpm -qa | grep el7
```
で出てきたパッケージを削除する。

```
sudo dnf remove $(rpm -qa | grep el7)
```

```
sudo rm -fr /root/tmp_leapp_py3
```

```
sudo dnf clean all
```

```
rpm -q gpg-pubkey --qf '%{NAME}-%{VERSION}-%{RELEASE}\t%{SUMMARY}\n'
```
ででてくる古いGPGキーを消す。実行するにはこのコマンドででてくる第1
コラムを引数にしてrpm -eで消す。
