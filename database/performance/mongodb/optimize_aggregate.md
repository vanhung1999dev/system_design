# MongoDB Aggregation Pipeline Optimization

## 1. Use Indexes Effectively

- Ensure that queries in `$match` use indexed fields.
- Avoid unnecessary full collection scans.
- Use **compound indexes** where appropriate.

## 2. Place `$match` and `$sort` Early

- Filtering (`$match`) should be at the beginning to reduce document load.
- Sorting (`$sort`) should leverage indexes and be done as early as possible.

## 3. Use `$project` to Reduce Document Size

- Remove unused fields early with `$project` to reduce memory footprint.

## 4. Avoid `$unwind` When Possible

- If only counting array elements, use `$size` instead.
- Use `$arrayElemAt` when accessing specific elements.

## 5. Optimize `$group` Performance

- Reduce documents before grouping using `$match` and `$project`.
- Use `$mergeObjects` instead of `$push` where applicable.

## 6. Use `$facet` for Parallel Processing

- `$facet` allows running multiple aggregation paths simultaneously.
- Useful for handling different aspects of data analysis in one query.

Example:

```json
{
  "$facet": {
    "total": [{ "$count": "totalCount" }],
    "topItems": [{ "$sort": { "score": -1 } }, { "$limit": 5 }]
  }
}
```

## 7. Use `$lookup` Efficiently

- Prefer indexed fields in lookup operations.
- If possible, denormalize data to avoid excessive lookups.

## 8. Use `$merge` Instead of `$out` for Storing Results

- `$merge` allows writing aggregation results to a collection without overwriting all data.

## 9. Monitor Query Performance

- Use `db.collection.explain("executionStats")` to analyze performance bottlenecks.
- Identify and optimize slow stages.

## 10. Limit In-Memory Operations

- Ensure aggregation stays within `allowDiskUse` constraints.
- Use `$bucket` instead of `$group` for histogram-style grouping.

## 11. Limit data when return (Pagination)

- Do not return a large data when calculate

Following these best practices will enhance the efficiency and performance of MongoDB aggregation pipelines.
