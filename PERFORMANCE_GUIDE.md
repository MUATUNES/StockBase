# Performance Optimization Guide

This guide provides best practices for identifying and fixing slow or inefficient code in your projects.

## Table of Contents
1. [Common Performance Anti-Patterns](#common-performance-anti-patterns)
2. [Performance Best Practices](#performance-best-practices)
3. [Profiling and Measurement](#profiling-and-measurement)
4. [Language-Specific Optimizations](#language-specific-optimizations)
5. [Database Optimization](#database-optimization)
6. [Caching Strategies](#caching-strategies)

## Common Performance Anti-Patterns

### 1. N+1 Query Problem
**Problem:** Making multiple database queries when one would suffice.

**Bad Example:**
```python
# Inefficient - N+1 queries
users = get_all_users()
for user in users:
    user.posts = get_posts_by_user_id(user.id)  # Separate query per user
```

**Good Example:**
```python
# Efficient - Single query with JOIN
users_with_posts = get_users_with_posts()  # Uses JOIN
```

### 2. Nested Loops with High Complexity
**Problem:** O(n²) or worse time complexity when O(n) is possible.

**Bad Example:**
```javascript
// O(n²) complexity
function findCommon(arr1, arr2) {
    const common = [];
    for (let i = 0; i < arr1.length; i++) {
        for (let j = 0; j < arr2.length; j++) {
            if (arr1[i] === arr2[j]) {
                common.push(arr1[i]);
            }
        }
    }
    return common;
}
```

**Good Example:**
```javascript
// O(n) complexity using Set
function findCommon(arr1, arr2) {
    const set1 = new Set(arr1);
    return arr2.filter(item => set1.has(item));
}
```

### 3. Unnecessary Memory Allocation
**Problem:** Creating objects/arrays repeatedly in loops.

**Bad Example:**
```java
// Creates new ArrayList in each iteration
for (int i = 0; i < 1000; i++) {
    List<String> temp = new ArrayList<>();
    temp.add("data");
    process(temp);
}
```

**Good Example:**
```java
// Reuse the same list
List<String> temp = new ArrayList<>();
for (int i = 0; i < 1000; i++) {
    temp.clear();
    temp.add("data");
    process(temp);
}
```

### 4. Synchronous I/O in Hot Paths
**Problem:** Blocking operations in frequently-called code.

**Bad Example:**
```python
def handle_request(user_id):
    user = db.query_sync(user_id)  # Blocks entire thread
    return process_user(user)
```

**Good Example:**
```python
async def handle_request(user_id):
    user = await db.query_async(user_id)  # Non-blocking
    return process_user(user)
```

### 5. String Concatenation in Loops
**Problem:** Repeatedly concatenating immutable strings.

**Bad Example:**
```python
# Inefficient - creates new string each iteration
result = ""
for item in items:
    result += str(item) + ","
```

**Good Example:**
```python
# Efficient - uses list and join
result = ",".join(str(item) for item in items)
```

## Performance Best Practices

### 1. Use Appropriate Data Structures
- **Arrays/Lists:** O(1) access, O(n) search
- **Hash Maps/Dictionaries:** O(1) average case for lookup
- **Sets:** O(1) for membership testing
- **Trees:** O(log n) for ordered operations

### 2. Lazy Loading
Load data only when needed, not eagerly.

```python
# Lazy loading
class Report:
    def __init__(self, report_id):
        self.report_id = report_id
        self._data = None
    
    @property
    def data(self):
        if self._data is None:
            self._data = fetch_large_dataset(self.report_id)
        return self._data
```

### 3. Batch Processing
Process items in batches instead of one-by-one.

```javascript
// Batch database inserts
async function saveUsers(users) {
    const BATCH_SIZE = 100;
    for (let i = 0; i < users.length; i += BATCH_SIZE) {
        const batch = users.slice(i, i + BATCH_SIZE);
        await db.batchInsert(batch);
    }
}
```

### 4. Avoid Premature Optimization
- Profile first, optimize second
- Focus on algorithmic improvements before micro-optimizations
- Maintain code readability

### 5. Use Streaming for Large Datasets
```python
# Stream large files instead of loading entirely into memory
def process_large_file(filename):
    with open(filename, 'r') as file:
        for line in file:  # Reads one line at a time
            process_line(line)
```

## Profiling and Measurement

### Tools by Language

**Python:**
```bash
# cProfile for profiling
python -m cProfile -o output.prof script.py

# memory_profiler for memory usage
@profile
def my_function():
    pass
```

**JavaScript/Node.js:**
```bash
# Built-in profiler
node --prof app.js

# Chrome DevTools for browser
# Performance tab with flame graphs
```

**Java:**
```bash
# JProfiler, VisualVM, or YourKit
# JMH for microbenchmarks
```

### Key Metrics to Monitor
1. **Response Time:** How long operations take
2. **Throughput:** Operations per second
3. **Memory Usage:** Heap size, allocation rate
4. **CPU Usage:** Processor utilization
5. **I/O Wait:** Time spent waiting for I/O

## Language-Specific Optimizations

### Python
```python
# Use list comprehensions instead of loops
# Bad
squares = []
for i in range(1000):
    squares.append(i * i)

# Good
squares = [i * i for i in range(1000)]

# Use built-in functions (implemented in C)
# Good
sum(numbers)
max(numbers)
sorted(items)
```

### JavaScript
```javascript
// Use const/let instead of var (better scoping)
// Avoid global variables
// Use requestAnimationFrame for animations
// Debounce/throttle event handlers

function debounce(func, wait) {
    let timeout;
    return function executedFunction(...args) {
        clearTimeout(timeout);
        timeout = setTimeout(() => func(...args), wait);
    };
}
```

### Java
```java
// Use StringBuilder for string concatenation
StringBuilder sb = new StringBuilder();
for (String s : strings) {
    sb.append(s);
}
String result = sb.toString();

// Use primitive types when possible
int count = 0; // Instead of Integer count = 0;

// Lazy initialization for expensive objects
private volatile ExpensiveObject instance;
public ExpensiveObject getInstance() {
    if (instance == null) {
        synchronized (this) {
            if (instance == null) {
                instance = new ExpensiveObject();
            }
        }
    }
    return instance;
}
```

## Database Optimization

### 1. Index Strategically
```sql
-- Create indexes on frequently queried columns
CREATE INDEX idx_user_email ON users(email);
CREATE INDEX idx_order_user_date ON orders(user_id, created_at);
```

### 2. Use EXPLAIN to Analyze Queries
```sql
EXPLAIN SELECT * FROM users WHERE email = 'test@example.com';
```

### 3. Avoid SELECT *
```sql
-- Bad
SELECT * FROM users;

-- Good (only fetch needed columns)
SELECT id, name, email FROM users;
```

### 4. Use Connection Pooling
```python
# Reuse database connections
from sqlalchemy import create_engine
from sqlalchemy.pool import QueuePool

engine = create_engine(
    'postgresql://user:pass@localhost/db',
    poolclass=QueuePool,
    pool_size=10,
    max_overflow=20
)
```

### 5. Pagination for Large Result Sets
```sql
-- Use LIMIT and OFFSET
SELECT id, name FROM users 
ORDER BY created_at DESC 
LIMIT 20 OFFSET 40;
```

## Caching Strategies

### 1. In-Memory Caching
```python
from functools import lru_cache

@lru_cache(maxsize=128)
def expensive_computation(n):
    # Computation result is cached
    return sum(i * i for i in range(n))
```

### 2. HTTP Caching
```javascript
// Set cache headers
res.set('Cache-Control', 'public, max-age=3600');
res.set('ETag', generateETag(data));
```

### 3. Database Query Caching
```python
# Redis for caching database queries
import redis

cache = redis.Redis(host='localhost', port=6379, db=0)

def get_user(user_id):
    cache_key = f"user:{user_id}"
    cached = cache.get(cache_key)
    
    if cached:
        return json.loads(cached)
    
    user = db.query(user_id)
    cache.setex(cache_key, 3600, json.dumps(user))
    return user
```

### 4. Memoization
```javascript
function memoize(fn) {
    const cache = new Map();
    return function(...args) {
        const key = JSON.stringify(args);
        if (cache.has(key)) {
            return cache.get(key);
        }
        const result = fn.apply(this, args);
        cache.set(key, result);
        return result;
    };
}

const fibonacci = memoize(function(n) {
    if (n <= 1) return n;
    return fibonacci(n - 1) + fibonacci(n - 2);
});
```

## Performance Testing

### Load Testing Tools
- **Apache JMeter:** General purpose load testing
- **k6:** Modern load testing for developers
- **Gatling:** Scala-based performance testing
- **Artillery:** Modern, powerful load testing

### Example k6 Load Test
```javascript
import http from 'k6/http';
import { check, sleep } from 'k6';

export let options = {
    stages: [
        { duration: '30s', target: 20 },
        { duration: '1m', target: 100 },
        { duration: '30s', target: 0 },
    ],
};

export default function() {
    let response = http.get('http://api.example.com/endpoint');
    check(response, {
        'status is 200': (r) => r.status === 200,
        'response time < 500ms': (r) => r.timings.duration < 500,
    });
    sleep(1);
}
```

## Checklist for Code Review

When reviewing code for performance, check for:

- [ ] Appropriate algorithm complexity (avoid O(n²) when O(n log n) is possible)
- [ ] No N+1 database query problems
- [ ] Efficient data structure usage
- [ ] Proper indexing on database queries
- [ ] Caching implemented where appropriate
- [ ] Lazy loading for expensive operations
- [ ] Batch processing for bulk operations
- [ ] No blocking I/O in hot paths
- [ ] Connection pooling for external resources
- [ ] Proper error handling that doesn't degrade performance
- [ ] Memory leaks prevention (proper cleanup of resources)
- [ ] Pagination for large datasets
- [ ] Compression for large payloads
- [ ] CDN usage for static assets
- [ ] Asynchronous processing for long-running tasks

## Resources

- [Web Performance Working Group](https://www.w3.org/webperf/)
- [Google Web Vitals](https://web.dev/vitals/)
- [Database Performance Tuning](https://use-the-index-luke.com/)
- [High Performance Browser Networking](https://hpbn.co/)

---

Remember: **Measure before optimizing.** Use profiling tools to identify actual bottlenecks rather than guessing.
