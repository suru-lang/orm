English | [日本語](README_jp.md)

# 台帳 (an O/R mapper for Suru)

An O/R mapper for [Suru (「する」)](https://github.com/suru-lang/suru), a programming language in Japanese. It maps records (組) to database tables: save a record, get it back by its primary key, search with conditions written as values, and fetch one-to-many relations. It runs on the database layer [「蔵」](https://github.com/suru-lang/db), so the same program works on SQLite, PostgreSQL and MySQL. Plan: suru-lang/suru#164.

## Installing

In a [kumu](https://github.com/suru-lang/suru#building-projects-kumu) project, add 台帳 and the driver for your database:

```sh
kumu 追加 github.com/suru-lang/orm
kumu 追加 github.com/suru-lang/sqlite
```

「蔵」 (suru-lang/db) comes along as a dependency. The drivers are `SQLite駆動` ([suru-lang/sqlite](https://github.com/suru-lang/sqlite)), `PostgreSQL駆動` ([suru-lang/postgresql](https://github.com/suru-lang/postgresql)) and `MySQL駆動` ([suru-lang/mysql](https://github.com/suru-lang/mysql)).

## Usage

```
「蔵」 を 読み込んで 使う
「台帳」 を 読み込んで 使う
「SQLite」 を 読み込んで 使う
「標準」 を 読み込んで 使う

会員 とは 番号 と 名前 と 年齢 の 組

会員 を 会員行化する は DB値 の リスト
m を 会員行化する とは （m の 番号 の DB数） と （m の 名前 の DB文字列） と （m の 年齢 の DB数）

DB値 の リスト を 会員組化する は 会員
値群 を 会員組化する とは （値群 から 1 を 番目数取出する） と （値群 から 2 を 番目文字列取出する） と （値群 から 3 を 番目数取出する） の 会員

会員表 とは 「会員」 と （（「番号」 の 主鍵列） と （「名前」 の 文字列列） と （「年齢」 の 整数列）） と 会員行化すること と 会員組化すること の 表定義

会員 を 紹介文にする は 文字列
m を 紹介文にする とは （m の 番号 を 文字列化する） + 「番の 」 + （m の 名前） + 「さんは 」 + （m の 年齢 を 文字列化する） + 「歳です」

主処理する とは 入出力手順
  db とは 「:memory:」 を SQLite駆動 で DB開封した 結果
  会員表 を db に 表作成する
  花子 とは 会員表 で （0 と 「花子」 と 30 の 会員） を db に 表保存した 結果
  会員表 で （0 と 「太郎」 と 15 の 会員） を db に 表保存する
  会員表 で （年齢 は 31 の 花子） を db に 表保存する
  大人たち とは 会員表 から （「年齢」 と （20 の DB数） の 以上条件） を db で 表条件検索した 結果
  （大人たち の それぞれ を 紹介文にして 改行 で 結合する） を 表示する
  若い順 とは 全件条件 を 照会にして 「年齢」 で 昇順並べする
  （（会員表 から 若い順 を db で 表照会した 結果） の それぞれ を 紹介文にして 改行 で 結合する） を 表示する
  （「会員は 」 + （（会員表 から 全件条件 を db で 表件数取得した 結果） を 文字列化する） + 「人です」） を 表示する
  db を DB閉鎖する
  「以上です」 を 表示する

主処理する
```

Replacing `SQLite駆動` with `PostgreSQL駆動` or `MySQL駆動` (and the connection target) runs it on that database. The example is in [例/](例/) (`kumu 実行` there).

## Decisions

1. **A table definition is a record.** `表定義` is the table name, the columns, and two functions: one that turns a record into a row (a list of `DB値` in column order) and one that turns a row back into a record. The language is not changed for this; generating these from the record definition is planned (see Future work).
2. **Columns.** `「番号」 の 主鍵列`, `整数列`, `小数列`, `文字列列`, `バイト列列`, `真偽列`, and `参照列` (see Relations below). The first column must be the `主鍵列`, an auto-numbered integer. Column types come from the connection's dialect (`INTEGER PRIMARY KEY`, `BIGSERIAL PRIMARY KEY`, `BIGINT AUTO_INCREMENT PRIMARY KEY` and so on).
3. **Taking values out of a row.** `値群 から 2 を 番目文字列取出する` (also `番目数取出する`, `番目バイト列取出する`, `番目真偽取出する`) takes the n-th value (from 1). `番目真偽取出する` also accepts 1 and 0, for SQLite and MySQL.
4. **Saving.** `表 で x を db に 表保存する` INSERTs a record whose primary key is 0 and gives the record with the new number filled in (from `INSERT … RETURNING` where the dialect allows it, otherwise from `DB最終番号取得する`). A record with any other primary key is UPDATEd and given back unchanged.
5. **Getting and searching.** `表 から 番号 を db で 表取得する` gives `x の 有` or `無`. `表 から 条件 を 値群 で db に 表検索する` adds `WHERE 条件` (a string written with 「?」 like everywhere in 「蔵」; ORDER BY may follow; conditions as values are usually easier, see below) and gives a list of records. `表 から db に 表全件取得する` gives all records in primary key order.
6. **Creating and deleting.** `表 を db に 表作成する` runs `CREATE TABLE IF NOT EXISTS`. `表 で x を db に 表削除する` deletes by primary key and gives the number of deleted rows.
7. **The SQL can be seen.** `表 を 方言 で 表作成文にする` (also `表挿入文にする`, `表更新文にする`, `表選択文にする`, `表削除文にする`) are pure functions giving the SQL; names are quoted with the dialect. Transactions are those of 「蔵」 (`db で 処理 を DB取引する`).
8. **Conditions are values.** `表条件` is `全件条件` (every row), `「年齢」 と （20 の DB数） の 以上条件` (also `等号条件` =, `不等条件` <>, `未満条件` <, `以下条件` <=, `超過条件` >), `「名前」 と 「花%」 の 類似条件` (LIKE; whether case matters depends on the database), `「名前」 の 無値条件` (IS NULL), `「番号」 と 値群 の 含有条件` (IN; an empty list matches nothing), `条件群 の 全部条件` (AND; empty matches everything), `条件群 の 一部条件` (OR; empty matches nothing) and `c の 否定条件` (NOT). Values go to 「?」, never into the SQL text, and column names are quoted with the dialect. `表 から 条件 を db で 表条件検索する` gives the matching records in primary key order.
9. **Ordering and limits.** `条件 を 照会にする` makes a `表照会`; chain `「年齢」 で 降順並べする` (or `昇順並べする`; several are applied in order), `10 で 上限付けする` and `20 で 開始付けする` (OFFSET) with 「して」, as in `全件条件 を 照会にして 「年齢」 で 降順並べして 10 で 上限付けする`. `表 から 照会 を db で 表照会する` gives the records. With no ordering it is primary key order; a limit of 0 means no limit.
10. **Counting and deleting by condition.** `表 から 条件 を db で 表件数取得する` gives the number of matching rows (COUNT(*)), and `表 から 条件 を db で 表条件削除する` deletes them and gives how many.
11. **Relations (one-to-many).** In the child table, `親表 を 「利用者番号」 で 参照列にする` is an integer column with `REFERENCES` to the parent's primary key. `子表 から 「利用者番号」 で 番号 を db に 表関連取得する` gives the children of the parent with that number, in primary key order; the parent of a child is `親表 から （子 の 利用者番号） を db で 表取得する`. Create the parent table first and drop the child table first (SQLite checks the reference only with `PRAGMA foreign_keys = ON`).
12. **The SQL for queries can be seen too.** `条件 を 方言 で 条件句にする` gives a `表句` (`句文`, the text after WHERE, and `句値群`, the values for its 「?」). `表 から 照会 を 方言 で 表照会文にする` and `表 から 条件 を 方言 で 表件数文にする` give the whole SQL.

## Tests

```sh
kumu 試験          # the SQL it builds, for the three dialects (no database)
kumu 点検
```

[結合試験/](結合試験/) runs the same tests (save, update, get, search, conditions, ordering and limits, relations, delete, rollback) on real databases: `kumu 試験` in 結合試験/SQLite, 結合試験/PostgreSQL (`PGHOST` etc.) and 結合試験/MySQL (`MYSQL_HOST` etc.). The shared test body is 結合試験/共通/台帳共通試験.suru.

## Future work

- Fetching the children together with the parents in one query (JOIN), and many-to-many through a join table
- Generating the table definition from the record definition (with lib/構文.suru)
- Migrations (adding columns)

## License

[MIT](LICENSE)
