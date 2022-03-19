---
title: "Docker Compose V2でできるようになった書き方"
emoji: "🐳"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["docker", "docker-compose", "compose-spec"]
published: false
---

# 概要

# TL;DR

- Docker Compose v2 は、Compose Spec に準拠して実装されているという点が肝
- Compose Spec に準拠していることでプラットフォームに依存しないよ
- Compose Spec に準拠していることでこんな風にdocker-compose.ymlを書けるようになったよ

# Docker Compose V2 とは

# Compose Spec とは

プラットフォームによらないComposeファイルの仕様規約であるCompose Specを理解する
Compose Spec を理解することで、Docker Compose v2 が何であるかが見えてくる
内容
docker-compose.ymlは実はいまこんなふうにかけるようになっている
Compose Specとは
compose-spec
プラットフォームに依存しないマルチコンテナ・アプリケーションの定義のための標準を確立するもの
仕様は、spec.mdに詳細に書いてある
Compose Spec に則った実装の例
以下のようなツールが有る
•	Docker Compose v2
•	Kompose
•	Nerdctl
など
Docker Compose v2 とはなんなのか
Docker Compose v1 のメジャーアップデート版
	v1	v2	備考
形式	スタンドアロンバイナリ	docker cli plugin	
言語	Python	Golang	大して重要ではない
Compose Spec	準拠していない	準拠している	ここが最も重要
後方互換性があるため、v2 は既存のdocker-compose.ymlの記法も扱える
ただし、Extension Field のマージの挙動が違ったりと、微妙に異なる部分もあるっぽい（どちらも結局DockerのAPIを叩いているのでほとんど差異はないという認識は正しい）
Compose Spec で定義されている記法（一部）
versionは後方互換性のために残されているだけでCompose Spec的には必要のない項目
一部のみ説明
configやsecretなどを中心に
ECSやACIなどにデプロイする際は、どうしてもプラットフォーム特有の要素（仮想ネットワークや認可の仕組み）が関わってくるが、そういったものは各プラットフォームごとにExtension Fieldで定義することで上手く連携することが出来る

v1 / v2 どちらを使うべきか
基本的にはv2を使い、docker-compose.ymlの記法もCompose Spec に則ったものにすべき
ただし、既存のdocker-compose.ymlがあるならば、それをわざわざすべて修正するほどのコストをかけるべきでは無いとも思う（そのための後方互換性）
Compose Spec に則った記法にしておくと、KomposeによるKubernetesのマニフェストファイルへの変換が綺麗にできるようになるなどの恩恵もある
