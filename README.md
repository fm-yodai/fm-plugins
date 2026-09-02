# fm-claude-plugins

株式会社FREEMIND 社内向けの **Claude Code プラグインマーケットプレイス** です。マーケットプレイス名は `fm-plugins` で、プラグインは `プラグイン名@fm-plugins` の形で指定します。

現在含まれるプラグイン:

| プラグイン | 用途 | 呼び出し |
|---|---|---|
| `hello` | 導入確認用サンプル。名前・バージョン・更新コマンドを表示 | `/hello:hello` |

## 前提

- Claude Code がインストール済みで、Anthropic Team プランのアカウントでログインしていること
- GitHub Organization `koo-inc` の private リポジトリに **git でアクセスできる**こと。Claude Code はあなたの git 認証（SSH 鍵、または HTTPS の credential helper）をそのまま使います。`git clone git@github.com:koo-inc/fm-claude-plugins.git` が通れば大丈夫です

## 導入

```bash
claude plugin marketplace add koo-inc/fm-claude-plugins
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

## リポジトリ公開前の暫定手順（作成者向け）

`koo-inc/fm-claude-plugins` が作成される前は、ローカルのクローンをマーケットプレイスとして追加できます。

```bash
claude plugin marketplace add /home/yodai/fm-claude-plugins
claude plugin install hello@fm-plugins
```

GitHub 側が使えるようになったら、ローカル版を外して GitHub 版に付け替えます。

```bash
claude plugin uninstall hello@fm-plugins
claude plugin marketplace remove fm-plugins
claude plugin marketplace add koo-inc/fm-claude-plugins
claude plugin install hello@fm-plugins
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

4. 既存プラグインを変更したときは、その `plugin.json` の `version` を上げる（利用者の `claude plugin update` はこの値で更新を判断する）
5. PR を作成する

### 守ること

- プラグインは必ずこのリポジトリ内 `plugins/<name>/` に置き、相対パスで参照する（管理コンソールから組織配布する際の条件）
- トップレベルに `bin/` を置かない（同上）
- private リポジトリでも、トークンや鍵などの秘密情報はコミットしない

## Org Owner への依頼事項

作成者（fm-yodai）は `koo-inc` の member でリポジトリ作成権限がありません。次を依頼してください。

1. `koo-inc/fm-claude-plugins` を **private** で作成する（README や .gitignore は生成しない）
2. 作成者に **Write 以上**の権限を付与する
3. 利用者となるメンバーが Read できることを確認する（Org メンバー全員なら base permission が Read 以上か）

作成後、作成者が次を実行して初回 push します。

```bash
cd ~/fm-claude-plugins
git remote add origin git@github.com:koo-inc/fm-claude-plugins.git
git push -u origin main
```

## 今後の拡張

- **案 B（自動配布・CI）:** 社内プロジェクトの `.claude/settings.json` に貼るスニペット（`extraKnownMarketplaces` + `enabledPlugins`）を `templates/` に用意し、リポジトリを開いた人に自動でインストールを提案させる。GitHub Actions で `claude plugin validate --strict` を回し、`claude plugin tag --push` でリリースタグを打つ
- **案 C（組織配布）:** Team プラン管理者が Claude.ai 管理コンソールの Organization settings > Plugins にこのリポジトリを登録し、組織全体に配布する。必要なら managed settings の `strictKnownMarketplaces` で許可マーケットプレイスを固定する

## 参考

- https://code.claude.com/docs/en/plugins.md
- https://code.claude.com/docs/en/plugin-marketplaces.md
- https://code.claude.com/docs/en/admin-setup.md
