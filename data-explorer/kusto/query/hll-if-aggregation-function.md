---
title:  hll_if() (aggregation function)
description: Learn how to use the hll_if() function to calculate the intermediate results of the dcount() function.
ms.reviewer: ziham
ms.topic: reference
ms.date: 02/02/2025
---
# hll_if() (aggregation function)

> [!INCLUDE [applies](../includes/applies-to-version/applies.md)] [!INCLUDE [fabric](../includes/applies-to-version/fabric.md)] [!INCLUDE [azure-data-explorer](../includes/applies-to-version/azure-data-explorer.md)] [!INCLUDE [monitor](../includes/applies-to-version/monitor.md)] [!INCLUDE [sentinel](../includes/applies-to-version/sentinel.md)]

Calculates the intermediate results of [`dcount`](dcount-aggregation-function.md) in records for which the *predicate* evaluates to `true`.

Read about the [underlying algorithm (*H*yper*L*og*L*og) and the estimation accuracy](dcount-aggregation-function.md#estimation-accuracy).

[!INCLUDE [data-explorer-agg-function-summarize-note](../includes/agg-function-summarize-note.md)]

## Syntax

`hll_if` `(`*expr*, *predicate* [`,` *accuracy*]`)`

[!INCLUDE [syntax-conventions-note](../includes/syntax-conventions-note.md)]

## Parameters

| Name | Type | Required | Description |
|--|--|--|--|
| *expr* | `string` |  :heavy_check_mark: | The expression used for the aggregation calculation. |
| *predicate* | `string` |  :heavy_check_mark: | The *Expr* used to filter records to add to the intermediate result of `dcount`. |
| *accuracy* | `int` |   | The value that controls the balance between speed and accuracy. If unspecified, the default value is `1`. For supported values, see [Estimation accuracy](#estimation-accuracy). |

## Returns

Returns the intermediate results of distinct count of *Expr* for which *Predicate* evaluates to `true`.

> [!NOTE]
> - The results of hll(), hll_if(), and hll_merge() can be stored and later retrieved. For example, you might want to create a daily unique user summary, which can then be used to calculate weekly counts.
> However, the precise binary representation of these results might change over time. There's no guarantee that these functions produce identical results for identical inputs, and therefore we don't advise relying on them.
> - Use the [`hll_merge`](hll-merge-aggregation-function.md) function to merge more than one `hll` intermediate result. Only works with `hll` output.
> - Use [`dcount_hll`](dcount-hll-function.md), to calculate the distinct count from `hll`,`hll_merge`, or `hll_if` aggregation functions.

## Examples

The following query results in the number of unique flood event sources in Iowa and Kansas. It uses the `hll_if()` function to show only flood events.

:::moniker range="azure-data-explorer"
> [!div class="nextstepaction"]
> <a href="https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAAAz2OQQrCMBBF955imFUCuUIXXSiIYBcpuCw1ndBI2ylJqlQ8vDEBd/Ph/fdHR/bz8UlLDIcPvEbyBDr2kcAtIPDc3GpUgJf6qmuNMjFhm+feuzfBOE2dnZgHqPLtrNC8eUMKsrHdV4KqAjz9IJRw34s7WVbPDzKxZAWlFxqb0fJPsg6GtyV2SS7+Y/IL9SJmoLMAAAA=" target="_blank">Run the query</a>
::: moniker-end

```kusto
StormEvents
| where State in ("IOWA", "KANSAS")
| summarize hll_flood = hll_if(Source, EventType == "Flood") by State
| project State, SourcesOfFloodEvents = dcount_hll(hll_flood)
```

**Output**

|State|SourcesOfFloodEvents|
|---|---|
|KANSAS|11|
|IOWA|7|

## Estimation accuracy

| Accuracy | Speed | Error (%) |
|---|---|---|---|
| 0 | Fastest | 1.6 |
| 1 | Balanced | 0.8 |
| 2 | Slow | 0.4 |
| 3 | Slow | 0.28 |
| 4 | Slowest | 0.2 |

## Related content

* [Aggregation function types at a glance](aggregation-functions.md)
* [Using hll() and tdigest()](using-hll-tdigest.md)
* [hll() (aggregation function)](hll-aggregation-function.md)
* [hll_merge() (aggregation function)](hll-merge-aggregation-function.md)
