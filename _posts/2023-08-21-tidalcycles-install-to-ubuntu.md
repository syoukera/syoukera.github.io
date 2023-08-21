---
layout: post
title: TidalCyclesをUbuntuにインストールする
subtitle: Install TidalCycles on Ubuntu
author: syoukera
categories: tidalcycles
banner:
  video: https://vjs.zencdn.net/v/oceans.mp4
  loop: true
  volume: 0.8
  start_at: 8.5
  image: https://bit.ly/3xTmdUP
  opacity: 0.618
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
  subheading_style: "color: gold"
tags: tidalcycles ubuntu
sidebar: []
---

# 動作時の環境

- Ubuntu 22.04
- M-Audio Audiophile USB

# インストール

TidalCyclesのUbuntuへのインストールでも、WindowsやMacと同様にブートストラップでのインストールが可能。 
ここでは、[公式の記事](https://tidalcycles.org/docs/getting-started/linux_install/)に記載されている[Ansible method](https://github.com/cleary/ansible-tidalcycles)に従ってインストールする。  
以下のコマンドをターミナルで実行する。  

```bash
sudo apt install ansible git
git clone --recurse-submodules https://github.com/cleary/ansible-tidalcycles.git
cd ansible-tidalcycles/
```

使用したいエディタに合わせてインストールのコマンドを実行する。  
VSCodeを使用したい場合には以下のコマンドを実行することでTidalCyclesとVSCodeのプラグインがインストールされる。  

```bash
sudo ansible-playbook --connection=local -i localhost, tidal_vscode.play.yml
```

# オーディオ出力先の変更

デフォルトのスピーカーであれば上記の設定で音が鳴るのだが、オーディオインターフェースなどに出力先を変更するためには別途設定が必要と鳴る。  
WindowsやMacであれば[公式の記事](https://userbase.tidalcycles.org/Choosing_an_output_device.html)にあるように`Sever.default.options.outDevice_`を指定することによってオーディオの出力先を変更可能だが、Linuxにおいては使用できない。  
[SuperColliderのIssue](https://github.com/supercollider/supercollider/issues/4373)で知ったが、Linuxにおいてはオーディオハードウェアの選択はJACKによって行われているらしく、JACK上で設定する必要があるとのこと。  

JACKにはGUIのパッケージがいくつかあるので、今回はQJackCtlを使用する。  
インストールには以下のコマンドをターミナルで実行する  

```bash
sudo apt install qjackctl
```
