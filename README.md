# impostor-compose-template

- Docker Hub: <https://hub.docker.com/r/aeonlucid/impostor>
- <https://github.com/Impostor/Impostor/blob/6b6fa081250d235d1a7f0d4f8765c2816bb5f394/docs/Running-the-server.md>

## デプロイ

サーバ向けDocker EngineおよびDocker Compose V2が導入されたLinuxサーバ（Ubuntu）を想定する。

- <https://docs.docker.com/engine/install/ubuntu/>

`template.env`を`.env`にコピーする。以下、設定例。

- `HOST_PORT`は、必要があればバインドするアドレスに書き換える。通常、`0.0.0.0:22023`でよい。
    - NAT下にないVPSなどの場合、ポート番号の変更はここでする
    - サーバやVPSサービスにパケットフィルタがある場合、設定したポート番号でのUDP通信を許可しておく

```env
HOST_PORT=0.0.0.0:22023
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


## 接続

### カスタムサーバー設定機能のあるMOD

MODの機能を使って設定する。

### バニラ

`C:\Users\user\AppData\LocalLow\Innersloth\Among Us\regionInfo.json`を編集する。

`Regions`に以下を追加する。

ホスト名`myserver.example.com`の場合、

```json
{"$type":"DnsRegionInfo, Assembly-CSharp","Fqdn":"myserver.example.com","DefaultIp":"myserver.example.com","Port":22023,"UseDtls":false,"Name":"custom","TranslateName":1003}
```

IPアドレス`127.0.0.1`の場合、

```json
{"$type":"DnsRegionInfo, Assembly-CSharp","Fqdn":"127.0.0.1","DefaultIp":"127.0.0.1","Port":22023,"UseDtls":false,"Name":"custom","TranslateName":1003}
```

JSONを吐き出してくれるツールがあったが、公式サーバーの情報が古そうだったので個人的に非推奨（ <https://impostor.github.io/Impostor/> ）。

#### 2022.8.23s

<details>

```json
{
  "CurrentRegionIdx": 2,
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
    }
  ]
}
```

</details>

