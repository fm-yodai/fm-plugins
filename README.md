# fm-plugins

株式会社FREEMIND 社内向けの **Claude Code プラグインマーケットプレイス** です。マーケットプレイス名は `fm-plugins` で、プラグインは `プラグイン名@fm-plugins` の形で指定します。

現在含まれるプラグイン:

| プラグイン | 用途 | 呼び出し |
|---|---|---|
| `hello` | 導入確認用サンプル。名前・バージョン・更新コマンドを表示 | `/hello:hello` |

## 前提

- Claude Code がインストール済みで、Anthropic Team プランのアカウントでログインしていること
- GitHub Organization `koo-inc` の private リポジトリに **git でアクセスできる**こと。Claude Code はあなたの git 認証（SSH 鍵、または HTTPS の credential helper）をそのまま使います。`git clone git@github.com:koo-inc/fm-plugins.git` が通れば大丈夫です

## 導入

```bash
claude plugin marketplace add koo-inc/fm-plugins
claude plugin install hello@fm-plugins
```

Claude Code を再起動（または `/reload-plugins`）し、次を実行して動作を確認します。

```
/hello:hello
```

「fm-plugins マーケットプレイスからインストールされた hello プラグイン v0.1.0 が動作しています」と表示されれば導入完了です。

## 更新

```bash
claude plugin marketplace update fm-plugins   # マーケットプレイス定義を最新化
claude plugin update hello@fm-plugins         # プラグインを最新化（再起動で反映）
```

## プラグインを追加する

1. `plugins/<name>/` を作り、`.claude-plugin/plugin.json`（`name` と `version` は必須にする）と `skills/<skill>/SKILL.md` などを置く。`plugins/hello/` をコピーして書き換えるのが早い
2. `.claude-plugin/marketplace.json` の `plugins` 配列にエントリを 1 つ追加する

   ```json
   { "name": "<name>", "source": "./plugins/<name>", "description": "..." }
   ```

3. 検証を通す

   ```bash
   claude plugin validate --strict .                 # マーケットプレイス
   claude plugin validate --strict plugins/<name>    # プラグイン
   ```

4. 作業コピーからインストールして動作を確認する（次節）
5. 既存プラグインを変更したときは、その `plugin.json` の `version` を上げる（利用者の `claude plugin update` はこの値で更新を判断する）
6. ブランチを切って PR を作成する。`main` の最新コミットがそのまま利用者に配布されるため、`main` へ直接 push しない

### 作業コピーで動作確認する

push 前に、手元のクローンをそのままマーケットプレイスとして登録して確認できます。マーケットプレイス名が同じ `fm-plugins` になるため GitHub 版と同時には登録できず、確認中は一時的に付け替えます。

```bash
# GitHub 版 → 作業コピー
claude plugin marketplace remove fm-plugins
claude plugin marketplace add "$PWD"            # リポジトリ直下で実行
claude plugin install <name>@fm-plugins
```

Claude Code を再起動（または `/reload-plugins`）して `/<name>:<skill>` を実行し、期待どおり動くことを確認します。終わったら GitHub 版に戻します。

```bash
# 作業コピー → GitHub 版
claude plugin uninstall <name>@fm-plugins
claude plugin marketplace remove fm-plugins
claude plugin marketplace add koo-inc/fm-plugins
claude plugin install <name>@fm-plugins
```

### 守ること

- プラグインは必ずこのリポジトリ内 `plugins/<name>/` に置き、相対パスで参照する（管理コンソールから組織配布する際の条件）
- トップレベルに `bin/` を置かない（同上）
- private リポジトリでも、トークンや鍵などの秘密情報はコミットしない

## 初期セットアップ（リポジトリ作成まで）

`koo-inc/fm-plugins` が GitHub 上にできるまでの一回限りの手順です。完了したらこの節は削除してください。

作成者（fm-yodai）は `koo-inc` の member で、Org 設定上メンバーはリポジトリを作成できません（private も不可）。

### 1. Org Owner に依頼する

1. `koo-inc/fm-plugins` を **private** で作成する（README / .gitignore / LICENSE は生成せず空のまま。`gh repo create koo-inc/fm-plugins --private` でも可）
2. 作成者に **Write 以上**の権限を付与する（`main` のブランチ保護まで作成者に任せる場合は Admin）
3. 利用者となるメンバーが Read できることを確認する（Org の base permission が Read 以上か、該当チームに Read を付与）

### 2. 初回 push（作成者）

```bash
cd ~/fm-plugins
git remote add origin git@github.com:koo-inc/fm-plugins.git
git push -u origin main
gh repo view koo-inc/fm-plugins
```

SSH で弾かれる場合は `git remote set-url origin https://github.com/koo-inc/fm-plugins.git` に切り替え、`gh auth setup-git` で gh の認証を git に使わせます。

### 3. ローカルを GitHub 版に付け替えて確認（作成者）

作成者の環境では作業コピーが `fm-plugins` として登録されているので、GitHub 版に付け替えて配布経路を確認します。

```bash
claude plugin uninstall hello@fm-plugins
claude plugin marketplace remove fm-plugins
claude plugin marketplace add koo-inc/fm-plugins
claude plugin install hello@fm-plugins
```

Claude Code を再起動して `/hello:hello` が動けば、GitHub 経由の配布が成立しています。

### 4. 仕上げ

- 社内メンバーに「導入」節のコマンドを案内する
- 可能なら `main` にブランチ保護（PR 必須）を設定する
- この節を削除する

## 今後の拡張

- **案 B（自動配布・CI）:** 社内プロジェクトの `.claude/settings.json` に貼るスニペット（`extraKnownMarketplaces` + `enabledPlugins`）を `templates/` に用意し、リポジトリを開いた人に自動でインストールを提案させる。GitHub Actions で `claude plugin validate --strict` を回し、`claude plugin tag --push` でリリースタグを打つ
- **案 C（組織配布）:** Team プラン管理者が Claude.ai 管理コンソールの Organization settings > Plugins にこのリポジトリを登録し、組織全体に配布する。必要なら managed settings の `strictKnownMarketplaces` で許可マーケットプレイスを固定する

## 参考

- https://code.claude.com/docs/en/plugins.md
- https://code.claude.com/docs/en/plugin-marketplaces.md
- https://code.claude.com/docs/en/admin-setup.md
