---
title: "Docker Compose V2で変わったdocker-compose.ymlの書き方"
emoji: "🐳"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["docker", "dockercompose"]
published: false
---

# 概要

2021年の後半に`Docker Compose V2`がリリースされました
`Docker Compose V2`はCompose Spec[^1]に準拠しているため、`docker-compose.yml`もその仕様に則った書き方ができるようになっています

```yaml docker-compose.yml
services:
  app1:
    image: awesome/webapp
    configs:
      - my_config
    secrets:
      - server-certificate

  app2:
    image: awesome/database
    extends:
      service: app1

configs:
  my_config:
    file: ./my_config.txt

secrets:
  server-certificate:
    file: ./server.cert
```

:::details 展開後のyaml
```yaml
services:
  app1:
    configs:
    - source: my_config
    image: awesome/webapp
    networks:
      default: null
    secrets:
    - source: server-certificate
  app2:
    configs:
    - source: my_config
    extends:
      service: app1
    image: awesome/database
    networks:
      default: null
    secrets:
    - source: server-certificate
networks:
  default:
    name: sample_default
secrets:
  server-certificate:
    name: sample_server-certificate
    file: /home/nkagami/repository/sample/server.cert
configs:
  my_config:
    name: sample_my_config
    file: /home/nkagami/repository/sample/my_config.txt
```
:::

# 前提

## Docker Compose V2 とは

[Compose Specification](https://github.com/compose-spec/compose-spec)に準拠した新しい`docker compose`コマンドです

大雑把に以下のような違いがあります

|              | v1                  | v2                |
| ------------ | ------------------- | ----------------- |
| 形式          | スタンドアロンバイナリ | docker cli plugin |
| 実装          | Python             | Golang            |
| Compose Spec | 準拠していない        | 準拠している        |

https://qiita.com/tearoom6/items/b06118acb801bd918f35

## Compose Spec とは

現在のComposeファイルの最新の仕様と考えて良さそうです

> Compose仕様は、クラウドやプラットフォームに依存しないコンテナベースのアプリケーションを定義するための、開発者向けの標準仕様です。

https://www.compose-spec.io/



### Compose Spec の実装はほかにも

- [Kompose](https://github.com/kubernetes/kompose)
- [Nerdctl](https://github.com/containerd/nerdctl)
- [Okteto Stacks](https://okteto.com/docs/reference/stacks)
- [Docker Cloud Integrations](https://github.com/docker/compose-cli)

# 具体的に何が変わったのか

気になったものをいくつか取り上げていきます。
仕様はすべて[spec.md](https://github.com/compose-spec/compose-spec/blob/master/spec.md#version-top-level-element)にまとまっているので、詳しくはそちらを参照してください。

## Composeファイルの名前[^2]

`--file`オプションを指定せずに`docker-compose`コマンドを実行したときに読み込まれるComposeファイルのデフォルトパスに違いがあります

```bash
# V1
$ docker-compose up

# V2
$ docker compose up
```

### V1

v1では作業ディレクトリに以下のファイルが存在すればそれが読み込まれます

- docker-compose.yaml
- docker-compose.yml

### V2

一方v2では、基本的に作業ディレクトリにある`compose.yaml`を読み込みます
ただし、後方互換を維持するために、これまでと同じく`docker-compose.yaml`も読み込んでくれます

- compose.yaml
- compose.yml
- docker-compose.yaml（後方互換のためサポート）
- docker-compose.yml（後方互換のためサポート）

:::message
両方のファイルが存在する場合、compose.yamlの方が優先される
```bash
.
├── compose.yaml  # ←こっちが優先される
└── docker-compose.yaml
```
:::

## version[^3]

```yaml
version: 3.9  # ←非推奨

services:
  foo:
    image: busybox
```

`Compose V1`では、この`version`の値を見てComposeファイルの検証が行われていました
Compose Specでは後方互換性のために仕様として定義されてはいるものの、この`version`の値を見てComposeファイルの検証を行うべきではないとされており、`Compose V2`を使う限りにおいてはもはや不要な項目となっています

## configs[^4]

設定ファイルのマウントを行う`configs`セクションが設定できるようになりました
`configs`は、コンテナ・ファイルシステムにマウントされるため、動きとしてはこれまでのマウントと同等です
（このように抽象化されてふつうのマウントと区別されることで、Compose Specに準拠したプラットフォームごとに`configs`を扱う方法を柔軟に変更できるというメリットが有るようです）

短い構文と長い構文の2種類の構文がサポートされています

### short syntax

コンテナ内の`/<config_name>`に読み取り専用でマウントされます

```yaml short syntax
services:
  redis:
    image: redis:latest
    configs:
      - my_config
configs:
  my_config:
    file: ./my_config.txt
```

### long syntax

```yaml long syntax
services:
  redis:
    image: redis:latest
    configs:
      - source: my_config
        target: /redis_config  # コンテナ内にマウントされるパス
        uid: "103"
        gid: "103"
        mode: 0440  # ファイルのパーミッション
configs:
  my_config:
    file: ./my_config.txt
```

## secrets[^5]

機密情報のマウントを想定した`secrets`セクションが設定できます
プラットフォームの実装が`configs`と大きく異なる可能性があるため、専用の`secrets`セクションとして分けられているようです
短い構文と長い構文の2種類の構文がサポートされています

### short syntax

コンテナ内の`/run/secrets/<secret_name>`に読み取り専用でマウントされる

```yaml
services:
  frontend:
    image: awesome/webapp
    secrets:
      - server-certificate
secrets:
  server-certificate:
    file: ./server.cert
```

### long syntax

```yaml
services:
  frontend:
    image: awesome/webapp
    secrets:
      - source: server-certificate
        target: server.cert  # コンテナ内の/run/secrets/にマウントされるファイル名
        uid: "103"
        gid: "103"
        mode: 0440  # ファイルのパーミッション
secrets:
  server-certificate:
    file: ./server.cert
```

`configs`もそうですが、この辺はKubernetesのマニフェストファイルの書き方に近くなった気がします

## extends[^6]

Composeファイルに定義された別のサービスを拡張し、オプションで設定を上書きするこができる`extends`が利用出るようになりました
`extends`をうまく活用することでスッキリとした`compose.yml`を書くことができそうです

### こういう`compose.yml`があるとき

```yaml compose.yml
services:
  common:
    image: busybox
    security_opt:
      - label:role:ROLE
  cli:
    extends:
      service: common
    security_opt:
      - label:user:USER
```

### このように展開されます

```bash
# 余談ですが、v2からconfigコマンドはconvertコマンドに置き換わっています
#（configはconvertのaliasとして設定されている）
$ docker compose convert
```

```yaml
services:
  cli:
    extends:
      service: common
    image: busybox
    networks:
      default: null
    security_opt:
    - label:role:ROLE
    - label:user:USER
  common:
    image: busybox
    networks:
      default: null
    security_opt:
    - label:role:ROLE

networks:
  default:
    name: sample_default
```

# まとめ

Compose Specに準拠して実装されたDocker Compose V2によって新しく使えるようになった`compose.yml`の書き方を紹介してきました
V2には後方互換性があるため、既存のdocker-compose.ymlをわざわざ修正するほどのコストをかけるべきでは無いかもしれませんが、新しく作る場合はCompose Specに準拠した書き方にしていくのが良さそうです

# 参考資料

https://www.compose-spec.io/
https://github.com/compose-spec/compose-spec
https://docs.docker.com/compose/cli-command/
https://zenn.dev/flyingbarbarian/articles/54d016c462c55b
https://zenn.dev/skanehira/articles/2021-06-03-new-docker-compose
https://qiita.com/tearoom6/items/b06118acb801bd918f35

[^1]: https://github.com/compose-spec/compose-spec
[^2]: https://github.com/compose-spec/compose-spec/blob/master/spec.md#compose-file
[^3]: https://github.com/compose-spec/compose-spec/blob/master/spec.md#version-top-level-element
[^4]: https://github.com/compose-spec/compose-spec/blob/master/spec.md#configs-top-level-element
[^5]: https://github.com/compose-spec/compose-spec/blob/master/spec.md#secrets-top-level-element
[^6]: https://github.com/compose-spec/compose-spec/blob/master/spec.md#extends