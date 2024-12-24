# GitHubリポジトリセットアップガイド

## 0. GitHubトークンの取得と設定

### Personal Access Token (PAT) の作成

1. GitHubにログインし、右上のプロフィールアイコンから`Settings`を選択
2. 左サイドバーの一番下にある`Developer settings`をクリック
3. 左サイドバーから`Personal access tokens`→`Tokens (classic)`を選択
4. `Generate new token`→`Generate new token (classic)`をクリック
5. トークンの設定:
   - `Note`: トークンの用途を記述（例：「開発用アクセストークン」）
   - `Expiration`: 有効期限を設定（セキュリティのため、必要な期間のみ）
   - 必要なスコープにチェック:
     - `repo`: すべてにチェック（リポジトリの完全な制御）
     - `workflow`: GitHub Actionsの制御
     - `admin:org`: Organization設定（必要な場合）
     - `delete_repo`: リポジトリの削除権限（必要な場合）
6. `Generate token`をクリック
7. 表示されたトークンを安全な場所にコピー（この画面を閉じると二度と表示されません）

### トークンの設定方法

```bash
# 環境変数として設定（一時的）
export GITHUB_TOKEN="your-token"

# Windowsでの永続的な設定
[Environment]::SetEnvironmentVariable("GITHUB_TOKEN", "your-token", "User")

# macOS/Linuxでの永続的な設定
echo "export GITHUB_TOKEN=your-token" >> ~/.bashrc  # bashの場合
echo "export GITHUB_TOKEN=your-token" >> ~/.zshrc   # zshの場合

# GitHub CLIでの認証
gh auth login --with-token < token.txt
# または
echo "your-token" | gh auth login --with-token
```

### セキュリティのベストプラクティス

1. トークンの保護
   - トークンを直接コードにハードコーディングしない
   - パブリックリポジトリでトークンを公開しない
   - トークンを誤ってコミットした場合は即座に無効化

2. スコープの最小化
   - 必要最小限の権限のみを付与
   - 不要なスコープは選択しない

3. 有効期限の設定
   - 長期的なトークンは避ける
   - 定期的にトークンを更新

4. 環境変数の使用
   - CI/CDでは環境変数として設定
   - ローカル開発でも環境変数を使用

## 1. 初期セットアップ

```bash
# GitHubトークンの設定
export GITHUB_TOKEN="your-token"

# リポジトリの初期化
git init
git add .
git commit -m "初期コミット"

# GitHubリポジトリの作成とリモート設定
gh repo create version-management --private --source=. --remote=origin
git branch -M main
git push -u origin main
```

## 2. ブランチ構造のセットアップ

```bash
# 開発ブランチの作成
git checkout -b develop
git push -u origin develop

# ステージングブランチの作成
git checkout -b staging
git push -u origin staging

# 各ブランチの保護設定
gh api repos/OWNER/REPO/branches/main/protection \
  --method PUT \
  --field required_status_checks=null \
  --field enforce_admins=true \
  --field required_pull_request_reviews='{"required_approving_review_count":1}' \
  --field restrictions=null

# developとstagingブランチにも同様の保護設定を適用
for branch in "develop" "staging"; do
  gh api repos/OWNER/REPO/branches/$branch/protection \
    --method PUT \
    --field required_status_checks=null \
    --field enforce_admins=true \
    --field required_pull_request_reviews='{"required_approving_review_count":1}' \
    --field restrictions=null
done
```

## 3. GitHub Actionsワークフローの設定

```bash
# ワークフローディレクトリの作成
mkdir -p .github/workflows

# CI/CDワークフローファイルの作成
cat > .github/workflows/ci-cd.yml << 'EOF'
name: CI/CD Pipeline

on:
  push:
    branches: [ main, develop, staging ]
  pull_request:
    branches: [ main, develop ]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    - name: Run tests
      run: echo "テストを実行" # 実際のテストコマンドに置き換え

  version:
    needs: test
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - name: Bump version and push tag
      uses: anothrNick/github-tag-action@1.36.0
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        WITH_V: true
        DEFAULT_BUMP: patch
EOF

git add .github/workflows/ci-cd.yml
git commit -m "ci: GitHub Actionsワークフローを追加"
git push
```

## 4. イシューテンプレートの設定

```bash
# イシューテンプレートディレクトリの作成
mkdir -p .github/ISSUE_TEMPLATE

# バグ報告テンプレート
cat > .github/ISSUE_TEMPLATE/bug_report.md << 'EOF'
---
name: バグ報告
about: バグの報告用テンプレート
title: '[BUG] '
labels: bug
assignees: ''
---

## バグの説明
簡潔な説明を記載してください。

## 再現手順
1. まず '...'
2. 次に '....'
3. その後 '....'
4. エラーが発生

## 期待される動作
期待される動作の説明を記載してください。

## スクリーンショット
可能であれば、スクリーンショットを追加してください。
EOF

# 機能リクエストテンプレート
cat > .github/ISSUE_TEMPLATE/feature_request.md << 'EOF'
---
name: 機能リクエスト
about: 新機能のリクエスト用テンプレート
title: '[FEATURE] '
labels: enhancement
assignees: ''
---

## 関連する問題
問題の説明を記載してください。

## 提案する解決策
解決策の説明を記載してください。

## 代替案の検討
検討した他の解決策があれば記載してください。

## その他の情報
実装に関する追加情報があれば記載してください。
EOF

git add .github/ISSUE_TEMPLATE
git commit -m "docs: イシューテンプレートを追加"
git push
```

## 5. プルリクエストテンプレートの設定

```bash
# プルリクエストテンプレートの作成
cat > .github/pull_request_template.md << 'EOF'
## 変更の種類
- [ ] バグ修正
- [ ] 新機能
- [ ] 破壊的変更
- [ ] ドキュメント更新

## 変更内容
変更の概要を記載してください。

## 関連イシュー
関連するイシュー番号を記載してください。

## テスト
- [ ] ユニットテスト
- [ ] 統合テスト
- [ ] E2Eテスト

## レビューチェックリスト
- [ ] コーディング規約に準拠している
- [ ] 適切なテストが追加されている
- [ ] ドキュメントが更新されている
EOF

git add .github/pull_request_template.md
git commit -m "docs: プルリクエストテンプレートを追加"
git push
```

## 6. セキュリティ設定

```bash
# セキュリティポリシーの作成
mkdir -p .github/SECURITY.md

cat > .github/SECURITY.md << 'EOF'
# セキュリティポリシー

## サポートされているバージョン

| バージョン | サポート状況 |
| -------- | ----------- |
| 1.x.x    | ✅          |
| < 1.0    | ❌          |

## 脆弱性の報告方法

セキュリティの脆弱性を発見した場合は、以下の手順で報告してください：

1. イシューを作成せず、直接管理者に連絡
2. 脆弱性の詳細な説明を提供
3. 可能であれば、修正案も提供

報告された脆弱性は優先的に対応いたします。
EOF

git add .github/SECURITY.md
git commit -m "docs: セキュリティポリシーを追加"
git push
```

## 7. リポジトリの設定確認

```bash
# ブランチ保護の確認
gh api repos/OWNER/REPO/branches/main/protection

# Actionsの確認
gh workflow list

# コラボレーターの確認
gh api repos/OWNER/REPO/collaborators

# Webhookの確認
gh api repos/OWNER/REPO/hooks
```

## 8. 日常的な開発フロー

```bash
# 機能開発
git checkout develop
git pull
git checkout -b feature/新機能名
# ... 開発作業 ...
git add .
git commit -m "feat: 新機能を追加"
git push -u origin feature/新機能名

# プルリクエスト作成
gh pr create --base develop --head feature/新機能名 --title "新機能の追加" --body "機能の説明"

# レビュー後のマージ
gh pr merge --merge --delete-branch

# バージョンタグの作成
git tag -a v1.0.0 -m "バージョン1.0.0リリース"
git push origin v1.0.0
``` 