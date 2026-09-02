# hello

`fm-plugins` マーケットプレイスの導入確認用サンプルプラグインです。インストール後に `/hello:hello` を実行すると、プラグイン名とバージョン、更新コマンドを表示します。

## 使い方

```
/hello:hello
```

自然言語でも起動します（「プラグイン動作確認」「fm-plugins の導入確認」）。

## 構成

```
hello/
├── .claude-plugin/plugin.json   name / version
└── skills/hello/SKILL.md        導入確認スキル
```

## 変更するとき

1. `skills/hello/SKILL.md` を編集する
2. `.claude-plugin/plugin.json` の `version` を上げる（例: 0.1.0 → 0.1.1）
3. リポジトリ直下で `claude plugin validate --strict plugins/hello` を実行して `Validation passed` を確認する
