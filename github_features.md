# GitHub機能ガイド

## 1. Code管理

### リポジトリの基本操作
```bash
# クローン
git clone https://github.com/USERNAME/REPO.git

# ブランチ作成と切り替え
git checkout -b feature/新機能

# 変更の追跡
git status
git diff

# 変更の保存
git add .
git commit -m "feat: 新機能を追加"
git push origin feature/新機能
```

### コードブラウジング
- ファイル履歴の確認: `Blame`ビュー
- コミット履歴の確認: `History`ビュー
- コードナビゲーション: `Go to file` (キーボードショートカット: `t`)
- ブランチ比較: `Compare`ビュー

## 2. Issues

### イシュー管理
```bash
# イシューの作成
gh issue create --title "バグ: ログイン機能が動作しない" --body "詳細な説明"

# イシューの一覧表示
gh issue list

# イシューの割り当て
gh issue edit 1 --assignee USERNAME

# イシューのクローズ
gh issue close 1
```

### イシューのベストプラクティス
1. テンプレートの活用
2. ラベルの適切な使用
3. マイルストーンとの紐付け
4. 関連PRのリンク

## 3. Pull Requests

### PR管理
```bash
# PRの作成
gh pr create --base main --head feature/新機能 --title "新機能の追加" --body "機能の説明"

# PRの一覧表示
gh pr list

# PRのレビュー
gh pr review 1 --approve
gh pr review 1 --comment "修正が必要です"

# PRのマージ
gh pr merge 1 --merge --delete-branch
```

### PRのベストプラクティス
1. 適切なサイズ管理（大きすぎないこと）
2. レビュー前のセルフチェック
3. CI/CDの結果確認
4. コードオーナーの確認

## 4. Actions

### ワークフロー管理
```yaml
# .github/workflows/ci.yml
name: CI Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Run tests
        run: npm test
```

### Actionsの操作
```bash
# ワークフローの一覧表示
gh workflow list

# ワークフローの実行
gh workflow run ci.yml

# 実行結果の確認
gh run list
gh run view [run-id]
```

## 5. Projects

### プロジェクト管理
1. プロジェクトの作成
   - `Projects` → `New project`
   - テンプレートの選択（かんばんボード等）

2. カスタムフィールドの追加
   - Status（ToDo, In Progress, Done）
   - Priority（High, Medium, Low）
   - Sprint（Sprint 1, Sprint 2...）

3. イシューとPRの紐付け
   - ドラッグ&ドロップで追加
   - `#`でイシュー番号を参照

## 6. Wiki

### Wiki管理
```bash
# Wikiのクローン
git clone https://github.com/USERNAME/REPO.wiki.git

# ページの作成/編集
echo "# Welcome" > Home.md
git add Home.md
git commit -m "docs: Wikiのホームページを追加"
git push
```

### Wikiの構成例
1. Home（概要）
2. Getting Started（導入ガイド）
3. API Documentation（API仕様）
4. Contributing Guidelines（貢献ガイド）
5. FAQ（よくある質問）

## 7. Security

### セキュリティ設定
1. 脆弱性アラート
   - `Settings` → `Security & analysis`
   - Dependabot alertsの有効化

2. コードスキャン
   - `Settings` → `Security & analysis`
   - Code scanningの設定

3. シークレット管理
```bash
# シークレットの設定
gh secret set API_KEY --body "your-api-key"

# シークレットの使用（GitHub Actions）
${{ secrets.API_KEY }}
```

## 8. Insights

### 分析機能
1. Pulse
   - アクティビティの概要
   - コミット数、PR数、イシュー数

2. Contributors
   - 貢献者の統計
   - コミット頻度

3. Traffic
   - クローン数
   - ビュー数

4. Dependencies
   - 依存関係グラフ
   - 脆弱性アラート

## 9. Settings

### リポジトリ設定
1. 基本設定
   - リポジトリ名
   - 説明
   - 可視性

2. ブランチ保護
```bash
# メインブランチの保護
gh api repos/OWNER/REPO/branches/main/protection \
  --method PUT \
  --field required_status_checks=null \
  --field enforce_admins=true \
  --field required_pull_request_reviews='{"required_approving_review_count":1}'
```

3. Webhook設定
```bash
# Webhookの追加
gh api repos/OWNER/REPO/hooks \
  --method POST \
  --field url=https://example.com/webhook \
  --field content_type=json
```

## 10. Releases

### リリース管理
```bash
# タグの作成
git tag -a v1.0.0 -m "バージョン1.0.0"
git push origin v1.0.0

# リリースの作成
gh release create v1.0.0 \
  --title "バージョン1.0.0" \
  --notes "リリースノート" \
  --draft

# リリースの公開
gh release edit v1.0.0 --draft=false
```

### リリースノートの例
```markdown
# バージョン1.0.0

## 新機能
- ユーザー認証機能の追加
- ダッシュボードの実装

## バグ修正
- ログイン画面のレイアウト崩れを修正
- パフォーマンスの改善

## 変更点
- APIエンドポイントの更新
- 依存パッケージのアップデート
```

## 11. Packages

### パッケージ管理
```bash
# パッケージの公開（npm例）
npm publish

# パッケージの公開（Docker例）
docker tag image:latest ghcr.io/USERNAME/image:latest
docker push ghcr.io/USERNAME/image:latest
```

### パッケージ設定
1. パッケージの可視性設定
2. アクセス権限の管理
3. バージョニングの設定

## 12. Deployments

### デプロイメント管理
```yaml
# GitHub Actionsでのデプロイメント例
name: Deploy
on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Deploy to production
        run: |
          echo "デプロイ処理"
        env:
          DEPLOY_KEY: ${{ secrets.DEPLOY_KEY }}
```

### デプロイメント環境
1. 環境の設定
   - `Settings` → `Environments`
   - 環境変数の設定
   - デプロイメントルールの設定

2. 環境の保護
   - 必要な承認者の設定
   - 待機時間の設定
   - ブランチ制限の設定 