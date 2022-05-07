---
title: "Docker CLI Plugin とは"
emoji: "🎉"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["docker"]
published: false
---

# 概要

`Docker CLI Plugin`ってご存知でしょうか？

つい最近GAとなった`Docker Compose V2`[^1]やこちらも最近発表された`docker sbom`コマンド[^2]が`Docker CLI Plugin`として提供されています。
こいつがなんなのかよく分かってなかったので調べてみました。

:::message
公式のドキュメント等が存在せず（多分）、調べるにあたって以下の記事が非常に助かりました
https://zenn.dev/skanehira/articles/2021-06-03-new-docker-compose#docker-cli-plugin
:::

# Docker CLI Plugin

## とは

Dockerコマンドを拡張できる機構のことで、[Docker v19.03](https://github.com/docker/docker-ce/releases/tag/v19.03.0)から追加されているようです。
けっこう前からあるんですね、まったく知りませんでした。

`docker info`コマンドを叩くと現在インストールされているプラグインが表示できます
```bash
$ docker info
Client:
 Context:    default
 Debug Mode: false
 Plugins:
  buildx: Docker Buildx (Docker Inc., v0.8.2)
  compose: Docker Compose (Docker Inc., v2.4.1)
  sbom: View the packaged-based Software Bill Of Materials (SBOM) for an image (Anchore Inc., 0.6.0)
  scan: Docker Scan (Docker Inc., v0.17.0)
```

## プラグインに必要な要素

Docker CLI Pluginは以下の要件を満たしている必要があります。

- プラグイン名が`docker-`で始まっていること
- 次のようなJSONを出力する`docker-cli-plugin-metadata`サブコマンドが実装されていること

https://github.com/docker/cli/blob/4cc4385075205409c835454f6e5055147d8495b4/cli-plugins/manager/metadata.go


  ```json:Docker Compose V2 の例
  {
     "SchemaVersion": "0.1.0",
     "Vendor": "Docker Inc.",
     "Version": "v2.3.3",
     "ShortDescription": "Docker Compose"
  }
  ```

## Docker Plugin とは別

Docker Engineの機能を拡張する仕組みとしてDocker Pluginというものが存在しますが、これはまた別の仕組みなので混同しないように注意が必要です。
https://docs.docker.com/engine/extend/plugin_api/


# 作ってみた

## 

Docker CLIのリポジトリに、Docker CLI Pluginのサンプルコードがあったのでそれを参考に
https://github.com/docker/cli

https://github.com/docker/cli/blob/4cc4385075205409c835454f6e5055147d8495b4/cli-plugins/examples/helloworld/main.go

# 

# メモ

- 公式のドキュメント存在しない
- 僅かな情報
  - https://zenn.dev/skanehira/articles/2021-06-03-new-docker-compose#docker-cli-plugin%E3%81%AB%E3%81%A4%E3%81%84%E3%81%A6
  - https://github.com/docker/cli/issues/1534

```bash
/usr/local/lib/docker/cli-plugins/docker-compose docker-cli-plugin-metadata
{
     "SchemaVersion": "0.1.0",
     "Vendor": "Docker Inc.",
     "Version": "v2.3.3",
     "ShortDescription": "Docker Compose"
}
```

# 参考

https://zenn.dev/skanehira/articles/2021-06-03-new-docker-compose#docker-cli-plugin%E3%81%AB%E3%81%A4%E3%81%84%E3%81%A6
https://github.com/docker/cli/blob/master/cli-plugins


[^1]: V2の1.0のリリース自体は、2021年10月にされていましたが、GAではなかったらしい（https://www.docker.com/blog/announcing-compose-v2-general-availability/
[^2]: https://www.docker.com/blog/announcing-docker-sbom-a-step-towards-more-visibility-into-docker-images/