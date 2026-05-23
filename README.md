# Go + Next.js Clean Architecture Boilerplate

本プロジェクトは、Go（Clean Architecture）と Next.js（App Router）を組み合わせ、Docker / Podman 環境で安全かつ迅速に開発を開始するための汎用ボイラープレートです。

WebSocket 等の特定ドメインへの依存を排除し、ヘルスチェック疎通確認（API / DB / Redis）のみを実装しているため、あらゆる新規Webアプリケーション開発の土台として活用できます。

---

## 技術スタック

### Backend
- **Language**: Go 1.25 (Go 1.24+)
- **Library**: GORM (GORM/MySQL), go-redis/redis/v8
- **Architecture**: クリーンアーキテクチャ (`domain` -> `usecase` -> `delivery` -> `infrastructure`)

### Frontend
- **Framework**: Next.js 16 (App Router)
- **Library**: React 19, TypeScript
- **Styling**: Vanilla CSS

### Infrastructure (Docker / Podman)
- **Web Server / Reverse Proxy**: Nginx (Alpine)
- **Database**: MySQL 8.0
- **Cache**: Redis 7 (Alpine)

---

## 開発の開始方法

### 1. 環境変数の設定
ローカル起動用の設定として、`.env.example` から `.env` を作成します。
```bash
cp .env.example .env
```
また、E2Eテスト環境用に `.env.e2e.example` から `.env.e2e` を作成します。
```bash
cp .env.e2e.example .env.e2e
```

### 2. 依存関係のインストール (Frontend)
開発を始める前に、フロントエンドの依存パッケージをインストールして `package-lock.json` を生成してください。
```bash
cd frontend
npm install
```

### 3. コンテナ環境の起動
Makefile を利用して Podman / Docker 上で全サービスを一括起動します。
```bash
# ローカル開発環境の起動
make up

# ホットリロードの利用
# フロントエンド (Next.js) はポート3000でホスト側からマウントされており、コード変更が即座に反映されます。
```

起動後、ブラウザで [http://localhost:8080](http://localhost:8080) にアクセスすると、Backend API, MySQL DB, Redis Cache への接続状況がリアルタイムで確認できる **Boilerplate Dashboard** が表示されます。

### 4. コンテナの操作コマンド
```bash
# 停止
make down

# 再ビルドと起動
make rebuild

# ログの確認
make logs

# コンテナの状態確認
make ps

# ボリュームを含む完全なリセット
make clean
```

---

## テストの実行

ボイラープレートには、動作確認用のユニットテスト、結合テスト、E2Eテストがすべて整備されています。

```bash
# バックエンドの単体テスト
make test-backend

# バックエンドの結合テスト (testcontainers-go による MySQL/Redis 実コンテナテスト)
make test-integration

# フロントエンドの単体テスト (Vitest)
make test-frontend

# Playwright による E2E テスト
make test-e2e
```

---

## ディレクトリ構成と設計規則

### Backend (`/backend`)
クリーンアーキテクチャの原則に従い、依存関係は常に内側（ビジネスロジック）に向かいます。

- `/domain`: ビジネス実体とリポジトリインターフェースを定義（外部依存なし）。
- `/usecase`: アプリケーション固有のユースケースとビジネスロジックの調整。
- `/delivery`: HTTPルーティングとハンドラーの実装（REST API）。
- `/infrastructure`: MySQL(GORM)やRedisへの接続設定や永続化アダプターの実装。

### Frontend (`/frontend`)
Next.js 16 App Router を基準にしたモダンなコンポーネント構成です。

- `/src/app`: ページ遷移・レイアウト (`page.tsx`, `layout.tsx`)。
- `/src/components`: 再利用可能なUIコンポーネント。
- `/src/hooks`: 状態管理やAPI呼び出しを行うカスタムフック。
- `/src/styles`: CSSスタイルファイル。

---

## 環境変数の詳細 (Environment Variables)

このボイラープレートで使用される主な環境変数は以下の通りです。ローカル開発時には `.env`、E2Eテスト時には `.env.e2e`、CD (デプロイ) 時には GitHub Secrets / Variables または環境ごとの設定ファイル（`.env.stg` / `.env.prod`）を通じて設定します。

### 全般設定
| 環境変数名 | 設定例 / 推奨値 | 説明 |
|---|---|---|
| `APP_ENV` | `dev` / `stg` / `prod` | アプリケーションの動作環境を指定する環境識別子。 |
| `ALLOWED_ORIGIN` | `http://localhost:3000` | CORS (Cross-Origin Resource Sharing) で許可するフロントエンドのオリジン。 |
| `GHCR_REPO` | `yourusername/boilerplate` | コンテナイメージのビルドおよびプッシュ先となる GitHub Container Registry (GHCR) のリポジトリパス。 |
| `NODE_ENV` | `development` / `production` | Node.js の動作モード。ローカル開発時は `development`、本番・ステージング時は `production`。 |
| `RESTART_POLICY` | `no` / `always` | Docker/Podman コンテナの再起動ポリシー。ローカル開発時は `no`、本番環境等は `always` または `unless-stopped`。 |
| `NGINX_PORT` | `8080` | Nginx (リバースプロキシ) がホスト側で公開するポート番号。 |

### データベース (MySQL) 設定
| 環境変数名 | 設定例 / 推奨値 | 説明 |
|---|---|---|
| `MYSQL_ROOT_PASSWORD` | `your_root_password_here` | MySQLの `root` ユーザーのパスワード。 |
| `MYSQL_DATABASE` | `appdb` | アプリケーション用データベース名。 |
| `MYSQL_USER` | `app_user` | アプリケーションから接続する一般ユーザー名。 |
| `MYSQL_PASSWORD` | `your_app_password_here` | アプリケーション用一般ユーザーの接続パスワード。 |
| `MYSQL_DSN` | `app_user:password@tcp(mysql:3306)/appdb?parseTime=true&loc=Local` | Go バックエンドが MySQL に接続するための DSN (Data Source Name)。ホスト名は `mysql` (Dockerネットワーク内) にします。 |
| `MYSQL_EXPOSE` | `127.0.0.1:3306` | ホスト側から MySQL コンテナへ直接接続する場合のポート設定。ローカル開発以外の環境では、セキュリティのため `127.0.0.1:3306` のようにループバックアドレスに制限することを推奨します。 |
| `MYSQL_VOLUME_NAME` | `boilerplate_mysql_dev_data` | MySQL データを永続化するための外部ボリューム名。事前に作成しておく必要があります。 |

### キャッシュ (Redis) 設定
| 環境変数名 | 設定例 / 推奨値 | 説明 |
|---|---|---|
| `REDIS_ADDR` | `redis:6379` | Go バックエンドから接続する Redis のアドレス。ホスト名は `redis` (Dockerネットワーク内) にします。 |
| `REDIS_EXPOSE` | `127.0.0.1:6379` | ホスト側から Redis コンテナへ直接接続する場合のポート設定。 |
| `REDIS_VOLUME_NAME` | `boilerplate_redis_dev_data` | Redis データを永続化するための外部ボリューム名。事前に作成しておく必要があります。 |

### フロントエンド / その他設定
| 環境変数名 | 設定例 / 推奨値 | 説明 |
|---|---|---|
| `FRONTEND_TARGET` | `dev` / `runner` | `frontend/Dockerfile` で使用するマルチステージビルドのターゲット。ローカル開発時は `dev`、本番ビルド時は `runner` を指定します。 |
| `NEXT_PUBLIC_API_URL` | `http://localhost:8080/api` | フロントエンド (Next.js) から Backend API へリクエストを送信する際のエンドポイント。ブラウザ（クライアント）から到達可能なアドレスを指定します。 |
| `CLOUDFLARE_TUNNEL_TOKEN` | (任意) | Cloudflare Tunnel を使用して外部公開する場合のトンネルトークン。空文字の場合は cloudflared コンテナは立ち上がりますが機能しません。 |
| `AUTH_BASIC_STATE` | `off` / `Restricted` | `Restricted` に設定すると、Nginx レベルで Basic 認証が有効化されます（ステージング環境などで有用）。 |

---

## CI/CDワークフローとデプロイ設定

GitHub Actions (`.github/workflows`) を用いた継続的インテグレーション(CI)と継続的デプロイ(CD)の設定手順です。

### CI ワークフロー (`ci.yml`)
コードが `main` ブランチにプッシュされるか、プルリクエストが作成された時に自動実行されます。
- バックエンドの単体テスト & 結合テスト
- フロントエンドの単体テスト (Vitest)
- Playwright による E2E テスト (Docker Compose 環境を CI 上で一時的に立ち上げてテストします)
*※ CI 実行にあたって特別な事前設定 (GitHub Secrets 等) は不要です。*

### CD ワークフロー (`deploy.yml`)
CI ワークフローが正常に完了した際、または手動 (`workflow_dispatch`) で実行され、自宅サーバー等のデプロイ先環境へアプリケーションを自動デプロイします。

CDを正常に動作させるには、GitHub リポジトリの設定で以下の **Secrets**、**Variables**、および**デプロイ環境**を設定する必要があります。

#### 1. GitHub Variables (環境変数)
ビルド時に必要な非機密情報は GitHub Variables に設定します。
- **`NEXT_PUBLIC_API_URL`**: Next.js のビルド時に埋め込まれる API のエンドポイント URL (例: `https://stg-api.yourdomain.com/api` または `https://api.yourdomain.com/api`)。
  *※ GitHub の Repository > Settings > Secrets and variables > Variables に登録します。環境 (Environment) ごとに異なる値を設定することも可能です。*

#### 2. GitHub Secrets (機密情報)
データベースの認証情報などの機密情報は GitHub Secrets に設定します。
- **`MYSQL_ROOT_PASSWORD`**: デプロイ先 MySQL の root パスワード
- **`MYSQL_DATABASE`**: デプロイ先で作成するデータベース名
- **`MYSQL_USER`**: デプロイ先で作成するアプリケーション接続ユーザー名
- **`MYSQL_PASSWORD`**: デプロイ先アプリケーション接続ユーザーのパスワード
- **`MYSQL_DSN`**: バックエンドが MySQL に接続するための DSN (例: `app_user:password@tcp(mysql:3306)/appdb?parseTime=true&loc=Local`)
- **`CLOUDFLARE_TUNNEL_TOKEN`** (任意): Cloudflare Tunnel の接続トークン
- **`BASIC_AUTH_USER`** (任意/stg用): ステージング環境等で Basic 認証を有効にする場合のユーザー名
- **`BASIC_AUTH_PASS`** (任意/stg用): ステージング環境等で Basic 認証を有効にする場合のパスワード

#### 3. デプロイ先環境（セルフホストランナー）のセットアップ
`deploy.yml` は `runs-on: self-hosted` として定義されており、デプロイ対象のサーバー上でセルフホストランナーが動作していることを前提としています。

1. **GitHub Actions Self-hosted Runner の登録**
   デプロイ先のサーバーで GitHub Actions Runner をインストールし、リポジトリに登録します。
2. **必要なツールのインストール**
   サーバー上に `podman`、`podman-compose`、および `systemd` がインストールされ、ユーザー権限で実行できる状態である必要があります。
3. **作業ディレクトリの用意**
   ランナーを実行するユーザーのホームディレクトリ配下に、プロジェクトディレクトリを作成します。
   ```bash
   mkdir -p ~/projects/boilerplate
   ```
4. **systemd ユーザーサービスの作成**
   デプロイ先サーバーでコンテナ起動ライフサイクルを管理するため、systemd のユーザー用サービスファイルを配置します（例: `~/.config/systemd/user/boilerplate-stg.service` または `boilerplate-prod.service`）。
   以下は systemd サービスファイルの構成例です。

   `~/.config/systemd/user/boilerplate-stg.service`:
   ```ini
   [Unit]
   Description=Boilerplate Application Suite (stg)
   After=network.target

   [Service]
   Type=simple
   WorkingDirectory=%h/projects/boilerplate
   Environment=XDG_RUNTIME_DIR=/run/user/%U
   ExecStartPre=/usr/bin/podman-compose -p boilerplate-stg --env-file .env.stg pull
   ExecStart=/usr/bin/podman-compose -p boilerplate-stg --env-file .env.stg up
   ExecStop=/usr/bin/podman-compose -p boilerplate-stg --env-file .env.stg down
   Restart=always

   [Install]
   WantedBy=default.target
   ```
   設定後、以下のコマンドでサービスを有効化します。
   ```bash
   systemctl --user daemon-reload
   systemctl --user enable boilerplate-stg.service
   # ユーザーがログアウトしてもプロセスが実行され続けるように linger を有効化
   loginctl enable-linger
   ```

