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

公式 bundled skill の公開仕様は次のとおりである。

- code-review は upstream より先のコミットと未コミット変更、PR、ブランチ、パス、ref range を対象とする。
  正しさを確認し、model と effort に応じて cleanup も扱う。
- --fix は作業ツリーに修正を適用する。
  --comment は投稿、ultra はクラウドレビュー、--post はクラウドレビューの投稿指定を扱う。
  effort、--max-findings、ReportFindings などのホスト統合もある。
- simplify は再利用、簡素化、効率、抽象度の4並列担当で cleanup を確認して適用する。
  正しさのバグ探しは code-review の対象である。

code-review の5観点と確信度判定は下記の公開 plugin を参考にしており、bundled skill の内部判定を再現したものではない。

ローカルスキルは Claude Code の同名 bundled command を置き換える。
公式 `/review` の別名はこのローカル実装を呼ばない。

## 公開 plugin との違い

[claude-plugins-official](https://github.com/anthropics/claude-plugins-official/tree/315c4e48967d9541c29c3c656441dded353ca7aa) の commit は `315c4e48967d9541c29c3c656441dded353ca7aa`。
[code-review command](https://github.com/anthropics/claude-plugins-official/blob/315c4e48967d9541c29c3c656441dded353ca7aa/plugins/code-review/commands/code-review.md) は PR 向けの別実装である。
実行指示は5並列担当（CLAUDE.md、明白なバグ、履歴、過去PR、コメント）と候補ごとの独立評価を指定し、0〜100 の確信度で80未満を除外して PR に投稿する。
closed、draft、単純、自分がレビュー済みのPRを除外し、投稿前に再確認する。
同じ commit の README は4担当と記載しているため、公開実装の担当数は command を基準に確認した。

[code-simplifier agent](https://github.com/anthropics/claude-plugins-official/blob/315c4e48967d9541c29c3c656441dded353ca7aa/plugins/code-simplifier/agents/code-simplifier.md) は `code-simplifier` plugin `1.0.0` の公開エージェントで、bundled `/simplify` とは別物である。
最近変更した範囲、振る舞いの維持、読みやすさを優先する指示を確認した。

公開 plugin の各 [code-review/LICENSE](https://github.com/anthropics/claude-plugins-official/blob/315c4e48967d9541c29c3c656441dded353ca7aa/plugins/code-review/LICENSE) と [code-simplifier/LICENSE](https://github.com/anthropics/claude-plugins-official/blob/315c4e48967d9541c29c3c656441dded353ca7aa/plugins/code-simplifier/LICENSE) は Apache License 2.0 である。
コピーや改変を配布する場合は同ライセンスの配布条件に従う必要がある。
