---
id: sql-create-table
title: CREATE TABLE
description: Create a table.
slug: /sql-create-table
---
<head>
  <link rel="canonical" href="https://docs.risingwave.com/docs/current/sql-create-table/" />
</head>

Use the `CREATE TABLE` command to create a new table. Tables consist of fixed columns and insertable rows. Rows can be added using the [`INSERT`](sql-insert.md) command. When creating a table, you can specify connector settings and data format.

:::info
If you choose to not persist the data from the source in RisingWave, you should use [`CREATE SOURCE`](sql-create-source.md).
:::

## Syntax

```sql
CREATE TABLE [ IF NOT EXISTS ] table_name (
    col_name data_type [ PRIMARY KEY ] [ AS generation_expression ],
    ...
    [ PRIMARY KEY (col_name, ... ) ]
    [ watermark_clause ]
)
[ APPEND ONLY ]
[ WITH (
    connector='connector_name',
    connector_parameter='value', ...)]
[FORMAT data_format ENCODE data_encode [ (
    message='message',
    schema.location='location', ...) ]
];
```

import rr from '@theme/RailroadDiagram'

export const svg = rr.Diagram(
    rr.Stack(
        rr.Sequence(
            rr.Terminal('CREATE TABLE'),
            rr.Optional(rr.Terminal('IF NOT EXISTS')),
            rr.NonTerminal('table_name', 'wrap'),
            rr.Terminal('('),
        ),
        rr.Stack(
            rr.OneOrMore(
                rr.Sequence(
                    rr.NonTerminal('col_name', 'skip'),
                    rr.NonTerminal('data_type', 'skip'),
                    rr.Optional(rr.Terminal('PRIMARY KEY')),
                    rr.Optional(rr.Terminal('AS generation_expression')),
                    rr.Optional(rr.Terminal(',')),
                ),
                rr.Comment('Alternative format: PRIMARY KEY (col_name, ... )'),
            ),
            rr.Optional(rr.Terminal('watermark_clause'), 'skip'),
        ),
        rr.Sequence(
            rr.Terminal(')'),
        rr.Optional(
            rr.Stack(
                rr.Sequence(
                    rr.Terminal('WITH clause'),
            ),
        ),
        ), rr.Terminal(';'),
        ),
    )
);

<drawer SVG={svg} />

This is the WITH clause and the rest of the source parameters:

export const svgTwo = rr.Diagram(
     rr.Stack(
        rr.Optional(
            rr.Stack(
                rr.Sequence(
                    rr.Terminal('WITH'),
                    rr.Terminal('('),
                        rr.Sequence(
                            rr.Terminal('connector'),
                            rr.Terminal('='),
                            rr.NonTerminal('connector_name', 'skip'),
                            rr.Terminal(','),
                        ),
                        rr.OneOrMore(
                            rr.Sequence(
                                rr.NonTerminal('connector_parameter', 'skip'),
                                rr.Terminal('='),
                                rr.NonTerminal('value', 'skip'),
                                rr.Terminal(','),
                            ),
                        ),
                    rr.Terminal(')'),
                ),
            ),
        ),
        rr.Stack(
            rr.Sequence(
                rr.Terminal('FORMAT'),
                rr.NonTerminal('format', 'skip')
            ),
            rr.Sequence(
                rr.Terminal('ENCODE'),
                rr.NonTerminal('encode', 'skip'),
                rr.Optional(
                    rr.Sequence(
                    rr.Terminal('('),
                    rr.NonTerminal('encode_parameter', 'skip'),
                    rr.Terminal(')'),
                    ),
                ),
            ),
        ),
    )
);

<drawer SVG={svgTwo} />

## Notes

For tables with primary key constraints, if you insert a new data record with an existing key, the new record will overwrite the existing record.

Names and unquoted identifiers are case-insensitive. Therefore, you must double-quote any of these fields for them to be case-sensitive.

The syntax for creating a table with connector settings and the supported connectors are the same as for creating a source. See [`CREATE SOURCE`](sql-create-source.md) for a full list of supported connectors and data formats.

To know when a data record is loaded to RisingWave, you can define a column that is generated based on the processing time (`<column_name> timestampz AS proctime()`) when creating the table or source.

## Parameters

| Parameter| Description|
|-----------|-------------|
|`table_name`    |The name of the table. If a schema name is given (for example, `CREATE TABLE <schema>.<table> ...`), then the table is created in the specified schema. Otherwise it is created in the current schema.|
|`col_name`      |The name of a column.|
|`data_type`|The data type of a column. With the `struct` data type, you can create a nested table. Elements in a nested table need to be enclosed with angle brackets ("<\>"). |
|`generation_expression`| The expression for the generated column. For details about generated columns, see [Generated columns](/sql/query-syntax/query-syntax-generated-columns.md).|
|`watermark_clause`| A clause that defines the watermark for a timestamp column. The syntax is `WATERMARK FOR column_name as expr`. For the watermark clause to be valid, the table must be an append-only table. That is, the `APPEND ONLY` option must be specified. This restriction only applies to a table. For details about watermarks, refer to [Watermarks](/transform/watermarks.md).|
|`APPEND ONLY` | When this option is specified, the table will be created as an append-only table. An append-only table cannot have primary keys. `UPDATE` and `DELETE` statements are not valid for append-only tables. Note that append-only tables is an experimental feature. Its functionality is subject to change. You may use this feature at your own risk.|
|**WITH** clause |Specify the connector settings here if trying to store all the source data. See the [Data ingestion](/data-ingestion.md) page for the full list of supported source as well as links to specific connector pages detailing the syntax for each source. |
|**FORMAT** and **ENCODE** options |Specify the data format and the encoding format of the source data. To learn about the supported data formats, see [Data formats](sql-create-source.md#supported-formats). |

## Watermarks

RisingWave supports generating watermarks when creating an append-only streaming table. Watermarks are like markers or signals that track the progress of event time, allowing you to process events within their corresponding time windows. For more information on the syntax on how to create a watermark, see [Watermarks](/transform/watermarks.md).

## Examples

The statement below creates a table that has three columns.

```sql
CREATE TABLE taxi_trips(
    id VARCHAR,
    distance DOUBLE PRECISION,
    city VARCHAR
);
```

The statement below creates a table that includes nested tables.

```sql
CREATE TABLE IF NOT EXISTS taxi_trips(
    id VARCHAR,
    distance DOUBLE PRECISION,
    duration DOUBLE PRECISION,
    fare STRUCT<
      initial_charge DOUBLE PRECISION, 
      subsequent_charge DOUBLE PRECISION, 
      surcharge DOUBLE PRECISION, 
      tolls DOUBLE PRECISION>);
```

The statement below creates a materialized source with a Kafka broker as the source.

```sql
CREATE TABLE IF NOT EXISTS source_abc (
   column1 varchar,
   column2 integer,
)
WITH (
   connector='kafka',
   topic='demo_topic',
   properties.bootstrap.server='172.10.1.1:9090,172.10.1.2:9090',
   scan.startup.mode='latest',
   scan.startup.timestamp_millis='140000000',
) FORMAT PLAIN ENCODE JSON;
```
