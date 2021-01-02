---
title: "Raspberry Pi 4 でつくる ”おうち Kubernetes Cluster”"
date: 2021-01-02T07:52:07+09:00
draft: false
tags:
- Kubernetes
- Raspberry Pi
---

これまでも仕事では GKE で Kubernetes 環境を構築したりしていたのですが、 GKE だとあまりにスムーズにオペレーションできてしまいます。  
ただ、 GKE にありがたみを感じればよいのか Kubernetes にありがたみを感じればよいのか、明確になっていないとかねてから感じていました。  
そこでありがたみを感じるべき相手を明確にすべく、自前で Kubernetes Cluster を立てることを思い立ち挑戦してみることにしました。

ググると多くの先人達のポストが見つかりますが、今回は主に [Raspberry Pi 4 でおうちKubernetesを作ろう（Raspbian Buster Lite対応版）](https://qiita.com/kentarok/items/6e818c2e6cf66c55f19a) を参考にさせていただきました 🙇  
このポストの冒頭でも

> もはや何十番煎じどころか、何百番煎じか分かりませんが、Raspberry Pi 4 で、おうちKubernetesを作ったので、そのメモを残しておきます。

とあるように、もはや一家に一台 Kubernetes Cluster 時代に突入しているようです。
時代の波に乗って、早速我が家でも ”おうち Kubernetes Cluster” を構築していきたいと思います。

1. 物理編（作成中）
1. OS セットアップ編（作成中）
1. Kubernetes Cluster 構築編（作成中）

### Kubernetes でのオペレーション関連

* RBAC でのユーザー管理（作成中）
* いざ、インターネットにコンテンツ公開（作成中）
* GitHub Actions によるデプロイ自動化（作成中）
