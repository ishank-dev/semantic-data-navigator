---
layout: default
title: APACHE_DATASKETCHES_HLL_ESTIMATE
description: Reference material for APACHE_DATASKETCHES_HLL_ESTIMATE
great_grand_parent: SQL reference
grand_parent: SQL functions
parent: DataSketches functions
published: true
---

# HLL_COUNT_ESTIMATE

A scalar function.
Extracts a cardinality estimate of a
single [Apache DataSketches HLL sketches](https://datasketches.apache.org/docs/HLL/HLL.html) that was previously built
using the aggregate
function [APACHE_DATASKETCHES_HLL_BUILD](apache-datasketches-hll-build.md).

The `APACHE_DATASKETCHES_HLL_ESTIMATE` function is used to estimate the cardinality (number of unique elements) of a dataset represented by an HLL (HyperLogLog) sketch. This is particularly useful for large datasets where exact counting is computationally expensive.
## Syntax

{: .no_toc}

```sql
APACHE_DATASKETCHES_HLL_ESTIMATE
(<expression>)
```

## Parameters

{: .no_toc}

| Parameter      | Description                                                                                                                                                                                                    | Supported input types |
|:---------------|:---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|:----------------------|
| `<expression>` | An [Apache DataSketches HLL sketches](https://datasketches.apache.org/docs/HLL/HLL.html) in a valid format, e.g. the output of the [APACHE_DATASKETCHES_HLL_BUILD](apache-datasketches-hll-build.md) function. | `BYTEA`               |

## Return Type

`BIGINT`

## Error Handling

If the input expression is not a valid [Apache DataSketches HLL sketches](https://datasketches.apache.org/docs/HLL/HLL.html), the function will raise an error.
Ensure that the input is correctly formatted and generated by the [APACHE_DATASKETCHES_HLL_BUILD](apache-datasketches-hll-build.md) function.

## Example

{: .no_toc}

Following the [example](apache-datasketches-hll-build.md#example)
in [APACHE_DATASKETCHES_HLL_BUILD](apache-datasketches-hll-build.md):

```sql
SELECT APACHE_DATASKETCHES_HLL_ESTIMATE(a) AS estimate
FROM sketch_of_data_to_count
ORDER BY 1;
```

| estimate (BIGINT) |
|:------------------|
| 3333526           |
| 5001149           |

```sql
SELECT APACHE_DATASKETCHES_HLL_ESTIMATE(APACHE_DATASKETCHES_HLL_MERGE(a)) AS estimate
FROM sketch_of_data_to_count;
```

| hll_estimate (BIGINT) |
|:----------------------|
| 6673219               |
