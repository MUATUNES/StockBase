# Quick Performance Reference

A one-page reference for the most impactful performance optimizations.

## 🚀 Top 10 Performance Wins

1. **Use Hash Maps for Lookups**
   ```javascript
   // ❌ O(n) - Slow
   array.find(item => item.id === searchId)
   
   // ✅ O(1) - Fast
   map.get(searchId)
   ```

2. **Avoid N+1 Database Queries**
   ```sql
   -- ❌ Multiple queries
   SELECT * FROM posts;
   -- Then for each post: SELECT * FROM users WHERE id = ?
   
   -- ✅ Single query with JOIN
   SELECT posts.*, users.name 
   FROM posts JOIN users ON posts.user_id = users.id;
   ```

3. **Batch Operations**
   ```python
   # ❌ Individual operations
   for item in items:
       db.insert(item)
   
   # ✅ Batch operation
   db.batch_insert(items)
   ```

4. **Use Appropriate Algorithms**
   - Sorting: O(n log n) vs O(n²)
   - Searching: Binary search O(log n) vs Linear O(n)
   - Hash table lookups: O(1) vs Array search O(n)

5. **Implement Caching**
   ```javascript
   const cache = new Map();
   
   function expensiveOperation(key) {
       if (cache.has(key)) return cache.get(key);
       
       const result = doExpensiveWork(key);
       cache.set(key, result);
       return result;
   }
   ```

6. **Stream Large Data**
   ```python
   # ❌ Load all into memory
   data = file.read()
   
   # ✅ Stream line by line
   for line in file:
       process(line)
   ```

7. **Parallelize Independent Operations**
   ```javascript
   // ❌ Sequential (300ms total)
   const a = await fetch(url1);
   const b = await fetch(url2);
   const c = await fetch(url3);
   
   // ✅ Parallel (100ms total)
   const [a, b, c] = await Promise.all([
       fetch(url1), fetch(url2), fetch(url3)
   ]);
   ```

8. **Add Database Indexes**
   ```sql
   CREATE INDEX idx_user_email ON users(email);
   CREATE INDEX idx_order_date ON orders(created_at);
   ```

9. **Use Connection Pooling**
   ```python
   # Reuse connections instead of creating new ones
   pool = ConnectionPool(size=10)
   conn = pool.get_connection()
   ```

10. **Paginate Large Results**
    ```sql
    SELECT * FROM products 
    ORDER BY id 
    LIMIT 20 OFFSET 0;
    ```

## ⚡ Time Complexity Cheat Sheet

| Complexity | Name | Example |
|-----------|------|---------|
| O(1) | Constant | Hash table lookup, array access |
| O(log n) | Logarithmic | Binary search, balanced tree operations |
| O(n) | Linear | Array iteration, linear search |
| O(n log n) | Linearithmic | Efficient sorting (merge, quick, heap) |
| O(n²) | Quadratic | Nested loops, bubble sort |
| O(2ⁿ) | Exponential | Naive fibonacci, subset generation |

**Target:** Keep hot paths at O(n) or better.

## 🎯 Quick Wins by Language

### Python
```python
# Use list comprehensions
[x*2 for x in items]  # Fast

# Use built-in functions
sum(items)  # Implemented in C
max(items)  # Much faster than loops

# Use generators for large sequences
(x*2 for x in range(million))  # Memory efficient
```

### JavaScript
```javascript
// Use const/let, not var
const items = [];

// Use Map/Set for lookups
const userMap = new Map();
userMap.get(id);  // O(1)

// Debounce expensive operations
const debouncedSearch = debounce(search, 300);
```

### Java
```java
// Use StringBuilder for concatenation
StringBuilder sb = new StringBuilder();
sb.append("text");

// Pre-size collections
ArrayList<String> list = new ArrayList<>(1000);

// Use primitives when possible
int count = 0;  // Not Integer
```

### SQL
```sql
-- Use indexes on WHERE/JOIN columns
CREATE INDEX idx_user_email ON users(email);

-- Select only needed columns
SELECT id, name FROM users;  -- Not SELECT *

-- Use EXPLAIN to check query plans
EXPLAIN SELECT * FROM users WHERE email = 'test@test.com';
```

## 🔍 Finding Bottlenecks

### Profile First
```bash
# Python
python -m cProfile script.py

# Node.js
node --prof app.js

# Chrome DevTools
Performance tab → Record → Analyze
```

### Measure Response Times
```javascript
console.time('operation');
performOperation();
console.timeEnd('operation');
```

### Monitor Key Metrics
- Response time (P50, P95, P99)
- Throughput (requests/sec)
- Memory usage
- CPU utilization
- Database query time

## 🚫 Common Mistakes

### ❌ String Concatenation in Loops
```python
result = ""
for item in items:
    result += str(item)  # Creates new string each time
```

### ❌ Nested Loops Instead of Hash Map
```javascript
for (let user of users) {
    for (let id of activeIds) {
        if (user.id === id) { ... }  // O(n²)
    }
}
```

### ❌ Loading Everything into Memory
```python
data = file.read()  # Could be GBs!
```

### ❌ No Pagination
```sql
SELECT * FROM users;  -- Could return millions
```

### ❌ Synchronous I/O in Loops
```javascript
for (let id of ids) {
    const data = await fetchData(id);  // Sequential
}
```

## ✅ Quick Fixes

1. **Slow database query?** → Add index, use EXPLAIN
2. **High memory usage?** → Use streaming, pagination
3. **Slow API endpoint?** → Add caching, optimize queries
4. **Slow frontend?** → Lazy load, code split, optimize images
5. **Slow loops?** → Use better algorithm, better data structure

## 📊 Performance Budgets

Set targets for your application:

- **Page Load:** < 3 seconds
- **API Response:** < 200ms (P95)
- **Database Query:** < 100ms (P95)
- **Bundle Size:** < 200KB gzipped
- **Time to Interactive:** < 5 seconds

## 🛠️ Tools

- **Profiling:** Chrome DevTools, cProfile, VisualVM
- **Load Testing:** k6, JMeter, Artillery
- **Monitoring:** Datadog, New Relic, Prometheus
- **Database:** EXPLAIN, slow query log
- **Frontend:** Lighthouse, WebPageTest

## 📚 Full Documentation

For detailed explanations and examples, see:
- [Performance Optimization Guide](PERFORMANCE_GUIDE.md)
- [Code Efficiency Checklist](CODE_EFFICIENCY_CHECKLIST.md)
- [Inefficient Code Patterns](INEFFICIENT_PATTERNS.md)

---

**Remember:** Profile before optimizing. Measure improvements. Keep code readable.
