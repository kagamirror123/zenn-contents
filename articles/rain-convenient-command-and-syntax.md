---
title: "rainの便利なコマンドと記法"
emoji: "☔️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Cloudformation,Rain,AWS]
published: false
---

# 概要

rainは、AWS CloudFormationのテンプレート作成やデプロイを簡素化するためのコマンドラインツールです。
テンプレートの検証、フォーマット、デプロイを効率的に行うことができます。
https://github.com/aws-cloudformation/rain

rainで利用できる、「あまり存在を知られていなそうだけど便利なコマンドや記法」について一部紹介します。

rainの基本的な使い方については触れません。以下の記事が大変わかりやすいです。
[rain: CloudFormation を便利に操作しちゃおう \- kakakakakku blog](https://kakakakakku.hatenablog.com/entry/2023/05/05/110658)

# コマンド

## deploy

チェンジセットで詳細を見る

## forecast

## log

--chart でガントチャートを表示

# 記法

## !Rain::Embed

## !Rain::Include

## !Rain::Env

## !Rain::S3Http

## Parameters