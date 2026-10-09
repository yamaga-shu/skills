# 独立セッションの起動

実行順序、引き渡し情報、停止条件は [implement-from-issue](../SKILL.md#pr-前の独立セッション) に従う。
以下は 2026-10-09 に公式ドキュメントで確認した機構である。
具体的なフラグは導入済みの CLI の help でも確認する。

## Claude Code

[公式 CLI reference](https://code.claude.com/docs/en/cli-reference) の `claude -p` を、対象領域を cwd として新しいプロセスで実行する。
`--session-id` に各段階で新しい UUID を指定し、`--output-format json` の session ID と結果を保存できる。
プロンプトには実行するローカルスキルの絶対パスと引き渡し情報を含め、公式 bundled skill と名前が衝突してもこの実装を読むよう指定する。

`--continue`、`--resume`、`--fork-session` は使わない。
fork は会話履歴をコピーするため、この手順の独立した評価を満たさない。

## Codex

[公式 non-interactive mode](https://learn.chatgpt.com/docs/non-interactive-mode) の新しい `codex exec` を段階ごとに起動する。
`resume` を使わず、対象領域を cwd とする。
`--json` のイベントと最終結果を記録し、各実行の thread ID を保存する。
Codex の既定の read-only 設定では修正できないため、環境で許可済みの sandbox 設定を使う。
スキル探索に頼れない環境では、委任プロンプトでローカルの SKILL.md を絶対パスで指定する。

## CLI 以外の起動方法

アプリ等に新しいタスクを作る機能がある環境では、履歴を継承しない新規タスクを2つ順番に作ってもよい。
実行環境に新しい独立コンテキストの subagent がある場合も、会話を継承しない設定と別々のセッション識別子を確認したうえで使える。
継承の有無を確認できない場合は代替にせず、必要な新規セッションをユーザーへ案内する。

## 権限不足

通常の権限設定を使い、権限を迂回するフラグを足さない。
必要な編集や commit が許可されない場合は、具体的な操作を報告して止める。
