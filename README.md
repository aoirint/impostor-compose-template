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
    - Extreme Rolesを使用している場合（それぞれの実装に詳しくないのでわからないが、バニラと異なるパケットを送るMod使用時は弾かれる？）
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

カスタムサーバの設定に対応したMODを導入し、IP（ホスト名）とポート番号を設定する。
