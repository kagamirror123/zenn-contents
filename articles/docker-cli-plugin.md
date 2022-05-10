---
title: "Docker CLI Plugin とは"
emoji: "🐳"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["docker"]
published: true
---

# 概要

`Docker CLI Plugin`ってご存知でしょうか？

つい最近GAとなった`Docker Compose V2`[^1]やこちらも最近発表された`docker sbom`コマンド[^2]が`Docker CLI Plugin`として提供されています。
こいつがなんなのかよく分かってなかったので調べてみました。

:::message
Docker CLI Pluginについて公式のドキュメントがほとんど存在していませんでした。
調べるにあたって以下の記事が大変役立ちました。
https://zenn.dev/skanehira/articles/2021-06-03-new-docker-compose
:::

# Docker CLI Plugin

Dockerコマンドにプラグインとしてサブコマンドを追加して機能を拡張できる機構です。
[Docker v19.03](https://github.com/docker/docker-ce/releases/tag/v19.03.0)から追加されているようです。
けっこう前からあるんですね、まったく知りませんでした。

```bash:プラグインの使用例
$ docker compose up

$ docker sbom イメージ名
```

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

:::message alert

Docker Engineの機能を拡張する仕組みとしてDocker Pluginというものが存在しますが、これはまた別の仕組みなので混同しないように注意が必要です。
https://docs.docker.com/engine/extend/plugin_api/
:::

## プラグインに必要な要素

Docker CLI Pluginは以下の要件を満たしている必要があります。

- プラグイン名が`docker-`で始まっていること
- `docker-cli-plugin-metadata`サブコマンドが実装されていること

### Metadataのスキーマ

`docker-cli-plugin-metadata`サブコマンドは、以下の`Metadata`構造体に従ったJSONを出力するように実装します

https://github.com/docker/cli/blob/4cc4385075205409c835454f6e5055147d8495b4/cli-plugins/manager/metadata.go

参考までに`Docker Compose V2`では以下のようになっていました
```json:Docker Compose V2 の例
{
  "SchemaVersion": "0.1.0",
  "Vendor": "Docker Inc.",
  "Version": "v2.3.3",
  "ShortDescription": "Docker Compose"
}
```

## プラグインのインストール

`docker-`からはじまる実行ファイルを特定のディレクトリに配置することで、Dockerコマンドのサブコマンドしてプラグインが使えるようになります。

- ユーザーごとのインストールの場合 ➡ $HOME/.docker/cli-plugins/
- システムへのインストールの場合 ➡ /usr/local/lib/docker/cli-plugins/

# プラグインの作り方

Docker CLIのリポジトリに、Docker CLI Pluginのサンプルコードがあったのでこれを参考にしました。
https://github.com/docker/cli

https://github.com/docker/cli/blob/4cc4385075205409c835454f6e5055147d8495b4/cli-plugins/examples/helloworld/main.go

プラグインのEntrypointとなる関数`plugin.Run()`が用意さているのでこれを利用します。

https://github.com/docker/cli/blob/master/cli-plugins/plugin/plugin.go#L54-L79

:::message
前述のDocker CLI Pluginの要件を満たしていればいいので必ずしもGoで書く必要はありません。
:::

## 実装

`docker psx`という名前で、docker composeで作成されたプロジェクトごとにコンテナを表示する簡単なプラグインを作ってみました。
https://github.com/kagamirror123/docker-psx

Cobra[^3]のお作法に倣ってディレクトリ構成は以下のようにしました
```
.
├── commands
│   └── root.go
├── go.mod
├── go.sum
└── main.go
```

`main.go`に`plugin.Run()`を宣言して、メタデータを記述します。
```go:main.go
package main

import (
	"github.com/docker/cli/cli-plugins/manager"
	"github.com/docker/cli/cli-plugins/plugin"
	"github.com/docker/cli/cli/command"
	"github.com/kagamirror123/docker-psx/commands"
	"github.com/spf13/cobra"
)

func main() {

	plugin.Run(func(dockerCli command.Cli) *cobra.Command {
		return commands.NewRootCmd("psx", dockerCli)
	},
		manager.Metadata{
			SchemaVersion:    "0.1.0",
			Vendor:           "kagamirror",
			Version:          "v0.0.1",
			ShortDescription: "this plugin display containers by compose project",
			URL:              "https://github.com/kagamirror123/docker-psx",
		})
}
```

`commands/root.go`に実際の処理を書いていきます

```go:commands/root.go
func NewRootCmd(name string, dockerCli command.Cli) *cobra.Command {
	cmd := &cobra.Command{
		Short: "psx",
		Use:   name,
		Run: func(cmd *cobra.Command, _ []string) {
			// ここに処理を書く
		}
	}
}
```

## 配置

ビルドして、所定のディレクトリに配置します

```bash
$ go build

$ mkdir -p ~/.docker/cli-plugins/
$ chmod +x docker-psx
$ mv docker-psx ~/.docker/cli-plugins/
```

ちゃんと追加されているかを確認します
```
$ docker info
Client:
 Context:    default
 Debug Mode: false
 Plugins:
  buildx: Docker Buildx (Docker Inc., v0.8.2)
  compose: Docker Compose (Docker Inc., v2.4.1)
  psx: this plugin display containers by compose project (nkagamirror, v0.0.1)  # ⬅追加された！
  sbom: View the packaged-based Software Bill Of Materials (SBOM) for an image (Anchore Inc., 0.6.0)
  scan: Docker Scan (Docker Inc., v0.17.0)
```

## 実行

```bash
$ docker psx
```

無事dockerのサブコマンドとして実行するこができいい感じに表示できました

![result](https://user-images.githubusercontent.com/56927897/160223384-d9b9de29-5f71-429c-be0c-bad48bf12534.png)

# 参考

https://zenn.dev/skanehira/articles/2021-06-03-new-docker-compose#docker-cli-plugin%E3%81%AB%E3%81%A4%E3%81%84%E3%81%A6
https://github.com/docker/cli/blob/master/cli-plugins
https://github.com/docker/cli/issues/1534

[^1]: V2の1.0のリリース自体は2021年10月にされていましたが、その時点ではGAではなかったらしい。
[Announcing Compose V2 General Availability - Docker](https://www.docker.com/blog/announcing-compose-v2-general-availability/)
[^2]: [Announcing Docker SBOM: A step towards more visibility into Docker images \- Docker](https://www.docker.com/blog/announcing-docker-sbom-a-step-towards-more-visibility-into-docker-images/)
[^3]: GoでCLIツールを作成するためのフレームワーク https://github.com/spf13/cobra
