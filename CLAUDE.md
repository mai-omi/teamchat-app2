# CLAUDE.md

このファイルは、このリポジトリで作業するClaude Code（claude.ai/code）への指示を記載したものです。

**最初に読む3行**：社内向けチームチャットアプリを、設計書を書きながら作る学習用の演習。実装コードはまだ1行も無い。まず `docs/spec.md`（要求）と `mockup/`（画面見本）を読み、`docs/design/` に設計を書いてから実装する。

## このリポジトリについて

これは、素のLaravel 10スケルトンの上に構築された学習用の演習（「Tutorial 16」）です。**アプリケーションのコードはまだ1行も書かれていません** — `app/` にはデフォルトのLaravel雛形（`User` モデルが1つと、デフォルトのmiddleware/providerのみ）しか無く、`routes/web.php` / `routes/api.php` もフレームワークの初期状態のままです。実際の作業は、仕様書とモックアップを読み、`docs/design/` に設計を書き、それから実装する、という流れです。

設計・実装のすべての判断は、次の3つを根拠とします（権威の優先順位順）。

1. **`docs/spec.md`** — クライアントからの要求を文章にしたもの。変更しないこと。曖昧な点や書かれていない点に気づいたら、勝手に想定で埋めるのではなく、確認事項（`docs/design/questions.md` 参照）としてまとめる。
2. **`mockup/*.html`** — 画面の正となる見本（静的HTML10枚。ブラウザで `mockup/index.html` を開く。JSもビルドも不要）。ピクセル単位での一致は不要だが、並び・項目名・言葉づかい・状態の見せ方（エラー・空・編集中・ホバー）は合わせる。色・余白・状態の決まりは `mockup/design-guide.md` を参照。
3. **`docs/design/*.md`** — 上記2つから導いた設計書一式。設計フェーズの提出物であり、実装の根拠となる。実装中に決めが変わったら、**コードより先に設計書を直す**。実物と設計書を食い違ったままにしないこと。

## 正本の在りか

中身はここに写さない。必要になったら開くこと。

| 何が書いてあるか | どこにあるか |
|:--|:--|
| 仕様書（クライアントの要求） | `docs/spec.md`。**変えない** |
| 設計書一式（このアプリの決めごとの正本） | `docs/design/`（構成は `docs/design/README.md`） |
| 画面の見本 | `mockup/`（`index.html` から全画面） |
| 受け入れ条件とテスト観点 | `docs/design/acceptance.md` |

実装と設計書が食い違ったら、**コードではなく `docs/design/` を先に直す**。設計書は提出物の中心で、最後まで実物と一致させる。

## 設計書一式（`docs/design/`）

Tutorial 13の章ごとの型に対応する、7つの固定ファイル — **ファイル名を変えたり統合したりしない。1ファイルに違う種類を混ぜない**。

| ファイル | 内容 |
|:--|:--|
| `features.md` | 機能一覧（F-01〜F-31）と、今回作らないものの明示 |
| `questions.md` | クライアントへの確認事項リストと、回答／決定事項 |
| `screens.md` | 画面一覧・画面遷移図・画面項目定義・入力チェックとメッセージ |
| `data.md` | ER図・テーブル定義書・CRUD表（行＝F-ID、列＝テーブル） |
| `permissions-api.md` | 権限マトリクス（行＝F-ID、列＝未ログイン/社員/メンバー/作成者）・ルーティング一覧・公開APIの設計 |
| `behavior.md` | 状態遷移図（メッセージのみ）・代表的なシーケンス図1本・例外方針 |
| `acceptance.md` | 実装単位ごとの受け入れ条件（各2〜4行）とテスト観点 |

これらの設計書に書くことは、必ず根拠を本文中に示す（例：`spec.md 3-2`、`mockup/channel-new.html`、`questions.md Q-04`、`features.md F-09`）。これらの設計書を元に拡張・実装する際も、根拠を明示しない記述ではなく、この「根拠を辿れる」書き方を保つこと。

図はMarkdown中にMermaidで書く — 画像ファイルは置かず、クラス図も作らない（LaravelのMVCの形にそのまま乗ることで足りる、という判断）。設計書は意図的に簡潔にしてある：シーケンス図は代表1本だけ、画面項目定義は主要画面のみ（同じ形の画面は「◯◯と同じ」で済ませる）、受け入れ条件は1単位あたり2〜4行、テスト観点は網羅ではなく観点のみ。

## ドメインモデル（`docs/design/data.md` で既に決定済み — ゼロから考え直さない）

- `users`: 表示名（30文字以内）、email（一意）、password（ハッシュ化）。
- `channels`: name（50文字以内、一意）、description（200文字以内、任意）、`type` enum `public`/`private`（**作成後は変更不可**）、`creator_id`。編集・削除ができるのは作成者のみ。
- `channel_user`: メンバーの行 — **プライベートチャンネルのみ作成する**。公開チャンネルの行は作らず、公開チャンネルの閲覧・投稿権限は `channels.type = public` のみで判定する。チャンネル作成者は暗黙的にメンバー扱い（公開：`creator_id` のみで表現／プライベート：作成時に `channel_user` の行も作る）。
- `messages`: `parent_id` による自己参照（返信。**入れ子にはしない。1階層のみ**）、`channel_id` を直接保持（返信にも持たせ、親をたどらずにそのメッセージのチャンネルが分かるようにする）、`edited_at`（nullなら未編集）、`deleted_at`（**論理削除** — 行と `body` はそのまま保持し、UIでは「このメッセージは削除されました」に置き換えて表示。削除済みメッセージにはこれ以上の編集・削除操作を出さない）。返信件数は `parent_id` が一致する行を数えて都度算出し、カラムにキャッシュしない。
- チャンネルの削除は**物理削除**で、配下のメッセージ・メンバーも一緒に消える（`questions.md` で取り消せない操作として確認済み） — ここだけが意図的な物理DELETEの箇所であり、メッセージの削除は常に論理削除。

## 権限の構造

チャンネルごとに評価する4つの利用者区分：未ログイン／社員（ログイン済み社員）／メンバー（プライベートチャンネルに追加された人）／作成者。主な規則：

- チャンネルの編集・削除、メンバー管理ができるのは作成者のみ。作成者は自分自身をメンバーから外せない。
- 自分が閲覧できるチャンネルであれば、誰でも**自分の**メッセージを投稿・編集・削除できる。チャンネルの作成者であっても他人のメッセージは操作できない。
- 検索と公開APIには「呼び出せるかどうか」の権限ゲートが無い（検索：ログイン済みなら誰でも／API：未ログインでも可）— 実際のアクセス制御は、**返す内容の絞り込み**（検索：閲覧可能なチャンネルに限定／API：`type = public` のみ）で行う。
- 自分がメンバーでないプライベートチャンネルのURLに直接アクセスした場合、および公開APIでプライベート／存在しないチャンネルを指定された場合は、**403ではなく404**を返す — プライベートチャンネルの存在自体を漏らさないため。

## ルーティング（`docs/design/permissions-api.md` に基づく計画）

`/login`・`/register`・公開の読み取り専用API2本（`GET /api/channels`、`GET /api/channels/{channel}/messages`）を除き、すべて `auth` ミドルウェア配下の、Laraver標準的なリソースルーティング。スレッドパネル（`mockup/thread.html`）は独立したルートではなく、`screens.md` のとおりチャンネル表示画面の1状態として、クエリパラメータ（例：`?thread={message}`）で表現する。返信も通常のメッセージ投稿と同じルートを通り、`parent_id` の有無で区別する。

## 初期データ投入の要件

`spec.md 5-4` のとおり、アプリは固定の初期データ（任意のfactoryではなく）で起動する必要がある：指定の3アカウント（sato@example.com／suzuki@example.com／takahashi@example.com、パスワードは `password`）、仕様書で指定された名前・作成者の4チャンネル（公開3・メンバー2名のプライベート1）、各公開チャンネルに約10件のメッセージ（編集済み1件以上・削除済み1件以上・3件以上の返信が付くスレッドを含む）。これらの固定値はシーディング時に変更しないこと。

## 実装時の運用ルール

`~/laravel-practice/post-app-practice/CLAUDE.md` から、このアプリにもそのまま通じるものだけ移したもの。

- バリデーションエラーメッセージはすべて日本語で出力する。
- マイグレーション実行後は出力を必ず確認し、エラーがないことを見てから次の作業に進む。
- 一覧表示のクエリには `with()` による Eager Loading が付いているかを確認する（N+1防止）。

## 開発コマンド

このプロジェクトはLaravel Sail（Docker）で動かす。`README.md` より：

```bash
# 初回セットアップ
docker run --rm -u "$(id -u):$(id -g)" -v "$(pwd):/var/www/html" -w /var/www/html \
  -e COMPOSER_CACHE_DIR=/tmp/composer_cache laravelsail/php82-composer:latest composer install
cp .env.example .env
./vendor/bin/sail up -d          # mysqlがhealthyになるまで30秒ほど待つ（./vendor/bin/sail ps で確認）
./vendor/bin/sail artisan key:generate
./vendor/bin/sail artisan migrate       # ここで --seed は付けない。初期データは機能実装と合わせて積む

./vendor/bin/sail stop            # 停止のみ（データは残る）
./vendor/bin/sail down -v         # 停止してデータベースの中身も消す
```

よく使う開発コマンド（いずれも `./vendor/bin/sail` 経由でアプリコンテナ内に実行される）：

```bash
./vendor/bin/sail artisan migrate:fresh --seed   # スキーマ再構築＋初期データ投入
./vendor/bin/sail artisan test                   # テスト全体を実行
./vendor/bin/sail artisan test --filter TestName # 単一テストを実行
./vendor/bin/sail artisan tinker
./vendor/bin/sail npm run dev                    # viteの開発サーバー
./vendor/bin/sail npm run build
./vendor/bin/sail composer require <pkg>
```

PHPUnit（`phpunit.xml`）は `tests/Unit` と `tests/Feature` に分かれている。テスト環境ではcache/sessionにインメモリのarrayドライバ、queueはsync、DBはSailのMySQLコンテナが自動作成する別の `testing` データベースを使う。

## プルリクエスト

イシュー1件に1本のブランチを切り、プルリクエストで受け取る。実装単位は `docs/design/acceptance.md` の7つ（1.認証／2.チャンネル／3.メンバー管理／4.メッセージ／5.スレッド／6.検索／7.公開API）で固定。1単位＝1イシュー＝1PRとして、このあと7回繰り返す。

本文には、テストの実行結果と、受け入れ条件それぞれの判定を載せ、最後に `Closes #<イシュー番号>` を入れる。毎回この型に沿って書く。

```markdown
## 概要
- 実装単位：<番号>. <単位名>（例：4. メッセージ）
- 対応するF-ID：<例：F-15〜F-19>

## テスト実行結果
`./vendor/bin/sail artisan test` の結果（件数・Pass/Fail）を貼る。

## 受け入れ条件との対応
`docs/design/acceptance.md` のAC-IDごとに、判定（Pass/Fail）と対応箇所を示す（条件ごとに1行。埋まらないAC-IDを残したままPRを出さない）。

| AC-ID | 受け入れ条件 | 判定 | 対応箇所 |
| :-- | :-- | :-- | :-- |
| AC-n-1 | | | |

Closes #<イシュー番号>
```
