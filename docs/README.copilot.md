# Superpowers for GitHub Copilot CLI

GitHub Copilot CLI 向けに Superpowers を導入するためのガイドです。

この版は `仮完成版` です。plugin install 自体は成立していますが、Copilot CLI 側の bootstrap や custom agent の公開方法には、現時点でまだ制約があります。

## この版でできること

- Copilot CLI plugin として Superpowers を導入する
- skill 群を plugin として配布する
- `wip/*`、`.ai-notes`、`clean な pr/*` を前提にした workflow を使う
- 日本語のメモ、報告、コミットメッセージ運用へ寄せる

## この版の位置づけ

この版で最優先しているのは、従来の Superpowers の品質を落とさないことです。

そのため、次の方針で組んでいます。

- `skills` と `agents` を本体として維持する
- plugin は薄い adapter に留める
- `spec compliance -> code quality` の二段レビューは維持する
- Git 運用だけを職場向けに調整する

## 導入方法

最短手順は `.copilot/INSTALL.md` を参照してください。

- [導入手順](../.copilot/INSTALL.md)

### 既存ユーザー向けの前提

すでに他の harness で Superpowers を使っている人でも、この仮完成版では **既存の公式 superpowers をいったん外し、この版を入れ直す** のを基本とします。

この版では、次の考え方を推奨します。

- まず `copilot plugin uninstall superpowers` を行う
- その後、この仮完成版の clone で `copilot plugin install .` を行う
- 既存 clone を再利用する場合でも、`update` より `uninstall -> install` を優先する

特に、複数の clone を持っている人は「どの clone の内容が Copilot CLI に反映されているか」が分かりにくくなりやすいので注意してください。

## 現在の運用前提

### 1. `wip/*` をローカル作業枝として使う

- 途中の試行錯誤や reviewer 用の差分境界は `wip/*` に積む
- 一時コミットはローカル用途
- 公開用履歴にはそのまま持ち込まない

### 2. `.ai-notes/` で 1 枚メモを持つ

- `wip/*` ごとに 1 ファイル
- 圧縮に強い文脈保持
- 最終報告用要約を含む
- `pr/*` には持ち込まない

### 3. `pr/*` は clean に作る

- base から clean に切る
- コードだけを移す
- 公開履歴は 2〜4 個の論点単位コミットへ整理する
- コミットメッセージは日本語

### 4. `push` は明示指示があるまでしない

この版では、予期しない公開を避けるため、`push` は明示指示がある場合だけに寄せています。

## 現時点で確認できていること

- `copilot --version` は通る
- `copilot plugin install obra/superpowers` は通る
- `copilot plugin install .` でも local plugin install が通る

## 現時点の制約

### 1. skill の自動起動精度

plugin install 自体は成功しても、Copilot CLI の startup bootstrap 制約により、skill の自動起動精度は環境差が出る可能性があります。

そのため、現時点では skill を明示的に含めた依頼が最も安定します。

例:

```text
Use the brainstorming skill to design this feature.
```

```text
Use writing-plans to create the implementation plan.
```

### 2. plugin 同梱 agent の直接選択

plugin に同梱した agent file 自体は install されますが、Copilot CLI 1.0.14 では `--agent` から直接選べないケースを確認しています。

そのため、現時点の fallback は次です。

- role 分離が必要なときは agent profile を読む
- その役割を宣言して同一 session で続行する

### 3. 既存 instructions との競合

すでに `~/.copilot` やプロジェクト側で custom instructions を使っている場合、そちらが Superpowers の workflow より強く効くことがあります。

その場合は:

- まず skill 名を明示して依頼する
- 強い既存 instruction があるなら、競合部分を見直す
- どちらのルールを優先したいかを明確にする

## 期待する使い方

### planning

- `brainstorming`
- `writing-plans`

### implementation

- `subagent-driven-development`
- `executing-plans`
- `test-driven-development`

### quality

- `requesting-code-review`
- `verification-before-completion`
- `systematic-debugging`

## 推奨プロンプト例

```text
Use the brainstorming skill to refine this feature before implementation.
```

```text
Use writing-plans to create a detailed implementation plan for this task.
```

```text
Use systematic-debugging before proposing any fix.
```

```text
Use subagent-driven-development to execute this plan.
```

## どこまでが安定していて、どこからが未解決か

### 安定している部分

- plugin install
- local plugin packaging
- skill / agent ファイルの同梱
- `wip/*` / `.ai-notes` / `pr/*` 前提の skill 文書

### 未解決の部分

- startup bootstrap の完全保証
- plugin 同梱 agent の direct invocation
- Copilot CLI 側の skill / agent 公開仕様差

## 更新方針

この版は、まず「別環境へ導入できること」と「既存 Superpowers の品質思想を壊さないこと」を優先して固めています。

次段階では以下を詰める想定です。

- Copilot CLI 上での bootstrap 安定化
- custom agent の公開方法の確定
- 実運用テストを通した wording の微修正
