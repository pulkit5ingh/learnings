# 07 — Phase 6: Caching and Performance Optimization

> **Week 7** · Reduce latency and database load.

## Checklist

**1. Caching layers**

- [ ] Client-side caching
- [ ] Browser and HTTP caching
- [ ] CDN caching
- [ ] Reverse-proxy and web-server caching
- [ ] Application caching
- [ ] Database and query caching

**2. Caching strategies**

- [ ] Cache-aside
- [ ] Read-through
- [ ] Write-through
- [ ] Write-behind
- [ ] Refresh-ahead

**3. Cache failure modes**

- [ ] Cache invalidation
- [ ] TTL and eviction policies
- [ ] LRU and LFU
- [ ] Cache stampede
- [ ] Cache penetration
- [ ] Hot keys
- [ ] Thundering herd
- [ ] Cache warming
- [ ] Distributed caching and Redis clustering

**4. Performance antipatterns**

- [ ] Busy database
- [ ] Busy frontend
- [ ] Chatty I/O
- [ ] Extraneous fetching
- [ ] Improper instantiation
- [ ] Monolithic persistence
- [ ] No caching
- [ ] Noisy neighbor
- [ ] Synchronous I/O
- [ ] Retry storms

## 1. Caching layers

### Client-side caching

- **Notes:**

### Browser and HTTP caching

- **Notes:**

### CDN caching

- **Notes:**

### Reverse-proxy and web-server caching

- **Notes:**

### Application caching

- **Notes:**

### Database and query caching

- **Notes:**

## 2. Caching strategies

| Strategy | How it works |
| --- | --- |
| Cache-aside | Application loads missing data into cache |
| Read-through | Cache loads data through its configured loader |
| Write-through | Writes update cache and backing storage together |
| Write-behind | Cache accepts writes and persists them asynchronously |
| Refresh-ahead | Cache refreshes entries before expiration |

### Cache-aside

- **Notes:**

### Read-through

- **Notes:**

### Write-through

- **Notes:**

### Write-behind

- **Notes:**

### Refresh-ahead

- **Notes:**

## 3. Cache failure modes

### Cache invalidation

- **Notes:**

### TTL and eviction policies

- **Notes:**

### LRU and LFU

- **Notes:**

### Cache stampede

- **Notes:**

### Cache penetration

- **Notes:**

### Hot keys

- **Notes:**

### Thundering herd

- **Notes:**

### Cache warming

- **Notes:**

### Distributed caching and Redis clustering

- **Notes:**

## 4. Performance antipatterns

### Busy database

- **Notes:**

### Busy frontend

- **Notes:**

### Chatty I/O

- **Notes:**

### Extraneous fetching

- **Notes:**

### Improper instantiation

- **Notes:**

### Monolithic persistence

- **Notes:**

### No caching

- **Notes:**

### Noisy neighbor

- **Notes:**

### Synchronous I/O

- **Notes:**

### Retry storms

- **Notes:**

## Practice

**Exercise:** Optimize a slow product listing API using Redis, proper database indexes, pagination and CDN caching.

- **Requirements:**
- **Scale estimation:**
- **APIs:**
- **Data model:**
- **High-level architecture:**
- **Bottlenecks & deep dive:**
- **Failure handling:**
- **Trade-offs:**
