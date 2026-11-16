# 受け入れ確認報告

実施日：2026-11-16

本書は、`docs/design/features.md`・`docs/design/data.md`（CRUD表）と実物（`docs/spec.md`・`docs/design/questions.md`・`mockup/`）の突き合わせ、テスト実行、コード整形チェックの3本を実施した結果の報告。**今回は確認のみで、設計書・仕様書への修正は行っていない。**

---

## 1. 設計書と実物の突き合わせ

対象：`docs/design/features.md`（機能一覧 F-01〜F-31）、`docs/design/data.md`（CRUD表）
比較先：`docs/spec.md`、`docs/design/questions.md`、`mockup/*.html`、`mockup/design-guide.md`

### 見つかったズレ

| # | 対象 | ズレの内容 | どちらに合わせるべきか（判断・未適用） |
| :-- | :-- | :-- | :-- |
| 1 | `features.md` F-10の根拠欄 | 根拠として挙げている `mockup/channels.html` の注記（「※ 見本のため、自分が作ったチャンネルにだけ『メンバー』『編集』が並んだ状態を示しています」）は、F-10の内容（公開チャンネルにメンバー管理操作がない）とは無関係 | 実物（`mockup/members.html` の「※ この画面を開けるのは、プライベートチャンネルを作った人だけです。公開チャンネルにはメンバーの管理はありません。」）に合わせ、`features.md` の根拠欄をこちらに直すべき。**未適用** |
| 2 | `data.md`（冒頭・F-17のCRUD=U）、`behavior.md` | メッセージ削除を論理削除（`deleted_at`）と確定して設計しているが、`docs/design/questions.md` Q-13は本文上「状態：保留」のまま。回答の追記がされていない | 実物（`questions.md`）側を更新し、Q-13に決定事項を追記して「回答済み」にすべき。設計側（論理削除という結論自体）は変更不要。**未適用** |
| 3 | `data.md`（`channel_user`は公開チャンネルでは行を作らない、という方針の根拠） | 「この会話で確認して決めた」とのみ記載され、`questions.md` 上に対応するQ-IDが無い | 内容自体は`spec.md 3-2`／`3-3`と矛盾しないため、`questions.md`側にQ-IDとして追記し、根拠を辿れる状態にすべき。**未適用** |

### 問題なしと確認できたもの

- `features.md` のスコープ外表（今回作らないもの）は `spec.md` 4.の表と完全一致（転記どおり）
- 追加のスコープ外2項目（スマートフォン対応／返信の入れ子）は `spec.md 5-2`・`mockup/design-guide.md`・`questions.md` Q-09と整合
- `data.md` CRUD表のF-ID（F-01〜F-31）は `features.md` と1:1で対応し、過不足なし
- 上記1件を除き、各F-IDの根拠（`mockup/*.html` の文言・項目）は実物と一致

---

## 2. テスト実行結果

```
./vendor/bin/sail artisan test
```

```
PASS  Tests\Unit\ExampleTest
✓ that true is true                                                    0.01s

DEPR  Tests\Feature\ExampleTest
! the application returns a successful response → Constant PDO::MYSQL… 0.09s

Tests:    1 deprecated, 1 passed (2 assertions)
Duration: 0.15s
```

判定：**1 passed, 1 deprecated（2 assertions）。Fail無し**。

`app/` にはLaravel雛形のみでアプリ固有のコードが無いため（`CLAUDE.md` の記載どおり）、既定の `ExampleTest` 以外のテストは存在しない。deprecation警告は `PDO::MYSQL…` 定数に関するPHP/MySQLドライバ側の非推奨通知で、テスト失敗ではない。

---

## 3. コード整形チェック

```
./vendor/bin/sail pint --test
```

```json
{"tool":"pint","result":"passed"}
```

判定：**passed（整形違反なし）**。

---

## 4. 引き渡しの状態

- ブランチ：`main`
- マイグレーション状態（`./vendor/bin/sail artisan migrate:status`）：Laravel標準の4本（`users`／`password_reset_tokens`／`failed_jobs`／`personal_access_tokens`）のみ `Ran`。アプリ固有のテーブル（`channels`／`channel_user`／`messages`）はまだ存在しない
- コンテナ：`teamchat-app2-mysql-1`（healthy）／`teamchat-app2-laravel.test-1` とも起動中
- 未コミットの変更：`docs/spec.md` に整形のみの差分（Markdownテーブルの桁揃え）が残っている。本文内容の変更は無いが、`CLAUDE.md` は `spec.md` を「変更しないこと」としているファイルであり、**今回の3本の確認作業より前（2026-11-15時点）から存在する差分**。コミットするか、元に戻すかの判断が必要
- 実装コード：`app/` はLaravel雛形のみで、アプリケーション固有の実装は1行も無い（設計フェーズの状態のまま）
