---
description: spec.mdに従って分析コードを実装し、進捗をspec.mdに反映する
allowed-tools: Read, Write, Edit, Glob, Bash
---

# Phase 2: 分析コードの実装

## 入力

現在の `dev/spec.md` を読み込む：

@dev/spec.md

## タスク

上記の `dev/spec.md` の未完了ステップ（`[ ]`）を実装し、完了したら `[x]` に更新する。

### コーディング規約（CLAUDE.md より）

- 不要な `print` 文を入れない
- コメントアウトしたコードを残さない
- **1セルは短く、1つの目的に絞る**
- DataFrame は `df_` プレフィックスをつける（例: `df_train`, `df_result`）
- 同じ変数の編集は1セルの中で完結させる
- Markdown セルでセクションを区切り、日本語で記述する
- 上から順に実行して再現できる状態を保つ

### 実装ツール

- `uv` で環境管理（`pyproject.toml` / `uv.lock` が設定済み）
- `context7` でライブラリのドキュメントを参照する
- `code-simplifier` で冗長なコードを避ける

## 手順

1. `dev/spec.md` の未完了ステップを確認する
2. 最初の未完了ステップから実装を開始し、ユーザーに確認しながら進める
3. 各ステップを実装したら：
   - `dev/spec.md` の対応する `[ ]` を `[x]` に更新する
   - 次のステップへ進んでよいかユーザーに確認する
4. すべてのステップが完了したら Phase 2 終了を報告する

完了後: `/analyze-article` で次フェーズへ
