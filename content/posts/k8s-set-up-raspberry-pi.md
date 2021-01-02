---
title: "Raspberry Pi 4 に Ubuntu 20.10 を入れてセットアップする"
date: 2020-01-03T08:25:05+09:00
draft: true
tags:
- Kubernetes
- Raspberry Pi
---

[Raspberry Pi 4 でつくる ”おうち Kubernetes Cluster” シリーズ](/k8s-create-cluster) の OS セットアップ編です

## OS インストール

せっかくだったら 64bit の方がいいだろうということで OS は Raspberry Pi OS ではなく Ubuntu に挑戦したいと思います  
[公式チュートリアル](https://ubuntu.com/tutorials/how-to-install-ubuntu-on-your-raspberry-pi#2-prepare-the-sd-card) で案内されている通り Imager アプリが用意されています  
以前はこんなのなかったと思うのですが、便利になりました

Imager を開くと



こんな感じで OS と SD カードが選択できるようになっています

Ubuntu から



`Ubuntu Server 20.10 (RPi 3/4/400)` を選択します

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

#### 参考ポスト

今回も先人たちの叡智にお世話になりました

* [5ステップで完了！ラズベリーパイ(B+)のセキュリティ設定まとめ！](http://masatolan.com/raspberry-pi/raspberry-pi-security/)
* [ラズパイでやらなければいけない４つのセキュリティ対策！](https://qiita.com/nokonoko_1203/items/94a888444d5019f23a11)
