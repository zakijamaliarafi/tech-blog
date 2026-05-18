---
heroImage: '/postgresql-performance-tuning-linux.svg'
title: 'PostgreSQL Performance Tuning on Linux'
description: 'Optimize your PostgreSQL database configuration for maximum performance on Linux environments.'
pubDate: 'May 13 2026'
---

PostgreSQL ships with a very conservative default configuration designed to run on systems with minimal resources. To unlock its full potential on dedicated server hardware, comprehensive tuning is required.

## Essential postgresql.conf Parameters

### 1. shared_buffers
This parameter determines how much memory PostgreSQL uses for caching data. A common recommendation is setting it to 25% of the total system RAM.
```ini
shared_buffers = 4GB # Example for a 16GB RAM server
```

### 2. work_mem
This sets the maximum amount of memory to be used by query operations (e.g., sorts, hash tables) before writing to temporary disk files. If you frequently perform complex sorts, increasing this can dramatically improve query performance.
```ini
work_mem = 16MB
```
*Note: This memory can be allocated multiple times per query, so be cautious not to set it too high.*

### 3. maintenance_work_mem
This dictates the maximum memory used for maintenance operations like `VACUUM`, `CREATE INDEX`, and `ALTER TABLE`.
```ini
maintenance_work_mem = 512MB
```

### 4. effective_cache_size
This parameter tells the PostgreSQL query planner how much memory is estimated to be available for disk caching by the operating system. It does not allocate memory; it simply informs the planner to prefer index scans over sequential scans if enough memory is likely available. Set it to roughly 50% to 75% of total RAM.
```ini
effective_cache_size = 12GB
```

## Linux OS Level Tuning

Database tuning isn't just about `postgresql.conf`. The underlying OS also matters.

### 1. Disable Transparent Huge Pages (THP)
THP can cause significant performance degradation for PostgreSQL. Disable it permanently via grub or systemd.

### 2. Adjust sysctl parameters
Tuning shared memory and semaphore settings in `/etc/sysctl.conf` can ensure PostgreSQL has sufficient OS resources.
```ini
kernel.shmmax = 17179869184
kernel.shmall = 4194304
```

Regularly analyzing slow queries using `pg_stat_statements` and indexing appropriately will further complement these systemic tuning efforts.

