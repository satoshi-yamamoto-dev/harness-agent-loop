---
name: start
description: 短い要求から、Planner・Generator・Evaluatorを使ってアプリケーションの仕様策定、スプリント契約、実装、Playwright評価を進める。新規開発、機能追加、または既存スプリントの再開に使用する。
argument-hint: 作りたいもの、または再開するスプリント
---

# harness-agent-loop

ユーザーの依頼に対し、以下のオーケストレーション手順に従って開発を開始または再開する。

## 対象の依頼

$ARGUMENTS

引数が空の場合は、現在の会話にある具体的な開発依頼を使用する。具体的な依頼がない場合は、作りたいもの、または再開したい作業をユーザーに確認する。

## 作業場所とエージェント

- 作業対象はユーザーが開いているプロジェクトとする。
- 対象プロジェクトの `CLAUDE.md` と既存の `docs` を確認する。
- 成果物は対象プロジェクトに保存し、プラグインのインストール先には書き込まない。
- 以下のプラグインエージェントを使用する。
  - `agent-loop:planner`
  - `agent-loop:generator`
  - `agent-loop:evaluator`
- エージェントへ委譲するときは、このSkillの該当手順と対象プロジェクトの情報を伝える。

## 設計方針

Anthropic の [Harness Design for Long-Running Apps](https://www.anthropic.com/engineering/harness-design-long-running-apps) に基づく。

Generator（機能と見た目）と Evaluator（採点）の2者ループを基本とする。Generator は仕様の方針に沿って一貫したビジュアル方針を決め、Evaluator の採点フィードバックを受けて改善する。

## エージェント構成

| エージェント | 役割 | 主な出力 |
|---|---|---|
| Planner | 仕様書・スプリント計画の策定 | `docs/010_製品仕様書/spec.md`、`docs/020_スプリント/sprint-N.md` |
| Generator | スプリント契約の起票、機能とUIの実装 | 動作するアプリケーションコード |
| Evaluator | 契約レビュー、Playwrightによる機能・デザイン評価 | 合格・不合格判定レポート |

標準パイプライン:

```text
Planner → Generator（契約起票）⇄ Evaluator（契約承認）→ Generator（実装とUI）→ Evaluator（最終判定）
                                                                ↑                         │
                                                                └──── 不合格時の修正 ────┘
```

契約は Generator が起票し、Evaluator が承認する。Planner は機能とゴールまでを定義する。

## オーケストレーション手順

### 1. Plannerによる仕様策定

- 新規開発または機能追加の依頼を受けたら開始する。
- `docs/010_製品仕様書/spec.md` と `docs/020_スプリント/sprint-N.md` を作成する。
- スプリントには機能とゴールを記載し、契約条件は空欄にする。
- ユーザーが仕様を承認してから次へ進む。

### 2. Generatorによる契約起票

- `sprint-N.md` に機能とゴールが記載されていることを確認する。
- Generator に「Sprint N の契約を起票してください。実装はまだ行わないでください」と指示する。
- Generator は「何を作るか」「成功をどう検証するか」をテスト可能な条件として `sprint-N.md` に記載する。

### 3. Evaluatorによる契約レビュー

- Evaluator に「Sprint N の契約を実装前レビューしてください」と指示する。
- 仕様適合、テスト可能性、曖昧さ、矛盾を確認させる。
- 差し戻しの場合は Generator に契約を修正させ、承認されるまでレビューを繰り返す。
- 承認されてから実装へ進む。

### 4. Generatorによる実装

- Generator に「承認された Sprint N の契約に基づき、機能とUIを完成させてください」と指示する。
- Generator は仕様に沿った一貫したビジュアル方針を決め、見た目まで実装する。
- Generator の完了報告を受け取ってから評価へ進む。

### 5. Evaluatorによる最終判定

- アプリが起動済みであることを確認する。
- Evaluator に「Sprint N を評価してください。アプリURL: [URL]」と指示する。
- 合格ならスプリントを完了する。
- 不合格なら、機能・デザインの問題をすべて Generator に差し戻す。

### 6. 不合格時のフィードバックループ

- Generator の修正後、Evaluator の最終判定を再実行する。
- 3回連続で不合格になった場合はループを停止し、状況をユーザーへ報告して方針を確認する。

## 継続的な見直し

モデルのバージョンアップ時は、各エージェントの制約を1つずつ取り除いてテストし、品質が落ちない制約を削除する。新しいモデル能力で達成できることがあれば追加する。複雑さは必要性に応じてのみ加える。

## 前提

Evaluator は Playwright MCP を使用するため、評価時には対象アプリが起動している必要がある。
