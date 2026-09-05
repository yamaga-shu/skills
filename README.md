# skills

複数のリポジトリで共通して使う Claude Code のスキル集。
各スキルは `skills/<name>/SKILL.md` に置く。

| スキル | 呼び出し | 用途 |
| --- | --- | --- |
| [grill-to-issue](skills/grill-to-issue/SKILL.md) | ユーザーのみ | 既存の issue と PR を一覧で確認したうえで、起票したい issue のスコープと完了条件をインタビューで固め、issue を立てる。設計を詰めることは grill-with-docs に委ねる |
| [grill-with-docs](skills/grill-with-docs/SKILL.md) | ユーザーのみ | issue 番号を受け取り、その issue の計画や設計を徹底的にインタビューし、用語集と ADR を書きながら決定を issue に記録する |
| [plan-from-issue](skills/plan-from-issue/SKILL.md) | ユーザーのみ | issue 番号を受け取り、本文、コメント、コードを読んで、git-convention のコミット単位に沿った実装計画を立て、承認後に issue のコメントに残す。計画を左右する不明点だけを確認する |
| [implement-from-issue](skills/implement-from-issue/SKILL.md) | モデルとユーザー | issue の「実装計画」コメントに従ってブランチを切り、計画のコミット単位で実装し、PR を出す。plan-from-issue の直後なら同じセッションで計画を読み直さずに続ける |
| [grilling](skills/grilling/SKILL.md) | モデルとユーザー | 設計ツリーをラウンド単位で質問するインタビューの手順 |
| [domain-modeling](skills/domain-modeling/SKILL.md) | モデルとユーザー | 用語集（CONTEXT.md）と ADR を書く規律 |
| [doc-convention](skills/doc-convention/SKILL.md) | モデルとユーザー | README と AGENTS.md の分担、同じ事実を複数箇所に書かない、コメントを書く基準 |
| [git-convention](skills/git-convention/SKILL.md) | モデルとユーザー | ブランチ名とコミットメッセージの規約 |
| [github-convention](skills/github-convention/SKILL.md) | モデルとユーザー | issue と PR の書き方、gh コマンドでの本文の渡し方 |

`grill-to-issue` で issue を立て、`grill-with-docs` でその issue の設計を詰め、
`plan-from-issue` で実装計画を立て、`implement-from-issue` で計画どおりに実装して PR を出す、
という流れを想定している。
`plan-from-issue` は計画を承認したセッションのまま `implement-from-issue` に進めるので、
計画と実装を同じセッションで続けても、別のセッションに分けてもよい。

`grill-to-issue`、`grill-with-docs`、`plan-from-issue`、`implement-from-issue` は単体では動かない。
入口ごとに、次のスキルを一緒に入れる。
表は、その入口から Skill ツールで読み込まれうるスキルをすべて含めた閉じた集合である。

| 入口 | 一緒に入れるスキル |
| --- | --- |
| `grill-to-issue` | `grilling`、`github-convention`、`git-convention` |
| `grill-with-docs` | `grilling`、`doc-convention`、`domain-modeling`、`github-convention`、`git-convention` |
| `plan-from-issue` | `grilling`、`git-convention`、`github-convention`、`doc-convention`、`domain-modeling`、`implement-from-issue` |
| `implement-from-issue` | `git-convention`、`github-convention`、`doc-convention`、`domain-modeling` |

`grilling`、`domain-modeling`、`grill-with-docs` は [mattpocock/skills](https://github.com/mattpocock/skills) を
日本語に改訂し、`doc-convention` への依存を加えたもの（原著: MIT License, Copyright (c) 2026 Matt Pocock）。

## インストール

[skills CLI](https://skills.sh/) を使う。

```bash
# 現在のプロジェクトに全部入れる
npx skills add yamaga-shu/skills --skill '*'

# 全プロジェクトで使う（ユーザーレベル）
npx skills add yamaga-shu/skills --skill '*' -g

# 一部だけ入れる
npx skills add yamaga-shu/skills --skill doc-convention
```

CLI を使わない場合は、`skills/<name>` を `.claude/skills/<name>`（プロジェクト）
または `~/.claude/skills/<name>`（ユーザー）にコピーする。

## リポジトリ固有の設定

スキルは用語集や ADR の置き場所、ブランチ名の形式やコミットメッセージの言語に既定値を持つ。
リポジトリで変える場合は、ルートの AGENTS.md に書く。書き方は doc-convention の「正本の表」、
git-convention の「前提」を参照。既定ブランチは git-convention の手順で検出するので設定しない。

スキルは AGENTS.md を明示的に読むので、エージェントが AGENTS.md を自動で読み込まなくても上書きは効く。
ただし Claude Code が自動で読むのは CLAUDE.md なので、スキルを通さない普段の作業にも上書きを効かせたいなら、
CLAUDE.md に `@AGENTS.md` と書いて取り込む。
