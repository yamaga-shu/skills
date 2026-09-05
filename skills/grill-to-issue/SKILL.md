---
name: grill-to-issue
description: 既存の issue・PR を踏まえて、起票したい issue の内容をインタビューで固め、GitHub に issue を立てる。設計の深掘りは起票後の grill-with-docs に委ねる。
argument-hint: "[テーマ]"
disable-model-invocation: true
---

引数: $ARGUMENTS

引数があれば、起票したい issue のテーマとして扱う。無ければ、一覧の確認を終えてから最初のラウンドで聞く。

開始前に Skill ツールを 2 回呼び、"grilling"、"github-convention" をこの順にそれぞれ読み込む。1 回の呼び出しに複数の名前を渡さない。

- grilling がインタビューの進め方を決める
- github-convention がタイトルと本文の書き方、gh コマンドでの本文の渡し方を決める

## grill-with-docs との分担

このスキルは「何を、どこまでやる issue か」を固めて起票するまでを担う。
ISSUE-FORMAT.md のうち埋めるのは、リード文、「スコープ」、「完了条件」、「注記」である。
「決定」「設計」「実装時に決めること」は、インタビューの中で自然に出てきた事柄だけを書き、埋めるための質問はしない。
設計の深掘りは起票後に `/grill-with-docs <番号>` で行う。同じ論点を二つのスキルで二度聞かない。

## 既存の issue・PR の確認

インタビューを始める前に、open の issue と PR を件数を絞って一覧で取り、以下の三つを把握する。本文は読まない。

```bash
gh issue list --state open --limit 50 --json number,title,labels,milestone,parent,subIssuesSummary,updatedAt
gh pr list --state open --limit 30 --json number,title,labels,updatedAt
gh label list --limit 50
```

- **重複・関連**: テーマに近い issue や PR があるか。親 issue の下にぶら下がるべきテーマかどうかも見る
- **ラベル・マイルストーンの慣習**: どのラベルがどう使われているか。起票時に付けるラベルの候補にする
- **タイトルの書き方**: 既存のタイトルの粒度と語調。新しい issue もそれに合わせる

テーマが分かった時点で、closed も含めて関連 issue を検索し直す。
一覧は open だけなので、過去に同じことを検討して閉じた issue はここで拾う。

```bash
gh issue list --state all --limit 20 --search "<テーマのキーワード>" --json number,title,state,updatedAt
```

関連しそうな issue が見つかったら、その issue だけ `gh issue view <番号> --json title,body` で本文を読む。

## 重複・分割の扱い

分割の判断と親子の結び方は github-convention の「Issue」節に従う。
行数の見積もりが必要なら、関係するファイルをサブエージェントに調べさせ、ユーザーには聞かない。

- 既存の open な issue がテーマを既に扱っているなら、新規に立てず、その issue の本文の改訂かコメントの追記を提案する。
  ユーザーが新規に立てると決めたら、本文の「注記」で既存 issue との違いを述べる。
- 既存の open な issue がテーマを含んでいるが、1 つの PR で閉じられない大きさなら、
  その issue を親にして、今回のテーマを sub-issue として立てることを提案する。
- テーマが既存の親 issue の一部なら、その親の sub-issue として立てる。
- 関連する closed な issue があれば、なぜ閉じたかを確認し、同じ理由で今回も不要にならないかをユーザーに問う。
- インタビューの途中で、テーマが 1 つの PR で閉じられない大きさだと分かったら、分割を提案する。
  テーマ全体を親 issue として先に立て、分割先をその sub-issue として立てる。
  親には分割の前に固めた「決定」と「含まない」を書き、各 sub-issue にはそれぞれのスコープと完了条件を書く。

## インタビュー

設計ツリーの根は「この issue を閉じたとき、何ができるようになっているか」である。
そこから「含む」「含まない」「完了条件」が枝分かれする。
最初のラウンドでは、既存 issue の一覧から読み取った関連や慣習を前提として提示し、テーマの確認から始める。

フロンティアが空になる条件は次の三つが揃うことである。

- リード文として「何を達成するか」を数行で言い切れる
- 「含む」「含まない」が列挙でき、「含まない」にはなぜ今やらないかの理由がある
- 「完了条件」が、達成したかどうかを外から判定できる形で書ける

「どう作るか」に踏み込む質問が出てきたら、それは grill-with-docs の領分である。
ユーザーの回答にすでに含まれていれば「決定」に書き、含まれていなければ深掘りせず「実装時に決めること」に残す。

## 起票

フロンティアが空になったら、タイトルと本文の全文を提示し、ユーザーの承認を得てから `gh issue create` を実行する。
承認前に issue を立てない。

- タイトルと本文は github-convention に従う。本文はヒアドキュメントで渡す
- ラベルは一覧から読み取った慣習に従い、`--label` で付ける。合うラベルが無ければ付けない
- sub-issue として立てるときは `--parent <親の番号>` を付ける。親を先に立て、その番号を使う
- リポジトリに `.github/ISSUE_TEMPLATE/` があれば、github-convention に従いその構成を優先する

起票したら issue の URL を示し、「決定」「設計」を詰める必要があるなら `/grill-with-docs <番号>` を案内する。
