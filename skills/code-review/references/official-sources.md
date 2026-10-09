# 公式仕様と公開実装

確認日: 2026-10-09。
このリポジトリの code-review と simplify は公開仕様を基に新しく記述した実装である。
公開プロンプトのコピーや翻訳、非公開の内部プロンプトの抽出は行っていない。
新しい指示はこのリポジトリの MIT License に従う。

## Bundled skills

正本は [Commands](https://code.claude.com/docs/en/commands)、[Review a diff locally](https://code.claude.com/docs/en/code-review#review-a-diff-locally)、[Skills](https://code.claude.com/docs/en/skills) である。
文書には commit ID がないため、確認日を版の基準とする。
公開 [claude-code リポジトリ](https://github.com/anthropics/claude-code/tree/e47cc82bdbd27b5f799acd5b05fd0c29d33bb48c) は commit `e47cc82bdbd27b5f799acd5b05fd0c29d33bb48c`、CHANGELOG 先頭は `2.1.295` だった。
この公開リポジトリでは bundled skill の内部プロンプトを確認できず、そのライセンスを推定して流用しない。

| 項目 | 公式公開仕様 | この実装 |
| --- | --- | --- |
| code-review 対象 | upstream より先のコミットと未コミット変更、PR、branch、path、ref range | 同じ対象を明示してローカルでレビュー |
| code-review 観点 | 正しさ。model と effort に応じて cleanup も扱う | 正しさと具体的な規則違反を5観点で確認。整理は simplify |
| code-review 実行 | model と effort に依存。background review、ultra cloud | 利用可能な並列担当と再検証。使えなければ順次確認を明示 |
| code-review --fix | 現在も正式対応 | 指摘の修正と最終コードの検証。commit は呼び出し元が明示依頼した場合だけ |
| simplify | 再利用、簡素化、効率、抽象度の4並列担当。cleanup を適用。バグ探しはしない | 同じ4観点で整理と検証。振る舞いを維持 |
| 投稿 | code-review --comment、ultra --post | 未対応。通常レビューで自動投稿しない |
| effort と上限 | low〜ultra、--max-findings、設定の再利用 | 未対応。引数を黙って読み替えない |
| ホスト統合 | ReportFindings と修正状態の更新 | 通常のテキスト報告 |

`simplify` が正式名で、`simplyfy` の別名スキルは追加しない。
以前の simplify と code-review の関係は版によって変わっているため、現在の文書を基準とする。
ローカルスキルは Claude Code の同名 bundled command を置き換える。
公式 `/review` の別名はこのローカル実装を呼ばない。

## 公開 plugin との違い

[claude-plugins-official](https://github.com/anthropics/claude-plugins-official/tree/315c4e48967d9541c29c3c656441dded353ca7aa) の commit は `315c4e48967d9541c29c3c656441dded353ca7aa`。
[code-review command](https://github.com/anthropics/claude-plugins-official/blob/315c4e48967d9541c29c3c656441dded353ca7aa/plugins/code-review/commands/code-review.md) は PR 向けの別実装である。
実行指示は5並列担当（CLAUDE.md、明白なバグ、履歴、過去PR、コメント）と候補ごとの独立評価を指定し、0〜100 の確信度で80未満を除外して PR に投稿する。
closed、draft、単純、自分がレビュー済みのPRを除外し、投稿前に再確認する。
同じ commit の README は4担当と記載しているため、公開実装の担当数は command を基準に確認した。
この実装は5観点と証拠による判定を参考にするが、投稿と自動除外は採用しない。
明示的なローカルレビューは draft 等でも使えるようにするためである。

[code-simplifier agent](https://github.com/anthropics/claude-plugins-official/blob/315c4e48967d9541c29c3c656441dded353ca7aa/plugins/code-simplifier/agents/code-simplifier.md) は `code-simplifier` plugin `1.0.0` の公開エージェントで、bundled `/simplify` とは別物である。
最近変更した範囲、振る舞いの維持、読みやすさを優先する指示を確認した。
特定言語のスタイル指定と自主的な修正開始は、この汎用スキルへ持ち込まない。

公開 plugin の各 [code-review/LICENSE](https://github.com/anthropics/claude-plugins-official/blob/315c4e48967d9541c29c3c656441dded353ca7aa/plugins/code-review/LICENSE) と [code-simplifier/LICENSE](https://github.com/anthropics/claude-plugins-official/blob/315c4e48967d9541c29c3c656441dded353ca7aa/plugins/code-simplifier/LICENSE) は Apache License 2.0 である。
コピーや改変を配布する場合は同ライセンスの配布条件に従う必要がある。
この実装ではそのソースを同梱せず、機能と観点を根拠に独自に記述した。
