---
layout: post
title:  "Spark collect_list vs collect_set"
date:   2017-10-23 08:22:00 +0200
# categories: spark
# tags: collect_list collect_set optimization spark groupby dataframe
---

I was wondering. Which one is faster: `collect_list` or `collect_set`? If you need your data in order and want to keep those precious duplicates, then `collect_list` is for you. If you don't care about that (for example you are sure that you won't have duplicata) you may wonder: **which `collect_*` would be faster?**

I have a complex `dataframe` in which I perform several aggregations. Most of them are some kind of `groupBy([x], collect_list([y]))`. I do it repeatedly in order to ge to something like:

```
root
 |-- CompanyCode: string (nullable = true)
 |-- Source: string (nullable = true)
 |-- Destination: string (nullable = true)
 |-- ProductionDate: integer (nullable = false)
 |-- EndDate: integer (nullable = false)
 |-- PackageSeq: array (nullable = true)
 |    |-- element: struct (containsNull = true)
 |    |    |-- PackageName: string (nullable = true)
 |    |    |-- PackageDestination: string (nullable = true)
 |    |    |-- ShippingDate: integer (nullable = false)
 |    |    |-- ArrivalDate: integer (nullable = false)
 |    |    |-- Vector: string (nullable = true)
 |    |    |-- SalesSeq: array (nullable = true)
 |    |    |    |-- element: struct (containsNull = true)
 |    |    |    |    |-- StartSaleDate: string (nullable = false)
 |    |    |    |    |-- EndSaleDate: string (nullable = false)
 |    |    |    |    |-- MarketSeq: array (nullable = true)
 |    |    |    |    |    |-- element: struct (containsNull = true)
 |    |    |    |    |    |    |-- Market: string (nullable = true)
 |    |    |    |    |    |    |-- ValueSeq: array (nullable = true)
 |    |    |    |    |    |    |    |-- element: struct (containsNull = true)
 |    |    |    |    |    |    |    |    |-- Code: string (nullable = true)
 |    |    |    |    |    |    |    |    |-- Value: string (nullable = true)
```

| Command            | Time `collect_list`                 | time `collect_set`                  |
| ------------------ | ----------------------------------- | ----------------------------------- |
| `df.count`         | `22s (10s +  5s +  6s +  1s +  0s)` | `29s (11s +  9s +  7s +  1s +  0s)` |
| `df.write.parquet` | `67s (35s + 12s +  5s +  9s +  6s)` | `73s (35s + 14s +  5s + 10s + 10s)` |
| `df.show(100)`     | `58s (33s + 12s +  4s +  9s +  0s)` | `60s (36s + 11s +  4s +  8s +  0s)` |

When you read `0s` it's actually a few negligible milliseconds.

The first stage is the read from the `parquet` files that I had previously prepared to do this test. The input `parquet` was also filtered to reduce the amount of input data: in total I had `627022` rows in input. By the end of the process I had `176382` rows. If you ignore that first stage, as it does not contain any `groupBy`, you will have:
| Command            | Time `collect_list` (no read time) | time `collect_set` (no read time) |
| ------------------ | ---------------------------------- | --------------------------------- |
| `df.count`         | `12s`                              | `18s`                             |
| `df.write.parquet` | `32s`                              | `38s`                             |
| `df.show(100)`     | `25s`                              | `24s`                             |

The `collect_set` was a bit faster only when doing the `df.show(100)`. This is likely because of the `spark` optimisations: it's cheaper to get the first `x` elements if you don't care about order. If you care about order, then you have to order them all before getting the first 100 as the very last one in the input could be one of those *elected*.

**TLDR**: `collect_list` is faster than `collect_set`.

> **Update 2023-08-12**. This post has been imported from my previous neglected blog.
{: .prompt-info}
