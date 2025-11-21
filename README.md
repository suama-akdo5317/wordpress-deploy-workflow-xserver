# WordPress Deploy Workflow

XserverへのWordPressデプロイを自動化するGitHub Actionsワークフローです。

## 概要

このリポジトリには、WordPressのテーマとプラグインをXserverに自動デプロイするためのGitHub Actionsワークフローが含まれています。

## ワークフロー構成

### deploy.yml

**デプロイ対象:**
- `wordpress/wp-content/plugins/` - WordPressプラグイン
- `wordpress/wp-content/themes/` - WordPressテーマ（`twenty*`テーマは除外）

**トリガー:**
- `main`ブランチへのpush → Production環境へデプロイ
- `staging`ブランチへのpush → Staging環境へデプロイ
- 手動実行（workflow_dispatch）→ 環境を選択してデプロイ

**デプロイ先:**
- Production: `REMOTE_TARGET_PROD`で指定されたパス
- Staging: `REMOTE_TARGET_STAGING`で指定されたパス

## セットアップ

### 必要なシークレット変数

GitHubリポジトリの Settings > Secrets and variables > Actions で以下のシークレットを設定してください:

| シークレット名 | 説明 | 例 |
|--------------|------|-----|
| `SSH_PRIVATE_KEY` | SSH秘密鍵 | `-----BEGIN OPENSSH PRIVATE KEY-----...` |
| `REMOTE_HOST` | デプロイ先ホスト名 | `xb766649.xbiz.jp` |
| `REMOTE_USER` | SSH接続ユーザー名 | `xb766649` |
| `REMOTE_TARGET_PROD` | Production環境のベースパス | `/home/xb766649/xb766649.xbiz.jp/public_html` |
| `REMOTE_TARGET_STAGING` | Staging環境のベースパス | `/home/xb766649/staging.xb766649.xbiz.jp/public_html` |

**注意:**
- XserverのSSH接続はポート`10022`を使用します
- `REMOTE_TARGET_*`には`/wp-content`を含めない基本パスを指定してください（ワークフローが自動的に`/wp-content`を追加します）

### ディレクトリ構成

リポジトリのルートに以下の構成でWordPressファイルを配置してください:

```
.
├── .github/
│   └── workflows/
│       └── deploy.yml
└── wordpress/
    └── wp-content/
        ├── plugins/
        │   └── your-plugin/
        └── themes/
            └── your-theme/
```

## 使い方

### 自動デプロイ

- `main`ブランチにpush → 自動的にProduction環境へデプロイ
- `staging`ブランチにpush → 自動的にStaging環境へデプロイ

### 手動デプロイ

1. GitHubリポジトリの「Actions」タブを開く
2. 「Deploy to Xserver」ワークフローを選択
3. 「Run workflow」をクリック
4. デプロイ先の環境（staging/production）を選択
5. 「Run workflow」を実行

### 新しい環境を追加する場合

[deploy.yml](.github/workflows/deploy.yml)の`Determine deploy target`ステップを編集して、新しいブランチと環境変数を追加してください:

```yaml
- name: Determine deploy target
  id: target
  run: |
    if [ "${{ github.ref }}" = "refs/heads/main" ]; then
      echo "base_path=${{ secrets.REMOTE_TARGET_PROD }}/wp-content" >> $GITHUB_OUTPUT
    elif [ "${{ github.ref }}" = "refs/heads/develop" ]; then
      echo "base_path=${{ secrets.REMOTE_TARGET_DEV }}/wp-content" >> $GITHUB_OUTPUT
    else
      echo "base_path=${{ secrets.REMOTE_TARGET_STAGING }}/wp-content" >> $GITHUB_OUTPUT
    fi
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

## FAQ

### `workflow_dispatch`の環境選択は機能しますか?

現在のワークフローでは、手動実行時の環境選択入力は定義されていますが、実際のデプロイ先はpushされたブランチ名で判定されます。手動実行時に環境を切り替えたい場合は、ワークフローの修正が必要です。

### Staging環境が不要な場合は?

[deploy.yml](.github/workflows/deploy.yml)の`on.push.branches`から`staging`を削除してください:

```yaml
on:
  push:
    branches:
      - main  # stagingを削除
```

## ライセンス

このワークフローは自由に使用・改変できます。
