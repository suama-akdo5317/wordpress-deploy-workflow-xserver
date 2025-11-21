# WordPress Deploy Workflow

XserverへのWordPressデプロイを自動化するGitHub Actions ワークフロー集です。

## 概要

このリポジトリには、WordPressのテーマとプラグインをXserverに自動デプロイするためのGitHub Actionsワークフローが含まれています。再利用可能なワークフロー設計により、環境に応じて必要なワークフローのみを使用できます。

## ワークフロー構成

### 1. deploy-reusable.yml（再利用可能ワークフロー）

共通のデプロイロジックを含む基盤ワークフロー。他のワークフローから呼び出されます。

**デプロイ対象:**
- `wp-content/plugins/` - WordPressプラグイン
- `wp-content/themes/` - WordPressテーマ（`twenty*`テーマは除外）

### 2. deploy-production.yml（Production環境）

**トリガー:**
- `main`ブランチへのpush
- 手動実行（workflow_dispatch）

**デプロイ先:** `REMOTE_TARGET_PROD`で指定されたパス

### 3. deploy-staging.yml（Staging環境）

**トリガー:**
- `staging`ブランチへのpush
- 手動実行（workflow_dispatch）

**デプロイ先:** `REMOTE_TARGET_STAGING`で指定されたパス

## セットアップ

### 必要なシークレット変数

GitHubリポジトリの Settings > Secrets and variables > Actions で以下のシークレットを設定してください:

| シークレット名 | 説明 | 例 |
|--------------|------|-----|
| `SSH_PRIVATE_KEY` | SSH秘密鍵 | `-----BEGIN OPENSSH PRIVATE KEY-----...` |
| `REMOTE_HOST` | デプロイ先ホスト名 | `example.xsrv.jp` |
| `REMOTE_USER` | SSH接続ユーザー名 | `xserver-user` |
| `REMOTE_TARGET_PROD` | Production環境のパス | `/home/user/example.com/public_html` |
| `REMOTE_TARGET_STAGING` | Staging環境のパス | `/home/user/staging.example.com/public_html` |

### ディレクトリ構成

リポジトリのルートに以下の構成でWordPressファイルを配置してください:

```
.
├── .github/
│   └── workflows/
│       ├── deploy-reusable.yml
│       ├── deploy-production.yml
│       └── deploy-staging.yml
└── wordpress/
    └── wp-content/
        ├── plugins/
        │   └── your-plugin/
        └── themes/
            └── your-theme/
```

## 使い方

### Staging環境が不要な場合

`deploy-staging.yml`を削除してください:

```bash
rm .github/workflows/deploy-staging.yml
```

### 新しい環境を追加する場合

`deploy-production.yml`や`deploy-staging.yml`を参考に、新しいワークフローファイルを作成してください:

```yaml
name: Deploy to Development

on:
  push:
    branches:
      - develop
  workflow_dispatch:

permissions:
  contents: read

jobs:
  deploy:
    uses: ./.github/workflows/deploy-reusable.yml
    with:
      environment: development
      remote_target: ${{ secrets.REMOTE_TARGET_DEV }}
    secrets:
      SSH_PRIVATE_KEY: ${{ secrets.SSH_PRIVATE_KEY }}
      REMOTE_HOST: ${{ secrets.REMOTE_HOST }}
      REMOTE_USER: ${{ secrets.REMOTE_USER }}
```

## 技術仕様

- **SSH接続:** ポート10022を使用（Xserverのデフォルト）
- **同期方法:** rsync（`-avz --delete`オプション）
- **セキュリティ:** SSH秘密鍵は実行後に自動削除

## トラブルシューティング

### デプロイが失敗する場合

1. シークレット変数が正しく設定されているか確認
2. SSH秘密鍵のフォーマットが正しいか確認（OpenSSH形式）
3. リモートホストへのSSH接続が可能か確認（ポート10022）
4. デプロイ先パスに書き込み権限があるか確認

### known_hostsエラーが発生する場合

ワークフロー内で`ssh-keyscan`を使用してホストキーを自動取得しているため、通常は発生しません。問題が発生する場合は、`REMOTE_HOST`の設定を確認してください。

## ライセンス

このワークフロー集は自由に使用・改変できます。
