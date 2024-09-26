---
title: "rainの便利なコマンドと記法"
emoji: "☔️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Cloudformation,Rain,AWS]
published: false
---

# 概要

rainで利用できる、「あまり存在を知られていなそうだけど便利なコマンドや記法」について一部紹介します。

rainの基本的な使い方については触れません。以下の記事が大変わかりやすいです。
[rain: CloudFormation を便利に操作しちゃおう \- kakakakakku blog](https://kakakakakku.hatenablog.com/entry/2023/05/05/110658)

## rainについて

rainは、AWS CloudFormationのテンプレート作成やデプロイを簡素化するためのコマンドラインツールです。
テンプレートの検証、フォーマット、デプロイを効率的に行うことができます。
https://github.com/aws-cloudformation/rain


# コマンド

## ls

通常 rain deploy の際に、以下のようにスタックの差分を確認することができますが、これだと「S3バケットに何らかの変更がある」ことは分かっても、「S3バケットにどんな変更があったか」までは分かりません。

![aaa](/images/スクリーンショット%202024-09-26%2017.13.09.png)

そこで、`ls --changeset`を使うことで差分の詳細を確認することができます。

```console
# --no-execでチェンジセットの作成までに留めておく
output=$(rain deploy template.yaml kagamirror-test --no-exec --yes)

# チェンジセットIDを取得
changeset_id=$(echo "$output" | grep 'not executed: ' | awk -F ': ' '{print $2}')

# チェンジセットを確認
rain ls -c kagamirror-test ${changeset_id}
```

before/afterが表示され、プロパティレベルで差分を確認することができました。
![aaa](/images/スクリーンショット%202024-09-26%2017.25.16.png)

差分が問題なければ、`rain deploy --changeset`でスタックをデプロイすればOK

## forecast

a

## log

--chart でガントチャートを表示

# 記法

## !Rain::Embed

## !Rain::Include

## !Rain::Env

## !Rain::S3Http

## Parameters