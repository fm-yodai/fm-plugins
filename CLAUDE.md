# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## このリポジトリについて

株式会社FREEMIND 社内向けの Claude Code プラグインマーケットプレイス `fm-plugins`。GitHub の private リポジトリ `koo-inc/fm-claude-plugins` がそのまま配布元で、利用者は次のコマンドで導入する。

```bash
claude plugin marketplace add koo-inc/fm-claude-plugins
claude plugin install <name>@fm-plugins
```

Claude Code は利用者自身の git 認証（SSH 鍵 / credential helper）でこのリポジトリを clone するため、利用者には `koo-inc` 配下 private リポジトリの Read 権限が必要。

実行コードはなく、JSON マニフェスト 2 種と Markdown スキルだけで構成される。

- 利用者向けの導入・更新・プラグイン追加手順: `README.md`
- 設計の背景と段階的拡張計画（案 A → B → C）: `docs/superpowers/specs/`
- 実装手順書: `docs/superpowers/plans/`

## コマンド

テストフレームワークやビルドはない。検証は次の 2 段階がすべて。

### 1. マニフェスト検証

変更後は必ず両方を実行し、**警告なしの `✔ Validation passed`** を確認する（`--strict` は警告もエラー扱いにする）。

```bash
claude plugin validate --strict .                 # マーケットプレイス定義
claude plugin validate --strict plugins/<name>    # 個別プラグイン
```

### 2. 作業コピーからのインストール確認（スモークテスト）

push 前に、作業ディレクトリ自体をマーケットプレイスとして登録して動作を確かめる。マーケットプレイス名は marketplace.json の `name`（`fm-plugins`）で決まるため、GitHub 版と作業コピーは同時に登録できない。切り替えは remove → add で行う。

```bash
claude plugin marketplace remove fm-plugins          # GitHub 版が登録済みなら外す
claude plugin marketplace add "$PWD"                 # リポジトリ直下で実行
claude plugin install <name>@fm-plugins
claude plugin details <name>@fm-plugins              # skills にコンポーネントが載るか
```

新しいセッションで `/<plugin>:<skill>` を実行して出力を確認する。確認が済んだら GitHub 版に戻す。

```bash
claude plugin uninstall <name>@fm-plugins
claude plugin marketplace remove fm-plugins
claude plugin marketplace add koo-inc/fm-claude-plugins
claude plugin install <name>@fm-plugins
```

### 3. リリース（利用者に届ける）

タグや CI はまだない（案 B 以降）。**`main` の最新コミットがそのまま配布物**になるので、`main` へ直接コミットせずブランチ → PR → マージで進める。

1. 変更したプラグインの `plugin.json` の `version` を上げる
2. 検証 1・2 を通す
3. PR を作り `main` にマージする
4. 利用者側は `claude plugin marketplace update fm-plugins`（`main` を取得）→ `claude plugin update <name>@fm-plugins`（`version` が上がっていれば更新）→ 再起動または `/reload-plugins`

## 構造

- リポジトリ直下の `.claude-plugin/marketplace.json` がマーケットプレイス本体。`plugins` 配列の各エントリが `"source": "./plugins/<name>"` で同一リポジトリ内のプラグインを相対パス参照する
- `plugins/<name>/` が 1 プラグイン。`.claude-plugin/plugin.json`（`name`・`version` 必須）と `skills/<skill>/SKILL.md` を持ち、スキルは `/<plugin>:<skill>` で呼ばれる。新規プラグインは `plugins/hello/` をコピーして作るのが早い
- **バージョンの正は各 `plugin.json` の `version`**。marketplace.json 側ではピンしない。利用者の `claude plugin update` はこの値で更新要否を判断するため、プラグインの中身を変えたら必ず `version` を上げる

### SKILL.md の書き方（`plugins/hello/skills/hello/SKILL.md` を手本にする）

- `` !`...` `` はコンテキスト注入。スキル起動時にシェルで実行され、結果が本文に差し込まれる。コマンドの先頭は frontmatter の `allowed-tools`（例: `Bash(cat:*)`）に一致させ、末尾を `|| true` にして終了コードを 0 に揃える（0 以外だと注入が中断される）
- `${CLAUDE_PLUGIN_ROOT}` はインストール先のプラグインディレクトリに解決される。自分の `plugin.json` を読むなどに使う
- `description` には `/<plugin>:<skill>` に加えて自然言語の起動トリガー（「プラグイン動作確認」など）も列挙する

## 守るべき制約

管理コンソールからの組織配布（案 C）の条件を今から満たしておく:

- プラグインは必ずこのリポジトリ内 `plugins/<name>/` に置き、相対パスで参照する。外部リポジトリや URL 参照にしない
- トップレベルに `bin/` を置かない
- `plugin.json` の `repository` は `https://github.com/koo-inc/fm-claude-plugins` に揃える

スコープと表記:

- hooks / MCP / agents / commands / CI / リリースタグは現時点では作らない（案 B 以降）。追加するときは先に `docs/superpowers/specs/` を更新する
- 利用者向け文書（README、スキル本文、スキルの出力）は日本語。識別子・マニフェストのキーは英語
- コミットメッセージは `feat:` / `docs:` など Conventional Commits の接頭辞を付ける
