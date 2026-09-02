---
name: hello
description: fm-plugins マーケットプレイスからインストールしたプラグインが動作しているか確認する導入確認スキル。「/hello:hello」「プラグイン動作確認」「fm-plugins の導入確認」「hello プラグインを試す」で起動
allowed-tools: Bash(cat:*), Bash(true:*)
---

> 以下の `!`...`` はコンテキスト注入。`${CLAUDE_PLUGIN_ROOT}` はこのプラグインのインストール先ディレクトリに解決される。

## プラグイン情報（自動取得）

!`cat "${CLAUDE_PLUGIN_ROOT}/.claude-plugin/plugin.json" || true`

## 手順

1. 上の「プラグイン情報（自動取得）」に差し込まれた plugin.json から `name` と `version` を読み取る。追加のツール呼び出しは行わない。
2. 次の形式で日本語で報告する:

   > fm-plugins マーケットプレイスからインストールされた **<name> プラグイン v<version>** が動作しています。
   >
   > 更新するには次を実行してください:
   > ```
   > claude plugin marketplace update fm-plugins
   > claude plugin update hello@fm-plugins
   > ```

3. 「プラグイン情報（自動取得）」が空、または JSON として読めない場合は、次のように報告する:

   > プラグインルート（CLAUDE_PLUGIN_ROOT）が解決できず、plugin.json を読めませんでした。`claude plugin list` でインストール状態を確認し、必要なら `claude plugin install hello@fm-plugins` を再実行してください。
