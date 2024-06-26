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
sudo yum -y update
```

一旦リブートしてリブート可能かどうか確認しておくのがよい。

```
sudo reboot
```

サービスを停止しておく:
```
systemctl stop httpd
systemctl disable httpd
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

リブート後CentOS 7のパッケージ、elevate (leapp)パッケージが残っていないかどうか確認する:
```
sudo rpm -qa | grep el7
sudo rpm -qa | grep elevate
sudo rpm -qa | grep leapp
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
sudo dnf remove $(rpm -qa | grep elevate)
sudo dnf remove $(rpm -qa | grep leapp)
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
ででてくるCentOSのGPGキーを消す。実行するにはこのコマンドででてくるCentOS
の行の第1コラムを引数にしてrpm -eで消す。例:

```console
# rpm -q gpg-pubkey --qf '%{NAME}-%{VERSION}-%{RELEASE}\t%{SUMMARY}\n'
gpg-pubkey-f4a80eb5-53a7ff4b	gpg(CentOS-7 Key (CentOS 7 Official Signing Key) <security@centos.org>)
gpg-pubkey-81b961a5-64106f70	gpg(ELevate <packager@almalinux.org>)
gpg-pubkey-3abb34f8-5ffd890e	gpg(AlmaLinux <packager@almalinux.org>)
gpg-pubkey-ced7258b-6525146f	gpg(AlmaLinux OS 8 <packager@almalinux.org>)
# rpm -e gpg-pubkey-f4a80eb5-53a7ff4b
# rpm -q gpg-pubkey --qf '%{NAME}-%{VERSION}-%{RELEASE}\t%{SUMMARY}\n'
gpg-pubkey-81b961a5-64106f70	gpg(ELevate <packager@almalinux.org>)
gpg-pubkey-3abb34f8-5ffd890e	gpg(AlmaLinux <packager@almalinux.org>)
gpg-pubkey-ced7258b-6525146f	gpg(AlmaLinux OS 8 <packager@almalinux.org>)
```
``ELevate <packager@almalinux.org>``はそのままでもいいだろう。

``*.rpmnew``、``*.rpmsave``ファイルを探し、対応する:
```
find / -name '*.rpmnew' -or -name '*.rpmsave'
```

``rpm -V``でパッケージファイルのチェックサムを調べて変更があった
ファイルを把握する:

```
sudo -s
for i in $(rpm -qa); do
    echo $i
    rpm -V $i >> /tmp/checkpackages.log
done
```

