---
layout: post
title: ESP32/ESP8266を使ってTouchDesignerからLEDバーを操作する
subtitle: Control LED bar from Touchdesigner using ESP32/8266
categories: touchdesigner 
banner:
  video: https://github.com/syoukera/syoukera.github.io/assets/39158849/4d146767-4e51-4ea8-8797-36b83a00bc42
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
tags: touchdesigner pesp8266 ledbar
sidebar: []
---

Ali Expressで入手可能なLEDバーをTouchDesignerから光らせる．LEDバーへの電気信号のやり取りにはESP8266を使用し，データの転送にはWi-Fiを介してArtNetの通信を行った．

# 必要なもの
- LEDバー本体
- 直流電源，±15 V
- ESP8266
- 抵抗 330Ω

# LEDバー・電源・ESP8266の配線

LEDバーの配線を剥くと+15 V(赤)，-15 V（白）と指令値（黄色）の配線にたどり着く．
まず，+15 Vと-15 Vを直流電源の出力に接続すると緑色のライトがつく．
ここから光を制御するために，指令値にESP8266デジタル出力を抵抗を挟んで繋げる．
ESP8266には上記の指令値以外に，直流電源とアースをつなげておく必要がある．

# ESP8266の設定

Arduino IDEにArtNetなどのライブラリをインストールする．
サンプルファイルのArtNetWiFiFastLEDをESP8266に書き込む．
今回使用するLEDバーは５個のLEDで１ピクセルの情報で稼働し，一本当たり8ピクセル分のRGBの情報が必要となる．
NUM_PIXELは合計のピクセル数であるから，使用したいLEDバーの本数*8に変更しておく．
PIN_OUTPUTに出力するデジタルピンの番号を指定する．

# TouchDesignerでの設定

ESP8266に書き込んだNUM_PIXELの分のRGB情報をTOPとして準備し，TOP to CHOPを用いて変換する．
作成したCHOPをDMX out CHOPに接続し，パラメータをArtNet，サブネットマスクを255.255.255.255と設定する．
ここまでの設定がうまく行っていると，TouchDesigner上のTOPの色に合わせてLEDバーがリアルタイムで変化するようになる．

