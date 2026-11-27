# 「台帳」

「台帳」, the O/R mapper of Suru. It maps records to database tables, on top of the database layer 「蔵」.
Add it with `kumu 追加 github.com/suru-lang/orm` and write `「台帳」 を 読み込んで 使う` in your file.
The connection to the database is made with 「蔵」 and a driver (`「蔵」 を 読み込んで 使う` and a driver such as `SQLite駆動`).

A `表定義` is a record of the table name, the columns, and how to go between records and rows:

```suru
利用者表 とは 「利用者」 と 利用者列群 と 利用者行化すること と 利用者組化すること の 表定義
```

- The columns are `「番号」 の 主鍵列`, `「名前」 の 文字列列`, `整数列`, `小数列`, `バイト列列` and `真偽列`. The first column is the primary key (an auto-numbered number).
- The row function turns a record into the list of `DB値` in column order, and the record function does the reverse (`値群 から 2 を 番目文字列取出する` and so on help).
- A record whose primary key is 0 is new and not in the table yet.
- For a record with field types, `会員 は 【表】 を 導出する` makes `会員行化する`, `会員組化する` and `会員表` (Suru 0.10.0 or later).

The functions that touch the database are actions (db is the `DB接続` of 「蔵」).

Conditions (`表条件`): `全件条件`, `「年齢」 と （20 の DB数） の 以上条件` (also `等号条件`, `不等条件`, `未満条件`, `以下条件` and `超過条件`), `「名前」 と 「花%」 の 類似条件`, `「名前」 の 無値条件`, `「番号」 と 値群 の 含有条件`, `条件群 の 全部条件` (AND), `条件群 の 一部条件` (OR) and `c の 否定条件`.

Queries (`表照会`): `条件 を 照会にして 「年齢」 で 降順並べして 10 で 上限付けして 20 で 開始付けする`.

The functions that end in `文にする` give the SQL without running it, for the dialect of the database (`db の DB方言値`).

```{toctree}
:maxdepth: 2
:caption: API
:glob:

api/*
```
