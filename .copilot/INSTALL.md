# GitHub Copilot CLI 向け Superpowers 導入手順

この手順は、現在の作業内容を `仮完成版` として別環境へ導入するためのものです。

対象:

- GitHub Copilot CLI
- Superpowers を plugin として導入したい環境

前提:

- GitHub Copilot CLI がインストール済み
- `copilot --version` が通る
- Git が使える

## 推奨手順

この仮完成版では、**既存の公式 superpowers をいったん削除し、この版を入れ直す** 方式を推奨します。

理由:

- どの clone の内容が反映されているか分かりやすい
- 公式版と仮完成版が混ざらない
- 動作確認時の切り分けがしやすい

### 1. 既存の `superpowers` plugin を削除する

```bash
copilot plugin uninstall superpowers
```

`superpowers` が未導入なら、そのまま次へ進んでかまいません。

### 2. リポジトリを取得する

```bash
git clone <このリポジトリのURL> superpowers
cd superpowers
```

### 既に Superpowers を使っている人へ

すでに別用途で Superpowers の clone を持っている場合は、**新しく clone し直す必要はありません**。

使い方は次のいずれかです。

#### 既存 clone をそのまま使う

例:

```bash
copilot plugin uninstall superpowers
cd <既存の superpowers clone>
copilot plugin install .
```

この方法は、Claude / Cursor / Codex / OpenCode / Gemini などで使っている既存 clone を再利用したい場合に向いています。

#### 既に Copilot CLI に `superpowers` を入れている

この仮完成版では、**update より uninstall -> install を優先**してください。

```bash
copilot plugin uninstall superpowers
cd <使いたい superpowers clone>
copilot plugin install .
```

その後に確認します。

```bash
copilot plugin list
```

### 3. plugin としてインストールする

```bash
copilot plugin install .
```

成功すると、次のようなメッセージが出ます。

```text
Plugin "superpowers" installed successfully. Installed 14 skills.
```

### 4. インストール確認

```bash
copilot plugin list
```

`superpowers` が表示されれば導入成功です。

## 推奨確認

### バージョン確認

```bash
copilot --version
```

### plugin 一覧確認

```bash
copilot plugin list
```

### Superpowers が入っているかの簡易確認

```bash
copilot -p "Do you have superpowers? Answer briefly." --allow-all-tools --no-ask-user
```

## 更新手順

この仮完成版では、更新時も `置き換え方式` を推奨します。

```bash
copilot plugin uninstall superpowers
git pull
copilot plugin install .
```

### 既存 clone を使っている場合の更新

```bash
copilot plugin uninstall superpowers
cd <その clone>
git pull
copilot plugin install .
```

## アンインストール

```bash
copilot plugin uninstall superpowers
```

## 現時点の注意点

この仮完成版には、次の特徴があります。

- `skills` は plugin install で導入される
- `wip/*`、`.ai-notes`、`clean な pr/*` を前提にした運用へ寄せている
- コミットメッセージ、メモ、報告は日本語を前提としている

一方で、次の点はまだ制約があります。

- Copilot CLI 上で plugin 同梱 agent を `--agent` から直接選べるとは限らない
- startup 時の bootstrap は、CLI の仕様差で完全には保証できない場合がある

そのため、現時点では次の使い方を推奨します。

- workflow 本体は skill で使う
- role 分離が必要なときは、agent profile を読み、その役割で続行する

## 運用上の推奨

- 公開用の作業は `pr/*` で行う
- ローカル試行錯誤は `wip/*` に閉じ込める
- `.ai-notes/` は `wip/*` 専用とし、`pr/*` に持ち込まない
- `push` は明示指示があるまで行わない

## トラブルシューティング

### `copilot plugin install .` が失敗する

次を確認してください。

1. `plugin.json` がリポジトリ直下にあるか
2. `copilot --version` が通るか
3. リポジトリ直下でコマンドを実行しているか

### 既に別の `superpowers` が入っていて挙動が混ざる

次を確認してください。

```bash
copilot plugin list
```

`superpowers` の反映元が曖昧だと、どの clone の内容を見ているか分かりにくくなります。迷ったら一度削除して、使いたい clone から入れ直してください。

```bash
copilot plugin uninstall superpowers
cd <使いたい superpowers clone>
copilot plugin install .
```

### 既存の custom instructions が強く効いている

すでに `~/.copilot` やリポジトリ側で custom instructions を使っている場合、Superpowers の skill 起動よりそちらが強く出ることがあります。

この仮完成版では、少なくとも初期確認時は次を推奨します。

- skill 名を明示して依頼する
- 既存 instructions と競合しそうな強いルールがある場合は一時的に弱める

### `superpowers` が一覧に出ない

```bash
copilot plugin list
```

で空になる場合は、削除して再導入してください。

```bash
copilot plugin uninstall superpowers
cd <この仮完成版の clone>
copilot plugin install .
```

### skill が期待通りに起動しない

この仮完成版では、plugin install 自体は通りますが、Copilot CLI 側の bootstrap 制約により、skill の自動起動精度は環境差が出る可能性があります。

その場合は、まず明示的に skill 名を含めて依頼してください。

例:

```text
Use the writing-plans skill to create an implementation plan.
```

```text
Use systematic-debugging before proposing a fix.
```

## 補足

詳細な背景や現時点の制約は `docs/README.copilot.md` を参照してください。
