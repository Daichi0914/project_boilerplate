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
