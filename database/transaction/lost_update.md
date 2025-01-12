# Problem

- how to handle multi transaction in the same time, and may be it lead to **race condition** and **lost update**

## 1-Atomic

```
SELECT quantity FROM inventory WHERE id = 123; // quantity = 1
// logic check quantity > 0
UPDATE inventory
SET quantity = quantity - 1
WHERE id = 123
```

instead of this, we use _anomic_

```
UPDATE inventory
SET quantity = quantity - 1
WHERE id = 123 AND quantity > 0;

-- Check số rows affected
SELECT ROW_COUNT();

```

<br>

## 2-Isolation level

- Can use **Repeatable and Serializable**

<br>

## 3- Pessimistic lock

```
BEGIN;
SELECT * FROM inventory
WHERE id = 123 FOR UPDATE;  -- Đặt lock cho row này

UPDATE inventory
SET quantity = quantity - 1
WHERE id = 123;
COMMIT;
```

- NOTE: need to be consider about **Deadlock** and not suit for **Distribute system**

<br>

## 4-Optimistic lock

```
-- get current version of item
_version := SELECT version FROM inventory WHERE id = 123;

UPDATE inventory  -- update with current version
SET quantity = quantity - 1
WHERE id = 123
AND version = _version;

```

- NOTE: should apply with system which accept **retry and small conflict**

<br>

## 5-Distribute lock

- It will same with **pessimistic lock**, acquire lock first, then update
- You can use **Database base locking** prefer user **Redis**.

<br>

## 6-Queue

- use **FIFO** first in first out
- NOTE: time for waiting may be so big

## 7-Reserved Counter

- step1: init counter === total item in database
- step2: when user make update => update in counter(can be in redis)
- step3: when user make payment => update in database
