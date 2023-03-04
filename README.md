# impostor-compose-template

- Docker Hub: <https://hub.docker.com/r/aeonlucid/impostor>
- <https://github.com/Impostor/Impostor/blob/6b6fa081250d235d1a7f0d4f8765c2816bb5f394/docs/Running-the-server.md>

## デプロイ

サーバ向けDocker EngineおよびDocker Compose V2が導入されたLinuxサーバ（Ubuntu）を想定する。

- <https://docs.docker.com/engine/install/ubuntu/>

`template.env`を`.env`にコピーする。以下、設定例。

- `HOST_GAME_PORT`は、必要があればバインドするアドレスに書き換える。通常、`0.0.0.0:22023`でよい。
    - NAT下にないVPSなどの場合、ポート番号の変更はここでする
    - サーバやVPSサービスにパケットフィルタがある場合、設定したポート番号でのUDP通信を許可しておく
- `HOST_HTTP_PORT`は、リバースプロキシでHTTPS化することを想定し、通常、`127.0.0.1:22000`でよい。

```env
HOST_GAME_PORT=0.0.0.0:22023
HOST_HTTP_PORT=127.0.0.1:22000
```

`template.config.json`を`config/config.json`にコピーする。以下、設定例。

- 以下の場合、AntiCheatは無効化する
    - うまくプレイできない（ルームの作成、参加時、試合開始時などに弾かれる）
    - MODを導入している
- `PublicIp`はWAN側IPアドレスに書き換える
- `PublicPort`はWAN側ポート番号に書き換える

```json
{
  "Server": {
    "PublicIp": "127.0.0.1",
    "PublicPort": 22023,
    "ListenIp": "0.0.0.0",
    "ListenPort": 22023
  },
  "AntiCheat": {
    "Enabled": false,
    "BanIpFromGame": false
  }
}
```

`template.config_http.json`を`config/config_http.json`にコピーする。以下、設定例。

```json
{
  "HttpServer": {
    "ListenIp": "0.0.0.0",
    "ListenPort": 22000,
    "UseHttps": false,
    "CertificatePath": ""
  }
}
```

### 起動

```shell
sudo docker compose pull
sudo docker compose up -d
```

### ログの確認

```shell
sudo docker compose logs -t -f
```

### 終了

```shell
sudo docker compose down
```

### Config作成の際に参考になるリンク

- <https://github.com/Impostor/Impostor/blob/6b6fa081250d235d1a7f0d4f8765c2816bb5f394/docs/Server-configuration.md>
- <https://github.com/Impostor/Impostor/blob/6b6fa081250d235d1a7f0d4f8765c2816bb5f394/src/Impostor.Server/config-full.json>
- <https://github.com/Impostor/Impostor.Http/blob/20adf632168cc8a7a590a8d4cc307134cca0e2f8/config_http.json>

## HTTPマッチングサーバを使用する場合の注意点

HTTPマッチングサーバにAmong Usの認証情報が送信される可能性があります（要調査）。

サーバー管理者は以下のような対策をするようにしてください。

- マッチングサーバをHTTPS化して平文で通信しないようにする
- カスタムマッチングサーバに接続する危険性についてユーザに説明する


## ゲームクライアントからの接続方法

### Among Us v2023.2.28 + HTTPマッチングサーバ

#### バニラの場合

`C:\Users\%USERNAME%\AppData\LocalLow\Innersloth\Among Us\regionInfo.json`を編集する。

`Regions`に以下を追加する。

HTTPマッチングサーバを`https://myserver.example.com:443`でホストしている場合、

```json
{
  "$type": "StaticHttpRegionInfo, Assembly-CSharp",
  "Name": "custom",
  "PingServer": "myserver.example.com",
  "Servers": [
    {
      "Name": "Http-1",
      "Ip": "https://myserver.example.com",
      "Port": 443,
      "UseDtls": false,
      "Players": 0,
      "ConnectionFailures": 0
    }
  ],
  "TranslateName": 1003
}
```

#### ExtremeRolesを導入する場合

起動時に`regionInfo.json`がMod側の設定で上書きされるため、`regionInfo.json`を書き換える方法が使用できません。

不定期更新ですが、サーバー設定UIからHTTPマッチングサーバを設定できるようにしたFork版を公開しています。

- [aoirint/ExtremeRoles Releases](https://github.com/aoirint/ExtremeRoles/releases)


### Among Us v2023.2.28 + UDPゲームサーバ

Among Us v2022.12.8以降、UDPゲームサーバで「ゲームを作成」しようとすると、ゲーム内にエラーが表示され、ログアウトするようになりました。

[参考画像](https://user-images.githubusercontent.com/27213639/210866901-d98914eb-eabe-46b6-a6d7-56d0336a7f6a.jpg)

ホストとなるユーザは、設定に加えてエラー回避操作が必要です。

ゲストとなるユーザは、設定のみで接続できるようですが、ホストを交代する可能性がある場合、エラー回避操作を把握しておくことを推奨します。

ゲーム映像を配信する場合、エラーメッセージにカスタムサーバのIPアドレス/ホスト名およびポート番号が含まれる点に注意してください。
マッチングルーム作成・参加の前後ではしばらく画面を映さないようにする対応を推奨します。

また、マッチングルーム入室時に同様のエラーメッセージが画面右下に表示される現象が条件不明で確認されているため、注意してください。

[参考画像](https://user-images.githubusercontent.com/27213639/211182985-ec1260aa-d67e-4f78-8514-abe9d0a7aa7c.png)

#### 設定

`C:\Users\%USERNAME%\AppData\LocalLow\Innersloth\Among Us\regionInfo.json`を編集する。

`Regions`に以下を追加する。

UDPゲームサーバを`127.0.0.1:22023`でホストしている場合、

```json
{
  "$type": "DnsRegionInfo, Assembly-CSharp",
  "Fqdn": "127.0.0.1",
  "DefaultIp": "127.0.0.1",
  "Port": 22023,
  "UseDtls": false,
  "Name": "custom",
  "TranslateName": 1003
}
```

UDPゲームサーバを`myserver.example.com:22023`でホストしている場合、

```json
{
  "$type": "DnsRegionInfo, Assembly-CSharp",
  "Fqdn": "myserver.example.com",
  "DefaultIp": "myserver.example.com",
  "Port": 22023,
  "UseDtls": false,
  "Name": "custom",
  "TranslateName": 1003
}
```

#### ホストのエラー回避操作

1. タイトル画面から「オンライン」を開き、リージョンを公式サーバのいずれかに設定する
2. ホストとして公式サーバで「ゲームを作成する」を開き、なにもせずに戻る
3. リージョンをカスタムサーバに設定する
4. 以後、通常の手順に従ってゲームを作成する。


## 古い情報

### Among Us 2022.10.25時点の接続手順

<details>

### カスタムサーバー設定機能のあるMOD

MODの機能を使って設定する。

### バニラ

`C:\Users\%USERNAME%\AppData\LocalLow\Innersloth\Among Us\regionInfo.json`を編集する。

`Regions`に以下を追加する。

ホスト名`myserver.example.com`の場合、

```json
{
  "$type": "DnsRegionInfo, Assembly-CSharp",
  "Fqdn": "myserver.example.com",
  "DefaultIp": "myserver.example.com",
  "Port": 22023,
  "UseDtls": false,
  "Name": "custom",
  "TranslateName": 1003
}
```

IPアドレス`127.0.0.1`の場合、

```json
{
  "$type": "DnsRegionInfo, Assembly-CSharp",
  "Fqdn": "127.0.0.1",
  "DefaultIp": "127.0.0.1",
  "Port": 22023,
  "UseDtls": false,
  "Name": "custom",
  "TranslateName": 1003
}
```

JSONを吐き出してくれるツールがあったが、公式サーバーの情報が古そうだったので個人的に非推奨（ <https://impostor.github.io/Impostor/> ）。

</details>


### Among Us 2022.8.23s時点のregionInfo.jsonの設定例

<details>

```json
{
  "CurrentRegionIdx": 3,
  "Regions": [
    {
      "$type": "StaticHttpRegionInfo, Assembly-CSharp",
      "Name": "North America",
      "PingServer": "matchmaker.among.us",
      "Servers": [
        {
          "Name": "Http-1",
          "Ip": "https://matchmaker.among.us",
          "Port": 443,
          "UseDtls": true,
          "Players": 0,
          "ConnectionFailures": 0
        }
      ],
      "TranslateName": 289
    },
    {
      "$type": "StaticHttpRegionInfo, Assembly-CSharp",
      "Name": "Europe",
      "PingServer": "matchmaker-eu.among.us",
      "Servers": [
        {
          "Name": "Http-1",
          "Ip": "https://matchmaker-eu.among.us",
          "Port": 443,
          "UseDtls": true,
          "Players": 0,
          "ConnectionFailures": 0
        }
      ],
      "TranslateName": 290
    },
    {
      "$type": "StaticHttpRegionInfo, Assembly-CSharp",
      "Name": "Asia",
      "PingServer": "matchmaker-as.among.us",
      "Servers": [
        {
          "Name": "Http-1",
          "Ip": "https://matchmaker-as.among.us",
          "Port": 443,
          "UseDtls": true,
          "Players": 0,
          "ConnectionFailures": 0
        }
      ],
      "TranslateName": 291
    },
    {
      "$type": "DnsRegionInfo, Assembly-CSharp",
      "Fqdn": "127.0.0.1",
      "DefaultIp": "127.0.0.1",
      "Port": 22023,
      "UseDtls": false,
      "Name": "custom",
      "TranslateName": 1003
    }
  ]
}
```

</details>

