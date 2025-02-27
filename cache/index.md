# What is cache

![](./images/2025-02-27_21-02.png)

# What it not to cache

![](./images/2025-02-27_21-05.png)

# Distribute cache

![](./images/2025-02-27_21-06.png)

- at least 3 server
- `in system design, only need to draw one box -> indicate that is cluster`

# Caching strategy

## Cache-aside (Lazy load - Read-Query)

![](./images/2025-02-27_21-09.png)

## Write-around

![](./images/2025-02-27_21-13.png)

### with option clear-cache (Prefer => slow but correct)

- what happen when we clear-cache
  - user get from catch (miss cache)-> need to get from DB -> then update to the cache => `take one step to write to cache`

### with update-cache (incorrect but faster)

- `it make a inconsistency data because 2 step don't have atomic and place in different database`
- example
  - Client 1 write v1 to DB
  - Client 2 write v2 to DB
  - Client 2 write v2to Cache
  - Client 2 write v1to Cache
    => now in DB, value is `v2`, but in Cache is `v1`

## Use Cache Aside + Write Around is the best way

# Cache Invalidation (with active delete, need config policy in redis)

![](./images/2025-02-27_21-27.png)

# Cache Eviction (Only need to setup policy in Redis)

![](./images/2025-02-27_21-31.png)

# Cache Problem

## Cache Penetration

![](./images/2025-02-27_21-33.png)

### Solution

- can prevent it from gateway, use rare-limiter

![](./images/2025-02-27_21-39.png)

## Stale Set

![](./images/2025-02-27_21-41.png)

## Thundering herd (Aka Cache stampede) Multi request comme

![](./images/2025-02-27_21-43.png)

## Solution for Stale Set and Thundering herd

![](./images/2025-02-27_21-50.png)

<br>

![](./images/2025-02-27_21-51.png)

## Data inconsistency when update data (in Write Around)

![](./images/2025-02-27_21-54.png)

### Imperfect Solution

![](./images/2025-02-27_21-57.png)

- Binlog (file log of DB)

<br>

![](./images/2025-02-27_21-59.png)

## Wrong Cache

![](./images/2025-02-27_22-03.png)

<br>

![](./images/2025-02-27_22-05.png)

<br>

![](./images/2025-02-27_22-07.png)

# Trade-off

1. Cost vs Latency
   - Store x% in cache(20%)
   - TTL
2. Consistency and System Complexity
3. Date Freshness vs Latency
   - TTL
