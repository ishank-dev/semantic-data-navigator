---
layout: default
title: Working with partitions
description: Learn when and how to partition Firebolt tables to accelerate query performance and simplify table maintenance.
parent: Working with tables
grand_parent: Overview
nav_order: 2
---

# Working with partitions
{: .no_toc}

* Topic ToC
{: toc}

Partitions are smaller physical parts of large fact and dimension tables. Partitions provide the first layer of sorting when you ingest data. Data is sorted in storage by partition first, and then pruned and sorted by the primary index definition next. When new data is ingested into a table, Firebolt saves rows automatically in the appropriate partition.

## When to use partitions

Partitions are particularly useful to simplify table maintenance by allowing you to drop partitions and delete rows in bulk. For example, consider a transaction table with an average of approximately 1,500,000 transactions a day, which you partition by month. At the end of each month, you can run [ALTER TABLE](../../sql_reference/commands/data-definition/alter-table.md) to delete the last month's data, and then [INSERT](../../sql_reference/commands/data-management/insert.md) to update the fact table with the most recent month's data.

{: .warning}
Dropping a partition deletes all the records stored in the partition.

## Considerations for partitioning and query performance

Although some applications may see a performance boost from partitions, Firebolt does not rely on partitioning for performance. Firebolt's indexing features are enough for many applications.

We recommend that you benchmark your application with and without partitions.

### Partition only large tables
We recommend that you consider partitioning only for tables greater than 100 million rows.

### Large, equally distributed partitions work best

Too many small partitions to read increases I/O and decreases performance. In addition, skewed (asymmetrically distributed) partitions can lead to poor performance. Choose columns for partition keys that create large partitions of relatively equal size.

## Defining partition keys

You define partitions using the [PARTITION BY](../../sql_reference/commands/data-definition/create-fact-dimension-table.md#partition-by) clause in a `CREATE TABLE` statement.

Rows with the same value in the column key and whose function expression resolves to the same value are included in the partition.

Partition key arguments must not evaluate to `NULL` and can be any of the following.

* Column names, as shown below.
  ```sql
  PARTITION BY date_column;
  PARTITION BY product_type;
  ```

* The result of the [EXTRACT](../../sql_reference/functions-reference/date-and-time/extract.md) function applied to a column of any of the date and time data types. For example ```PARTITION BY EXTRACT(MONTH FROM date_column);```.

* The result of the [DATE_TRUNC](../../sql_reference/functions-reference/date-and-time/date-trunc.md) function applied to a column of any of the date and time data types. For example ```PARTITION BY DATE_TRUNC('month', date_column);```.

* A composite key, with a mix of columns and `EXTRACT` functions, as shown below.
  ```PARTITION BY EXTRACT(MONTH FROM date_column), product_type;```

{: .caution}
Floating point data type columns are not supported as partition keys.

## Dropping partitions

Use the [ALTER TABLE](../../sql_reference/commands/data-definition/alter-table.md) statement to delete a partition and the data stored in that partition.

When you drop a partition created with a composite partition key, you must specify the full partition key. Dropping based on a subset of a composite key is not supported. See the example [Partition and drop by composite key](#partition-and-drop-by-composite-key) below.

```ALTER TABLE <table_name> DROP PARTITION 12,34;```

{: .warning}
Dropping a partition deletes the partition and the data stored in that partition.

### Examples

- [Working with partitions](#working-with-partitions)
  - [When to use partitions](#when-to-use-partitions)
  - [Considerations for partitioning and query performance](#considerations-for-partitioning-and-query-performance)
    - [Partition only large tables](#partition-only-large-tables)
    - [Large, equally distributed partitions work best](#large-equally-distributed-partitions-work-best)
  - [Defining partition keys](#defining-partition-keys)
  - [Dropping partitions](#dropping-partitions)
    - [Examples](#examples)
      - [Partition and drop by date](#partition-and-drop-by-date)
      - [Partition and drop by date extraction](#partition-and-drop-by-date-extraction)
      - [Partition and drop by integer](#partition-and-drop-by-integer)
      - [Partition and drop by composite key](#partition-and-drop-by-composite-key)

The examples in this section are based on the following common `CREATE TABLE` example. Each example is based on the addition of the `PARTITION BY` statement shown.

```sql
CREATE FACT TABLE game_information
(
    gameid INTEGER,
    title TEXT,
    abbreviation TEXT,
    launchdate DATE
)
PRIMARY INDEX gameid, title
<examples of PARTITION BY clauses below>
```

#### Partition and drop by date
{: .no_toc}

The example below creates a partition for each group of records with the same date value in `transaction_date`.

```sql
PARTITION BY transaction_date
```

The example below drops the partition for records with the date `2020-01-01`. The date is provided as a string literal and must be cast to the `DATE` data type in the command. The command uses the [:: operator for CAST](../../sql_reference/operators.md#-type-cast).

```sql
ALTER TABLE fct_tbl_transactions DROP PARTITION '2020-01-01'::DATE;
```

#### Partition and drop by date extraction
{: .no_toc}

The example below uses `EXTRACT` to create a partition for each group of records with the same year value in `transaction_date`.

```sql
PARTITION BY EXTRACT(YEAR FROM transaction_date), EXTRACT(MONTH FROM transaction_date);
```

The example below drops the partition for records where `transaction_date` is in the month of April 2022. The year and month are specified as integers in the command.

```sql
ALTER TABLE fct_tbl_transactions DROP PARTITION 2022,04;
```

#### Partition and drop by integer
{: .no_toc}

The example below creates a partition for each group of records with the same value for `gameid`.

```sql
PARTITION BY gameid
```

The example below drops the partition where `gameid` is `8188`.

```sql
ALTER TABLE games DROP PARTITION 8188;
```

#### Partition and drop by composite key
{: .no_toc}

The example below creates a partition for each group of records where `gameid` is the same value **and** `transaction_date` is the same year.

```sql
PARTITION BY gameid,EXTRACT(YEAR FROM transaction_date);
```

The example below drops the partition where `gameid` is `982` **and** `transaction_date` is `2020` .

```sql
ALTER TABLE games DROP PARTITION 982,2020;
```
