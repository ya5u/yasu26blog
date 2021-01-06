---
title: "Raspberry Pi 4 に Ubuntu 20.10 を入れてセットアップする"
date: 2020-01-06T08:25:05+09:00
draft: false
tags:
- Kubernetes
- Raspberry Pi
---

[Raspberry Pi 4 でつくる ”おうち Kubernetes Cluster” シリーズ](/posts/k8s-create-cluster) の OS セットアップ編です

## OS インストール

せっかくだったら 64bit の方がいいだろうということで OS は Raspberry Pi OS ではなく Ubuntu に挑戦したいと思います  
[公式チュートリアル](https://ubuntu.com/tutorials/how-to-install-ubuntu-on-your-raspberry-pi#2-prepare-the-sd-card) で案内されている通り Imager アプリが用意されています  
以前はこんなのなかったと思うのですが、便利になりました

Imager を開くと

![rpi_imager_home](/rpi_imager_home.png)

こんな感じで OS と SD カードが選択できるようになっています

Ubuntu から

![rpi_imager_os_selection](/rpi_imager_os_selection.png)

`Ubuntu Server 20.10 (RPi 3/4/400)` を選択します

![rpi_imager_ubuntu_selection](/rpi_imager_ubuntu_selection.png)

あとは SD カードを選択して `WRITE` を押すと書き込みが始まるので完了するのを待ちます  
コマンドを叩かなくても GUI で簡単にイメージをやけるのは本当にありがたいです

## Raspberry Pi 起動！

Ubuntu はデフォルトで SSH が有効になっているのでラズパイに SD カードをセットしたら電源ケーブルをつないで早速起動します  
ラズパイにモニターとキーボードにつないでも `arp` コマンドで IP にあたりをつけて SSH 接続してもお好みで

デフォルトのユーザー / パスワードは `ubuntu / ubuntu` ですが初回ログイン時にパスワードを変更させられるので任意のパスワードを設定します

```bash
Welcome to Ubuntu 20.10 (GNU/Linux 5.8.0-1006-raspi aarch64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of <timestamp>

  System load:                      0.08
  Usage of /:                       7.9% of 27.57GB
  Memory usage:                     31%
  Swap usage:                       0%
  Temperature:                      28.9 C
  Processes:                        134
  Users logged in:                  0
  IPv4 address for eth0: <IPv4 address>
  IPv6 address for eth0: <IPv6 address>

 * Introducing self-healing high availability clusters in MicroK8s.
   Simple, hardened, Kubernetes for production, from RaspberryPi to DC.

     https://microk8s.io/high-availability

53 updates can be installed immediately.
35 of these updates are security updates.
To see these additional updates run: apt list --upgradable


Last login: <timestamp> from <IP>
ubuntu@ubuntu:~$
```

こんな感じのプロンプトが出たら起動成功です🎉

## 初期設定

まずはライブラリの最新化

```bash
$ sudo apt update && sudo apt upgrade
```

サーバーとしてインターネットにコンテンツを公開することを目標にしているのでセキュリティ設定もしていきます

**root パスワードを設定**

```bash
$ sudo passwd root
```

**ユーザー追加**　_※ 以下 `<user name>` の部分は任意のユーザー名を指定してください_

```bash
$ sudo adduser <user name>
```

このあたりはプロンプトに従って設定していきます

**管理者権限付与**

```bash
$ groups ubuntu
ubuntu : ubuntu adm dialout cdrom floppy sudo audio dip video plugdev netdev lxd

$ sudo usermod -G ubuntu,adm,dialout,cdrom,floppy,sudo,audio,dip,video,plugdev,netdev,lxd <user name>

# 設定を確認
$ groups <user name>
<user name> : <user name> adm dialout cdrom floppy sudo audio dip video plugdev netdev lxd ubuntu
```

**SSHまわりの設定**

Ubuntu では ssh はデフォルトで有効になっているので Raspberry Pi OS のように有効化する手順は不要です

このあとパスワード認証を不可にするので秘密鍵と公開鍵を生成して ~/.ssh/authorized_keys に追加しておきます  
そして /etc/ssh/sshd_config を以下のように編集します

* `PasswordAuthentication` を `no` に変更
* `PermitEmptyPasswords` を `no` に変更
* `PermitRootLogin` を `no` に変更
* `Port` を 22 以外の未使用のポート番号に変更

その後以下を実行して設定を反映させます

```bash
$ sudo /etc/init.d/ssh restart
```

このあたりで鍵認証で ssh できるかどうか確認しておきましょう  
万一これがうまくいっていないとラズパイ本体に直接ディスプレイとキーボードつなぐなどして設定変更するしか手がなくなってしまいます…

**ubuntu ユーザーを削除**

```bash
$ sudo userdel -r ubuntu
userdel: group ubuntu not removed because it has other members.
userdel: ubuntu mail spool (/var/mail/ubuntu) not found

# 確認
$ id -a ubuntu
id: ‘ubuntu’: no such user
```

**IP 固定化**

Ubuntu での IP 固定は /etc/netplan/50-cloud-init.yaml を以下のように編集して設定します

```
# This file is generated from information provided by the datasource.  Changes
# to it will not persist across an instance reboot.  To disable cloud-init's
# network configuration capabilities, write a file
# /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg with the following:
# network: {config: disabled}
network:
    ethernets:
        eth0:
            addresses:
            - <NodeごとのIP>/24
            dhcp4: false
            gateway4: <ルーターのIP>
            nameservers:
                addresses:
                - <ルーターのIP>
    version: 2
```

**ホスト名変更**

```bash
$ sudo hostnamectl set-hostname <Nodeのホスト名>
```

他の Node のホスト名も教えるには /etc/hosts に以下のように追記します

```
127.0.0.1        <自分のホスト名>
<各NodeのIP>     <各Nodeのホスト名>
...
```

初期設定としてはここまでです  
お疲れさまでした🎊

#### 参考ポスト

今回も先人たちの叡智にお世話になりました

* [5ステップで完了！ラズベリーパイ(B+)のセキュリティ設定まとめ！](http://masatolan.com/raspberry-pi/raspberry-pi-security/)
* [ラズパイでやらなければいけない４つのセキュリティ対策！](https://qiita.com/nokonoko_1203/items/94a888444d5019f23a11)
* [[Ubuntu]ローカルIPアドレスを固定にする(18.04/16.04)](https://jyn.jp/ubuntu-localip-static/)
