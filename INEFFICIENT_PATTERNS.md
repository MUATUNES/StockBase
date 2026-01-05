# Inefficient Code Patterns and Solutions

This document provides real-world examples of inefficient code patterns and their optimized alternatives.

## Table of Contents
1. [Loop Optimizations](#loop-optimizations)
2. [Database Access Patterns](#database-access-patterns)
3. [Memory Management](#memory-management)
4. [String Operations](#string-operations)
5. [API and Network Calls](#api-and-network-calls)
6. [File I/O](#file-io)
7. [Caching Patterns](#caching-patterns)
8. [Frontend Performance](#frontend-performance)

---

## Loop Optimizations

### Example 1: Nested Loop with Hash Map Alternative

**❌ Inefficient (O(n×m) complexity):**
```javascript
function findMatchingUsers(users, activeUserIds) {
    const matches = [];
    for (let user of users) {
        for (let id of activeUserIds) {
            if (user.id === id) {
                matches.push(user);
                break;
            }
        }
    }
    return matches;
}

// Time: O(n×m) where n = users.length, m = activeUserIds.length
// For 1000 users and 1000 IDs: ~1,000,000 comparisons
```

**✅ Efficient (O(n+m) complexity):**
```javascript
function findMatchingUsers(users, activeUserIds) {
    const activeSet = new Set(activeUserIds);
    return users.filter(user => activeSet.has(user.id));
}

// Time: O(n+m)
// For 1000 users and 1000 IDs: ~2,000 operations
// ~500x faster!
```

### Example 2: Invariant Calculation in Loop

**❌ Inefficient:**
```python
def calculate_discounts(items, discount_rate):
    discounted_items = []
    for item in items:
        # Calculating same value repeatedly
        multiplier = 1 - (discount_rate / 100)
        discounted_price = item.price * multiplier
        discounted_items.append({
            'name': item.name,
            'original': item.price,
            'discounted': discounted_price
        })
    return discounted_items
```

**✅ Efficient:**
```python
def calculate_discounts(items, discount_rate):
    # Calculate once outside loop
    multiplier = 1 - (discount_rate / 100)
    return [
        {
            'name': item.name,
            'original': item.price,
            'discounted': item.price * multiplier
        }
        for item in items
    ]
```

### Example 3: Array Size Calculation in Loop Condition

**❌ Inefficient:**
```java
// Calls .size() on every iteration
for (int i = 0; i < list.size(); i++) {
    process(list.get(i));
}
```

**✅ Efficient:**
```java
// Calculate size once
int size = list.size();
for (int i = 0; i < size; i++) {
    process(list.get(i));
}

// Or use enhanced for loop
for (Item item : list) {
    process(item);
}
```

---

## Database Access Patterns

### Example 1: N+1 Query Problem

**❌ Inefficient:**
```python
# Makes 1 query for posts, then N queries for authors
def get_posts_with_authors():
    posts = db.query("SELECT * FROM posts LIMIT 100")
    
    for post in posts:
        # Additional query for each post!
        author = db.query(
            "SELECT * FROM users WHERE id = ?", 
            post.author_id
        )
        post.author = author
    
    return posts

# Total queries: 101 (1 + 100)
```

**✅ Efficient:**
```python
# Single query with JOIN
def get_posts_with_authors():
    posts = db.query("""
        SELECT 
            posts.*,
            users.name as author_name,
            users.email as author_email
        FROM posts
        JOIN users ON posts.author_id = users.id
        LIMIT 100
    """)
    
    return posts

# Total queries: 1
# ~100x faster!
```

### Example 2: Individual Inserts vs Batch Insert

**❌ Inefficient:**
```javascript
// Individual inserts
async function saveUsers(users) {
    for (const user of users) {
        await db.execute(
            'INSERT INTO users (name, email) VALUES (?, ?)',
            [user.name, user.email]
        );
    }
}

// For 1000 users: 1000 round trips to database
```

**✅ Efficient:**
```javascript
// Batch insert
async function saveUsers(users) {
    const BATCH_SIZE = 100;
    
    for (let i = 0; i < users.length; i += BATCH_SIZE) {
        const batch = users.slice(i, i + BATCH_SIZE);
        const values = batch.map(u => [u.name, u.email]);
        
        await db.batchInsert('users', ['name', 'email'], values);
    }
}

// For 1000 users: 10 round trips to database
// ~100x faster!
```

### Example 3: Unbounded Queries

**❌ Inefficient:**
```sql
-- Returns potentially millions of rows
SELECT * FROM orders WHERE status = 'pending';
```

**✅ Efficient:**
```sql
-- Use pagination
SELECT * FROM orders 
WHERE status = 'pending'
ORDER BY created_at DESC
LIMIT 100 OFFSET 0;

-- With cursor-based pagination (even better)
SELECT * FROM orders 
WHERE status = 'pending' 
  AND id > :last_seen_id
ORDER BY id
LIMIT 100;
```

---

## Memory Management

### Example 1: Loading Large Files into Memory

**❌ Inefficient:**
```python
def process_large_file(filename):
    # Loads entire file into memory
    with open(filename, 'r') as f:
        content = f.read()  # Could be GBs!
        lines = content.split('\n')
        
    for line in lines:
        process_line(line)
```

**✅ Efficient:**
```python
def process_large_file(filename):
    # Stream line by line
    with open(filename, 'r') as f:
        for line in f:  # Reads one line at a time
            process_line(line.strip())
```

### Example 2: Unnecessary Object Creation

**❌ Inefficient:**
```javascript
function formatDates(dates) {
    const results = [];
    for (let date of dates) {
        // Creates new Date object for each iteration
        const formatter = new Intl.DateTimeFormat('en-US', {
            year: 'numeric',
            month: 'long',
            day: 'numeric'
        });
        results.push(formatter.format(date));
    }
    return results;
}
```

**✅ Efficient:**
```javascript
function formatDates(dates) {
    // Create formatter once
    const formatter = new Intl.DateTimeFormat('en-US', {
        year: 'numeric',
        month: 'long',
        day: 'numeric'
    });
    
    return dates.map(date => formatter.format(date));
}
```

---

## String Operations

### Example 1: String Concatenation in Loops

**❌ Inefficient:**
```java
// Creates new String object on each concatenation
String result = "";
for (int i = 0; i < 1000; i++) {
    result += "Item " + i + ", ";
}

// Creates ~1000 intermediate String objects
```

**✅ Efficient:**
```java
// Mutable StringBuilder reuses same buffer
StringBuilder result = new StringBuilder();
for (int i = 0; i < 1000; i++) {
    result.append("Item ").append(i).append(", ");
}
String finalResult = result.toString();

// ~100x faster for large loops
```

### Example 2: Repeated String Operations

**❌ Inefficient:**
```python
def create_csv_row(items):
    row = ""
    for i, item in enumerate(items):
        row = row + str(item)
        if i < len(items) - 1:
            row = row + ","
    return row
```

**✅ Efficient:**
```python
def create_csv_row(items):
    return ",".join(str(item) for item in items)
```

---

## API and Network Calls

### Example 1: Sequential API Calls

**❌ Inefficient:**
```javascript
// Sequential - total time is sum of all requests
async function getUserData(userId) {
    const profile = await fetch(`/api/users/${userId}/profile`);
    const posts = await fetch(`/api/users/${userId}/posts`);
    const friends = await fetch(`/api/users/${userId}/friends`);
    
    return {
        profile: await profile.json(),
        posts: await posts.json(),
        friends: await friends.json()
    };
}

// If each request takes 100ms: total = 300ms
```

**✅ Efficient:**
```javascript
// Parallel - total time is max of all requests
async function getUserData(userId) {
    const [profileRes, postsRes, friendsRes] = await Promise.all([
        fetch(`/api/users/${userId}/profile`),
        fetch(`/api/users/${userId}/posts`),
        fetch(`/api/users/${userId}/friends`)
    ]);
    
    return {
        profile: await profileRes.json(),
        posts: await postsRes.json(),
        friends: await friendsRes.json()
    };
}

// If each request takes 100ms: total = 100ms
// 3x faster!
```

### Example 2: No Connection Reuse

**❌ Inefficient:**
```python
import requests

def fetch_multiple_endpoints(urls):
    results = []
    for url in urls:
        # Creates new connection each time
        response = requests.get(url)
        results.append(response.json())
    return results
```

**✅ Efficient:**
```python
import requests

def fetch_multiple_endpoints(urls):
    results = []
    # Reuse connection pool
    with requests.Session() as session:
        for url in urls:
            response = session.get(url)
            results.append(response.json())
    return results

# Significantly faster due to connection reuse
```

---

## File I/O

### Example 1: Unbuffered File Operations

**❌ Inefficient:**
```python
def copy_file(src, dst):
    with open(src, 'rb') as f_in:
        with open(dst, 'wb') as f_out:
            # Reading byte by byte is very slow
            byte = f_in.read(1)
            while byte:
                f_out.write(byte)
                byte = f_in.read(1)
```

**✅ Efficient:**
```python
def copy_file(src, dst):
    # Use buffered reading/writing
    BUFFER_SIZE = 64 * 1024  # 64KB
    
    with open(src, 'rb') as f_in:
        with open(dst, 'wb') as f_out:
            while True:
                chunk = f_in.read(BUFFER_SIZE)
                if not chunk:
                    break
                f_out.write(chunk)

# Or even simpler:
import shutil
shutil.copyfile(src, dst)
```

---

## Caching Patterns

### Example 1: No Caching for Expensive Operations

**❌ Inefficient:**
```javascript
function fibonacci(n) {
    if (n <= 1) return n;
    return fibonacci(n - 1) + fibonacci(n - 2);
}

// fibonacci(40) requires ~2 billion recursive calls!
// Extremely slow for n > 35
```

**✅ Efficient:**
```javascript
// Memoization
function fibonacci() {
    const cache = new Map();
    
    return function fib(n) {
        if (n <= 1) return n;
        
        if (cache.has(n)) {
            return cache.get(n);
        }
        
        const result = fib(n - 1) + fib(n - 2);
        cache.set(n, result);
        return result;
    };
}

const fib = fibonacci();
// fibonacci(40) now executes in microseconds!
```

### Example 2: Cache Without Expiration

**❌ Inefficient:**
```python
# Simple cache without expiration
cache = {}

def get_user_data(user_id):
    if user_id in cache:
        return cache[user_id]
    
    data = fetch_from_database(user_id)
    cache[user_id] = data  # Grows indefinitely!
    return data

# Memory leak - cache grows without bounds
```

**✅ Efficient:**
```python
from functools import lru_cache
from datetime import datetime, timedelta

# LRU cache with size limit
@lru_cache(maxsize=1000)
def get_user_data(user_id):
    return fetch_from_database(user_id)

# Or with TTL (Time To Live)
class TTLCache:
    def __init__(self, ttl_seconds=300):
        self.cache = {}
        self.ttl = ttl_seconds
    
    def get(self, key):
        if key in self.cache:
            value, timestamp = self.cache[key]
            if datetime.now() - timestamp < timedelta(seconds=self.ttl):
                return value
            del self.cache[key]
        return None
    
    def set(self, key, value):
        self.cache[key] = (value, datetime.now())

cache = TTLCache(ttl_seconds=300)
```

---

## Frontend Performance

### Example 1: Excessive DOM Manipulation

**❌ Inefficient:**
```javascript
// Causes multiple reflows and repaints
function addItems(items) {
    const list = document.getElementById('list');
    
    items.forEach(item => {
        const li = document.createElement('li');
        li.textContent = item.name;
        list.appendChild(li);  // DOM update on each iteration!
    });
}
```

**✅ Efficient:**
```javascript
// Batch DOM updates
function addItems(items) {
    const list = document.getElementById('list');
    const fragment = document.createDocumentFragment();
    
    items.forEach(item => {
        const li = document.createElement('li');
        li.textContent = item.name;
        fragment.appendChild(li);
    });
    
    list.appendChild(fragment);  // Single DOM update
}
```

### Example 2: No Event Delegation

**❌ Inefficient:**
```javascript
// Attaches event listener to each item
function setupButtons() {
    const buttons = document.querySelectorAll('.delete-btn');
    
    buttons.forEach(button => {
        button.addEventListener('click', function() {
            deleteItem(this.dataset.id);
        });
    });
}

// For 1000 buttons: 1000 event listeners
```

**✅ Efficient:**
```javascript
// Single event listener with delegation
function setupButtons() {
    document.getElementById('list').addEventListener('click', function(e) {
        if (e.target.classList.contains('delete-btn')) {
            deleteItem(e.target.dataset.id);
        }
    });
}

// Only 1 event listener regardless of number of buttons
```

### Example 3: Debouncing and Throttling

**❌ Inefficient:**
```javascript
// Fires on every keystroke
searchInput.addEventListener('input', function(e) {
    performSearch(e.target.value);  // Could fire 100s of times
});
```

**✅ Efficient:**
```javascript
// Debounce - waits for pause in typing
function debounce(func, wait) {
    let timeout;
    return function(...args) {
        clearTimeout(timeout);
        timeout = setTimeout(() => func.apply(this, args), wait);
    };
}

searchInput.addEventListener('input', debounce(function(e) {
    performSearch(e.target.value);
}, 300));  // Only searches 300ms after user stops typing
```

---

## Summary of Performance Gains

| Pattern | Inefficient Time | Efficient Time | Improvement |
|---------|-----------------|----------------|-------------|
| Nested loops → Hash map | O(n²) | O(n) | 100-1000x |
| N+1 queries → JOIN | 100+ queries | 1 query | 100x |
| Individual DB inserts → Batch | 1000 queries | 10 queries | 100x |
| String concat in loop | O(n²) | O(n) | 100x+ |
| Sequential API calls → Parallel | 300ms | 100ms | 3x |
| No memoization → Memoized fib(40) | Hours | Microseconds | 1,000,000x+ |

## Key Takeaways

1. **Use appropriate data structures** - Hash maps for lookups, not arrays
2. **Batch operations** - Database, API calls, DOM updates
3. **Cache expensive computations** - With appropriate invalidation
4. **Stream large datasets** - Don't load everything into memory
5. **Parallelize independent operations** - Use Promise.all, goroutines, etc.
6. **Profile before optimizing** - Measure to find real bottlenecks
7. **Move invariant calculations outside loops** - Don't recalculate the same value
8. **Use connection pooling** - For databases and HTTP clients
9. **Implement pagination** - Never return unbounded result sets
10. **Debounce/throttle high-frequency events** - Reduce unnecessary work

Remember: Always profile your code to identify actual bottlenecks before optimizing!
