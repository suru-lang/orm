English | [日本語](README_jp.md)

# 台帳 (an O/R mapper for Suru)

An O/R mapper for [Suru (「する」)](https://github.com/suru-lang/suru), a programming language in Japanese. It maps records (組) to database tables: save a record, get it back by its primary key, search with a condition. It runs on the database layer [「蔵」](https://github.com/suru-lang/db), so the same program works on SQLite, PostgreSQL and MySQL. Plan: suru-lang/suru#164.

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
  大人たち とは 会員表 から 「年齢 >= ?」 を （（20 の DB数） を 空 に 挿入する） で db に 表検索した 結果
  （大人たち の それぞれ を 紹介文にして 改行 で 結合する） を 表示する
  db を DB閉鎖する
  「以上です」 を 表示する

主処理する
```

Replacing `SQLite駆動` with `PostgreSQL駆動` or `MySQL駆動` (and the connection target) runs it on that database. The example is in [例/](例/) (`kumu 実行` there).

## Decisions

1. **A table definition is a record.** `表定義` is the table name, the columns, and two functions: one that turns a record into a row (a list of `DB値` in column order) and one that turns a row back into a record. The language is not changed for this; generating these from the record definition is planned (see Future work).
2. **Columns.** `「番号」 の 主鍵列`, `整数列`, `小数列`, `文字列列`, `バイト列列` and `真偽列`. The first column must be the `主鍵列`, an auto-numbered integer. Column types come from the connection's dialect (`INTEGER PRIMARY KEY`, `BIGSERIAL PRIMARY KEY`, `BIGINT AUTO_INCREMENT PRIMARY KEY` and so on).
3. **Taking values out of a row.** `値群 から 2 を 番目文字列取出する` (also `番目数取出する`, `番目バイト列取出する`, `番目真偽取出する`) takes the n-th value (from 1). `番目真偽取出する` also accepts 1 and 0, for SQLite and MySQL.
4. **Saving.** `表 で x を db に 表保存する` INSERTs a record whose primary key is 0 and gives the record with the new number filled in (from `INSERT … RETURNING` where the dialect allows it, otherwise from `DB最終番号取得する`). A record with any other primary key is UPDATEd and given back unchanged.
5. **Getting and searching.** `表 から 番号 を db で 表取得する` gives `x の 有` or `無`. `表 から 条件 を 値群 で db に 表検索する` adds `WHERE 条件` (written with 「?」 like everywhere in 「蔵」; ORDER BY may follow) and gives a list of records. `表 から db に 表全件取得する` gives all records in primary key order.
6. **Creating and deleting.** `表 を db に 表作成する` runs `CREATE TABLE IF NOT EXISTS`. `表 で x を db に 表削除する` deletes by primary key and gives the number of deleted rows.
7. **The SQL can be seen.** `表 を 方言 で 表作成文にする` (also `表挿入文にする`, `表更新文にする`, `表選択文にする`, `表削除文にする`) are pure functions giving the SQL; names are quoted with the dialect. Transactions are those of 「蔵」 (`db で 処理 を DB取引する`).

## Tests

```sh
kumu 試験          # the SQL it builds, for the three dialects (no database)
kumu 点検
```

[結合試験/](結合試験/) runs the same tests (save, update, get, search, delete, rollback) on real databases: `kumu 試験` in 結合試験/SQLite, 結合試験/PostgreSQL (`PGHOST` etc.) and 結合試験/MySQL (`MYSQL_HOST` etc.). The shared test body is 結合試験/共通/台帳共通試験.suru.

## Future work

- Conditions as values instead of strings, ordering and limits, counting
- Foreign keys and one-to-many fetches
- Generating the table definition from the record definition (with lib/構文.suru)
- Migrations (adding columns)

## License

[MIT](LICENSE)
