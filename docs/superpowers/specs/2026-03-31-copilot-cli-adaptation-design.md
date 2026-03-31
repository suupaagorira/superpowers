# Copilot CLI 適応設計

GitHub Copilot CLI 向けに Superpowers を適応しつつ、既存の Superpowers 品質を落とさない。

## 目的

現在の Superpowers は、`skills/` と `agents/` に品質の本体があり、platform 固有層は薄い。この構造を維持したまま、GitHub Copilot CLI で実用的に動く形へ寄せる。

今回の最優先事項は次の 2 点とする。

1. `spec compliance -> code quality` の二段レビューを維持する
2. 既存の公開履歴前提ではなく、職場向けの `wip` / `pr` 運用へ適応する

## 前提

- 対応対象は GitHub Copilot CLI のみ
- VS Code は別管理とし、本設計では扱わない
- 配布形態は Copilot CLI plugin を前提にする
- plugin install 経路は `obra/superpowers` の既存 manifest 読み込みを前提にする
- メモ、資料、コミットメッセージは日本語に統一する

## 現状認識

品質を支えている本体は Git ではない。主に次の 3 つである。

- `writing-plans` の粒度と具体性
- `subagent-driven-development` の二段レビュー
- `implementer-prompt.md` の「質問先行」「推測禁止」「self-review」「BLOCKED」

したがって、Git 手順を変更しても、上記 3 点を維持できれば品質低下は抑えられる。

一方で、reviewer 側は安定した差分境界を必要とする。差分境界を完全にメモへ置き換えると、`BASE_SHA..HEAD_SHA` で見ていた品質検査が弱くなる。

## 設計方針

### 1. plugin は薄く、skills / agents を本体にする

Copilot CLI 向けでも、platform 固有実装を厚くしない。

- plugin は manifest / hook / install の役割に留める
- workflow の本体は既存の `skills/` と `agents/` を正本にする
- Copilot CLI 固有差分は最小限の文言変更に留める

### 2. `wip` と `pr` の責務を分離する

- `wip/*` はローカル作業枝
- `wip/*` では一時コミットを許可する
- `pr/*` は公開用に clean に切る
- `pr/*` にはコードだけを移し、`.ai-notes` は持ち込まない
- `push` は直接表現で指示されたときだけ許可する

### 3. 1 枚メモは人間向けの製造記録として使う

1 枚メモは `.ai-notes/` 配下に `wip` ブランチごと 1 ファイルで保持する。

- 例: `.ai-notes/wip_ABC-123　検索条件整理.md`
- `wip/*` で追跡ファイルとして扱う
- `pr/*` には存在させない
- 最終報告用要約は「そのまま提出可能な完成文」とする

1 枚メモの主用途は次の通り。

- コンテキスト圧縮耐性
- 製造担当者としての報告
- 判断理由、見送り事項、未解決事項の保持

ただし reviewer の正式入力は、引き続きローカル一時コミットの差分を使う。

### 4. reviewer 精度維持のため、`wip` の一時コミットは残す

`wip/*` は「一時コミットを積むためのブランチ」として扱う。

- task 完了または review-ready 境界で日本語の一時コミットを作る
- これらは `pr/*` にそのまま持ち込まない
- `pr/*` では論点単位で 2〜4 個の日本語コミットに再構成する

これにより、

- reviewer は従来どおり安定差分を見られる
- 公開履歴は職場向けに整理できる
- `.ai-notes` に依存しすぎず、Git の差分検査も活用できる

## Copilot CLI 固有の制約

GitHub Docs と `obra/superpowers` の issue から、Copilot CLI には次の制約がある。

- plugin directory から skill / agent は読める
- ただし plugin 側から session start bootstrap を強制注入する経路は弱い、または未整備
- 実測では、plugin に同梱した agent profile ファイルは install されても、Copilot CLI 1.0.14 の `--agent` からは直接選択できなかった

このため、`using-superpowers` 相当の「常に skill を先に確認する」強制は、platform 固有 bootstrap に過度に依存しないようにする。

対処方針:

- `using-superpowers` の説明力を維持する
- skill description と agent description を明確化して、自然起動の精度を上げる
- role-separated agent を要求する箇所では、custom agent が直接選べない場合に agent profile を読んで同じ役割で継続する fallback を明記する
- plugin だけで補えない常時ルールは、必要なら user/repo instructions に逃がせる設計にする

## 変更対象

### 優先度 1

- `skills/finishing-a-development-branch/SKILL.md`
- `skills/using-git-worktrees/SKILL.md`

ここで `wip` / `pr` / `.ai-notes` / `push は直接指示のみ` を定義する。

### 優先度 2

- `skills/subagent-driven-development/implementer-prompt.md`
- `skills/subagent-driven-development/SKILL.md`

ここで実装者の Git 作法と reviewer 入力の境界を定義する。

### 優先度 3

- `skills/writing-plans/SKILL.md`
- `skills/executing-plans/SKILL.md`

ここで「頻繁なコミット」前提を新しい運用へ置換する。

## 変更しないもの

- `spec compliance -> code quality` の順序
- reviewer の再レビュー・ループ
- implementer の質問先行、推測禁止、self-review、BLOCKED
- `writing-plans` の細粒度ステップ、TDD、曖昧語禁止

## 成功条件

この適応が成功したとみなす条件は次の通り。

1. Copilot CLI で `skills + agents` を使える
2. `wip/*` で一時コミットと 1 枚メモを併用できる
3. `pr/*` へ `.ai-notes` を持ち込まない
4. `push` が明示指示なしに行われない
5. 既存の二段レビュー構造が壊れていない

## 開いている論点

Copilot CLI の plugin だけで startup bootstrap を完全に担保できるかは不透明である。ここは今回の skill / agent 適応とは分離し、必要なら別途 plugin / instructions 層で扱う。
