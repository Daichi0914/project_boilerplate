# boilerplate

本プロジェクトは、Go (Clean Architecture) と Next.js (App Router) をベースにした Web アプリケーション開発用のプロジェクトです。

テンプレートからプロジェクトを作成したのち、以下の手順に従って環境を立ち上げてください。

<!-- START_TEMPLATE_ONLY -->
## 最初に必要な設定

### GitHub Actions の書き込み権限の確認 (※初回のみ)
初期化ワークフローがプロジェクト名への自動置換と不要ファイルのクリーンアップを完了するために、Actions の書き込み権限が必要です。
1. 作成したリポジトリの **Settings** -> **Actions** -> **General** へ移動します。
2. **Workflow permissions** セクションで **Read and write permissions** を選択し、保存します。
3. **Actions** タブに移動し、**Initialize Boilerplate with Repository Name** ワークフローを選択後、「**Run workflow**」（手動実行）をクリックして再実行してください（初回の自動実行が権限エラーで失敗している場合のみ）。

※ 実行が完了すると、自動的にすべてのファイル内の `boilerplate` 文字列が本リポジトリ名に置換され、初期化ワークフロー定義ファイル自身も自動的に削除されます。

---
<!-- END_TEMPLATE_ONLY -->

## ローカル環境のセットアップと起動

1. **環境変数のコピー:**
   ```bash
   cp .env.example .env
   cp .env.e2e.example .env.e2e
   ```

2. **フロントエンド依存パッケージのインストール:**
   ```bash
   cd frontend
   npm install
   cd ..
   ```

3. **コンテナ環境の起動:**
   Make コマンドを利用して、MySQL、Redis、Nginx、Backend、Frontend を一括で起動します。
   ```bash
   make up
   ```

### 3. 起動の確認
コンテナ起動後、ブラウザで以下のアドレスにアクセスして接続ステータス（API、DB、Redis）を確認できます。
- **ダッシュボード:** [http://localhost:8080](http://localhost:8080)
- **Frontend 開発サーバー:** [http://localhost:3000](http://localhost:3000)

---

## デプロイ（Staging / Production）について

本プロジェクトには自動デプロイ用のワークフロー（`.github/workflows/deploy.yml`）が用意されています。デプロイ環境を構築する際は、GitHub Actions の Environment に必要な環境変数およびシークレットを設定してください。

必要な設定項目については、`.env.example`の末尾にある `GitHub Actions / CD (Deploy) Settings` セクションを参照してください。

### デプロイ先サーバー（セルフホストランナー）の事前準備
デプロイワークフロー（`deploy.yml`）は `runs-on: self-hosted` を指定しているため、対象サーバーが GitHub Actions のセルフホストランナーとして登録されている必要があります。

1. **GitHub Actions セルフホストランナーのセットアップ**
   - リポジトリの **Settings** -> **Actions** -> **Runners** に移動し、**New self-hosted runner** をクリックします。
   - プラットフォームの選択画面で **Linux** を選択し、表示される手順に従ってサーバー上でランナーをダウンロードおよびセットアップしてください。
   - ランナーをサーバー上で常時稼働させるため、セットアップ完了後にランナーディレクトリ内で `./svc.sh install` および `./svc.sh start` を実行し、システムサービスとして登録・起動することを推奨します。
     > [!NOTE]
     > **自分向けのメモ:**  
     > ホームサーバー上では `~/actions-runner/` 配下にプロジェクトごとのディレクトリを作成し、その中でダウンロードおよび `./svc.sh` によるシステムサービス化を実行する運用としています。

2. **`podman-compose` のインストールとパスの確認**
   - ワークフロー内の systemd サービス定義は `/usr/bin/podman-compose` に実行ファイルが存在することを前提としています。
   - `which podman-compose` の実行結果が異なるパス（例: `/usr/local/bin/podman-compose` や `~/.local/bin/podman-compose` など）である場合は、`.github/workflows/deploy.yml` の `ExecStart`/`ExecStop` のパスを実際のパスに修正してください。

3. **systemd ユーザーサービスの常時起動設定 (Linger)**
   - デプロイしたコンテナ（systemd ユーザーサービス）が、ランナーユーザーのログアウト時やサーバー再起動時にも常時稼働し続けるように、サーバー側で以下のコマンドを実行して Linger を有効化してください。
     ```bash
     sudo loginctl enable-linger <ランナーを実行するユーザー名>
     ```
