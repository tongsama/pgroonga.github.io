---
title: pgroonga_query_escape関数
upper_level: ../
---

# `pgroonga_query_escape`関数

1.1.9で追加。

## 概要

`pgroonga_query_escape`関数は[クエリー構文](http://groonga.org/ja/docs/reference/grn_expr/query_syntax.html)で特別な意味を持つ文字をエスケープします。[`&@~`演算子][query-v2]、[`&@~|`演算子][query-in-v2]などがクエリー構文を使っています。

`pgroonga_query_escape`関数は[`pgroonga_command`関数](pgroonga-command.html)経由でGroongaコマンドインジェクションが発生することを防ぐときに有用です。Groongaコマンドインジェクションを防ぐことについては[`pgroonga_command_escape_value`関数](pgroonga-command-escape-value.html)と[`pgroonga_escape`関数](pgroonga-escape.html)も参照してください。

## 構文

この関数の構文は次の通りです。

```text
text pgroonga_query_escape(query)
```

`query`は[クエリー構文](http://groonga.org/ja/docs/reference/grn_expr/query_syntax.html)を使っている`text`型の値です。

`pgroonga_query_escape`は`text`型の値を返します。この値の中の特別な意味を持つ文字はすべてエスケープされています。

## 使い方

サンプルスキーマとデータは次の通りです。

```sql
CREATE TABLE memos (
  content text
);

CREATE INDEX pgroonga_memos_index
          ON memos
       USING pgroonga (content);

INSERT INTO memos VALUES ('PGroonga (PostgreSQL+Groonga) is great!');
```

クエリーが「(PostgreSQL」の場合はエラーが発生します。なぜなら対応する閉じカッコがないからです。

```sql
SELECT * FROM memos WHERE content @@ '(PostgreSQL';
-- ERROR:  pgroonga: failed to parse expression: Syntax error: <(PostgreSQL||>
```

`pgroonga_query_escape`関数を使うことで「(PostgreSQL」というクエリーそのもの（「(」を特別な文字として扱わない）で検索できます。

```sql
SELECT * FROM memos WHERE content @@ pgroonga_query_escape('(PostgreSQL');
--                  content                 
-- -----------------------------------------
--  PGroonga (PostgreSQL+Groonga) is great!
-- (1 row)
```

[`pgroonga_command`関数](pgroonga-command.html)でも同じことが発生します。

```sql
SELECT jsonb_pretty(
  pgroonga_command('select ' ||
                   '--table ' || pgroonga_table_name('pgroonga_memos_index') || ' ' ||
                   '--match_columns content ' ||
                   '--query "(PostgreSQL"')::jsonb
);
--                jsonb_pretty               
-- ------------------------------------------
--  [                                       +
--      [                                   +
--          -63,                            +
--          1480432652.751489,              +
--          0.0007565021514892578,          +
--          "Syntax error: <(PostgreSQL||>" +
--      ]                                   +
--  ]
-- (1 row)
```

`pgroonga_query_escape`関数を[`pgroonga_command_escape_value`関数](pgroonga-command-escape-value.html)と一緒に使うとこのケースを防ぐことができます。

```sql
SELECT jsonb_pretty(
  pgroonga_command('select ' ||
                   '--table ' || pgroonga_table_name('pgroonga_memos_index') || ' ' ||
                   '--match_columns content ' ||
                   '--query ' || pgroonga_command_escape_value(pgroonga_query_escape('(PostgreSQL')))::jsonb
);
--                         jsonb_pretty                        
-- ------------------------------------------------------------
--  [                                                         +
--      [                                                     +
--          0,                                                +
--          1480432832.061276,                                +
--          0.0252687931060791                                +
--      ],                                                    +
--      [                                                     +
--          [                                                 +
--              [                                             +
--                  1                                         +
--              ],                                            +
--              [                                             +
--                  [                                         +
--                      "_id",                                +
--                      "UInt32"                              +
--                  ],                                        +
--                  [                                         +
--                      "content",                            +
--                      "LongText"                            +
--                  ],                                        +
--                  [                                         +
--                      "ctid",                               +
--                      "UInt64"                              +
--                  ]                                         +
--              ],                                            +
--              [                                             +
--                  1,                                        +
--                  "PGroonga (PostgreSQL+Groonga) is great!",+
--                  1                                         +
--              ]                                             +
--          ]                                                 +
--      ]                                                     +
--  ]
-- (1 row)
```

コマンドの引数を配列で指定するスタイルで[`pgroonga_command`関数](pgroonga-command.html)使ってもこのケースを防ぐことができます。

```sql
SELECT jsonb_pretty(
  pgroonga_command('select',
                   ARRAY[
                     'table', pgroonga_table_name('pgroonga_memos_index'),
                     'match_columns', 'content',
                     'query', pgroonga_query_escape('(PostgreSQL')
                   ])::jsonb
);
--                         jsonb_pretty                        
-- ------------------------------------------------------------
--  [                                                         +
--      [                                                     +
--          0,                                                +
--          1480433038.482539,                                +
--          0.0001201629638671875                             +
--      ],                                                    +
--      [                                                     +
--          [                                                 +
--              [                                             +
--                  1                                         +
--              ],                                            +
--              [                                             +
--                  [                                         +
--                      "_id",                                +
--                      "UInt32"                              +
--                  ],                                        +
--                  [                                         +
--                      "content",                            +
--                      "LongText"                            +
--                  ],                                        +
--                  [                                         +
--                      "ctid",                               +
--                      "UInt64"                              +
--                  ]                                         +
--              ],                                            +
--              [                                             +
--                  1,                                        +
--                  "PGroonga (PostgreSQL+Groonga) is great!",+
--                  1                                         +
--              ]                                             +
--          ]                                                 +
--      ]                                                     +
--  ]
-- (1 row)
```

## 参考

  * [`pgroonga_command`関数][command]

  * [`pgroonga_command_escape_value`関数][command-escape-value]

  * [`pgroonga_escape`関数][escape]

[query-v2]:../operators/query-v2.html

[query-in-v2]:../operators/query-in-v2.html

[command]:pgroonga-command.html
[command-escape-value]:pgroonga-command-escape-value.html
[escape]:pgroonga-escape.html
