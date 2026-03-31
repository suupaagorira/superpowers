# Copilot CLI 適応 実装計画

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** GitHub Copilot CLI 向けに Superpowers を適応しつつ、既存の二段レビュー品質を維持する。

**Architecture:** plugin は薄く保ち、既存の `skills/` と `agents/` を正本として流用する。Git 運用のみを `wip` / `pr` / `.ai-notes` へ差し替え、reviewer の入力としては `wip` のローカル一時コミット差分を使う。

**Tech Stack:** Markdown skill files, agent prompt templates, Git worktrees / branches, GitHub Copilot CLI plugin manifest and hooks

**Spec:** `docs/superpowers/specs/2026-03-31-copilot-cli-adaptation-design.md`

---

## ファイル構成

| ファイル | 役割 | 変更内容 |
|---|---|---|
| `skills/finishing-a-development-branch/SKILL.md` | 完了フロー | `wip -> clean pr` と `push 明示指示のみ` へ変更 |
| `skills/using-git-worktrees/SKILL.md` | 作業開始 | `wip` 命名、保護ブランチ回避、`.ai-notes` 初期化 |
| `skills/subagent-driven-development/implementer-prompt.md` | implementer 規律 | 1 枚メモ更新と `wip` 一時コミット運用 |
| `skills/subagent-driven-development/SKILL.md` | 実行フロー | `Committed` 前提を `wip` 一時コミット前提へ変更 |
| `skills/writing-plans/SKILL.md` | 計画書 | 「頻繁なコミット」を論点単位コミットへ置換 |
| `skills/executing-plans/SKILL.md` | inline 実行 | finish 誘導文言の調整 |
| `hooks/session-start` | bootstrap hook | Copilot CLI 対応余地の確認と必要差分の追加 |
| `.claude-plugin/plugin.json` | plugin manifest | Copilot CLI 読み込みで不足があれば component path を明記 |

---

### Task 1: `finishing-a-development-branch` を新しい finish フローへ適応する

**Files:**
- Modify: `skills/finishing-a-development-branch/SKILL.md`

- [ ] **Step 1: finish skill の現行構造を読み直す**

`skills/finishing-a-development-branch/SKILL.md` を読み、現行の 4-option menu、cleanup、merge / push / PR 前提の文言を洗い出す。

- [ ] **Step 2: 新しい finish 方針を反映する**

次を明記する。

- `wip/*` はローカル作業枝であり、公開対象ではない
- `pr/*` は base から clean に切る
- `pr/*` には `.ai-notes` を持ち込まない
- `push` は `pushして` などの直接表現があるときだけ許可
- `pr/*` のコミットは論点単位で 2〜4 個、日本語、`チケット番号 + 要約`

- [ ] **Step 3: cleanup ルールを明記する**

次を明記する。

- `wip/*` と `.ai-notes` は `push` または `merge` 完了までローカル保持
- `pr/*` 準備後に即削除しない
- cleanup は AI が勝手に断行せず、finish フロー内で扱う

- [ ] **Step 4: 検証**

ファイルを再読し、古い `merge/push/PR` 前提が新方針と矛盾していないことを確認する。

---

### Task 2: `using-git-worktrees` に `wip` / `.ai-notes` の開始規則を入れる

**Files:**
- Modify: `skills/using-git-worktrees/SKILL.md`

- [ ] **Step 1: 開始時ルールを定義する**

次を追加する。

- 作業開始時は保護対象ブランチ (`main`, `master`, `develop`, `release/*`) の上で直接作業しない
- `wip/<ticket>　<日本語要約>` を基本とする
- 相性問題が出た場合のみ英数字スラッグへ自動フォールバック可

- [ ] **Step 2: 1 枚メモの初期化を定義する**

`.ai-notes/wip_<ticket>　<日本語要約>.md` を `wip` ごとに 1 ファイル用意する旨を追加する。見出しは以下を初期値とする。

```markdown
# 作業メモ

## 目的・依頼内容
## 実施方針
## 判断理由
## 変更内容
## 変更ファイル
## 確認結果
## 見送り事項
## 未解決事項
## 最終報告用要約
```

- [ ] **Step 3: 検証**

既存の worktree 安全性、baseline test、directory selection を壊していないことを確認する。

---

### Task 3: implementer prompt を `wip` 一時コミット + 1 枚メモ前提へ置換する

**Files:**
- Modify: `skills/subagent-driven-development/implementer-prompt.md`

- [ ] **Step 1: `Commit your work` を置換する**

`Commit your work` をそのまま要求するのではなく、次の順へ変える。

1. 作業中は必要に応じて 1 枚メモを更新する
2. review-ready になったら 1 枚メモを最新化する
3. `wip/*` に日本語の一時コミットを作る
4. その差分を reviewer に渡す

- [ ] **Step 2: report format を調整する**

report に次を足す。

- 更新したメモの要点
- 一時コミットの要約
- `review-ready` と判断した理由

- [ ] **Step 3: 検証**

`Ask them now`, `Don't guess`, `Self-review`, `BLOCKED` の思想が残っていることを確認する。

---

### Task 4: `subagent-driven-development` の commit 前提を `wip` 一時コミットへ寄せる

**Files:**
- Modify: `skills/subagent-driven-development/SKILL.md`

- [ ] **Step 1: 図と本文の `Committed` 前提を洗い出す**

`Committed`, `Get git SHAs`, `implements, tests, commits, self-reviews` などの記述を確認する。

- [ ] **Step 2: review 境界を新方針へ置換する**

次へ置換する。

- implementer は `review-ready` になったら `wip` 一時コミットを切る
- spec reviewer / code quality reviewer はその一時コミット差分を前提にする
- final finish は `wip -> clean pr` へ渡す

- [ ] **Step 3: 検証**

二段レビュー順序、review ループ、TodoWrite 進行は維持されていることを確認する。

---

### Task 5: `writing-plans` の Git 指示を新運用へ置換する

**Files:**
- Modify: `skills/writing-plans/SKILL.md`

- [ ] **Step 1: `frequent commits` と `Commit` step を置換する**

「頻繁なコミット」をそのまま要求せず、次へ置換する。

- 検証は細かく
- `wip` では review-ready 境界で一時コミット
- `pr` では論点単位 2〜4 個の日本語コミット

- [ ] **Step 2: サンプル commit message を日本語へ寄せる**

例を `チケット番号 + 要約` 形式に変える。

- [ ] **Step 3: 検証**

TDD、曖昧語禁止、細粒度手順が維持されていることを確認する。

---

### Task 6: `executing-plans` と plugin 入口の整合性を取る

**Files:**
- Modify: `skills/executing-plans/SKILL.md`
- Modify if needed: `hooks/session-start`
- Modify if needed: `.claude-plugin/plugin.json`

- [ ] **Step 1: `executing-plans` の finish 誘導を新方針へ合わせる**

完了後の finish skill 呼び出しが `wip -> clean pr` 方針と矛盾しないようにする。

- [ ] **Step 2: Copilot CLI plugin 入口の不足を確認する**

次を確認する。

- `.claude-plugin/plugin.json` で Copilot CLI が必要 component を拾えるか
- `hooks/session-start` が Copilot CLI で使えるか、あるいは使えない前提で docs 側へ寄せるべきか

- [ ] **Step 3: no-regret な差分だけ入れる**

公式 docs と現行 issue に照らして安全な範囲のみ変更する。plugin bootstrap が未解決なら、無理に workaround を埋め込まない。

- [ ] **Step 4: 検証**

新しい skill 文言と plugin 入口の整合性を確認し、過大な platform 依存を追加していないことを確認する。

---

## 完了条件

- `wip` / `pr` / `.ai-notes` / 日本語運用が skill 文書へ反映されている
- 二段レビュー構造は維持されている
- `wip` の一時コミットが reviewer 入力として残っている
- `pr` に `.ai-notes` を持ち込まない方針が明記されている
- Copilot CLI plugin 入口について、現時点で安全な範囲の対応方針が明文化されている
