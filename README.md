# ローカルサイト https 転送

tailscale network 内のローカルサイトに https で接続できるようにするための docker compose コンフィグレーションサンプルです。

## セットアップ

### 1. Docker イメージの取得（最新化）

compose.yaml では、以下の latest イメージを使用しています。
必要に応じて適宜 docker pull して最新化してください。

```bash
sudo docker pull tailscale/tailscale:latest
sudo docker pull caddy:latest
```

### 2. 環境設定ファイルの作成

`.env.example` をコピーして `.env` ファイルを作成:

```bash
cp .env.example .env
```

必要に応じて `.env` ファイルの内容を編集してください。

設定例:

```bash
# Tailscale設定
TAILSCALE_HOSTNAME=tailscale-host
TS_EXTRA_ARGS=--advertise-tags=tag:container,tag:tailscale-host-acl --reset

# Caddy設定
CADDY_DOMAIN=tailscale-host.example.net
UPSTREAM_HOST=127.0.0.1
UPSTREAM_PORT=443
```

ここで advertise するタグが、最終的に登録される tailscale のホストに付与されます。

Access controls で事前にタグとACLの設定をしておいてください。

### 3. Tailscale OAuth Client の作成

1. Tailscale Admin Console にアクセス: https://login.tailscale.com/admin/settings/oauth
2. "Generate OAuth client" をクリック
3. 以下の権限を設定:
   - Devices > Core: Write
4. タグの設定（`.env` の `TS_EXTRA_ARGS` で指定したタグと一致させる）:
   - `tag:container`
   - `tag:tailscale-host-acl`
5. 生成された Client Secret をコピー

### 4. Docker Secret ファイルの作成

`secrets/ts_authkey.txt` ファイルを作成:
```bash
mkdir -p secrets
echo "tskey-client-xxxxxxxxxxxxx-yyyyyyyyyyyyyyy" > secrets/ts_authkey.txt
chmod 600 secrets/ts_authkey.txt
```

Client Secret を保存してください。

### 5. 起動

```bash
docker compose up -d
```

### 6. Tailscale ホストの承認

tailscale サービスの docker ログに出力される指示に従って tailscale network へのホスト登録を承認してください。

```
sudo docker compose logs tailscale
```
