# Code Efficiency Checklist

Use this checklist when writing and reviewing code to ensure optimal performance.

## Algorithm & Data Structure Efficiency

### Time Complexity
- [ ] Algorithm has appropriate time complexity for use case
- [ ] No nested loops where a single pass would work
- [ ] Using hash maps/sets for O(1) lookups instead of arrays for O(n) searches
- [ ] Consider sorting once and using binary search instead of multiple linear searches
- [ ] Recursive algorithms have base cases to prevent stack overflow

### Space Complexity
- [ ] Not storing unnecessary intermediate data
- [ ] Using generators/iterators for large datasets instead of loading all in memory
- [ ] Clearing large collections when no longer needed
- [ ] Using appropriate data structures (don't use HashMap for 2-3 items)

## Database Operations

### Query Optimization
- [ ] No N+1 query problems (use JOINs or batch queries)
- [ ] Selecting only needed columns, not using SELECT *
- [ ] Using appropriate indexes on frequently queried columns
- [ ] Using LIMIT for pagination instead of fetching all records
- [ ] Analyzing query plans with EXPLAIN
- [ ] Avoiding expensive operations in WHERE clauses (e.g., LIKE '%pattern%')

### Connection Management
- [ ] Using connection pooling
- [ ] Properly closing database connections
- [ ] Not opening new connections in loops
- [ ] Using prepared statements to prevent SQL injection and improve performance
- [ ] Batch inserts/updates instead of individual operations

### Caching
- [ ] Caching frequently accessed data
- [ ] Cache invalidation strategy in place
- [ ] Using appropriate TTL (Time To Live) values
- [ ] Not caching rapidly changing data

## I/O Operations

### File Operations
- [ ] Using buffered readers/writers
- [ ] Streaming large files instead of loading into memory
- [ ] Closing file handles properly
- [ ] Using asynchronous I/O where appropriate
- [ ] Not performing I/O in tight loops

### Network Requests
- [ ] Using connection pooling for HTTP clients
- [ ] Implementing timeouts to prevent hanging
- [ ] Using compression (gzip) for large payloads
- [ ] Batching API calls when possible
- [ ] Using async/await for non-blocking network calls
- [ ] Implementing retry logic with exponential backoff

## Memory Management

### Object Creation
- [ ] Not creating unnecessary objects in loops
- [ ] Reusing objects when possible
- [ ] Using object pooling for expensive objects
- [ ] Avoiding memory leaks (removing event listeners, clearing intervals)
- [ ] Using weak references where appropriate

### Collections
- [ ] Pre-sizing collections when size is known
- [ ] Using appropriate collection types (ArrayList vs LinkedList)
- [ ] Clearing large collections when done
- [ ] Not holding references to large objects longer than needed

## Concurrency & Parallelism

### Thread Safety
- [ ] Minimizing synchronized blocks
- [ ] Using concurrent collections (ConcurrentHashMap, etc.)
- [ ] Avoiding deadlocks with proper lock ordering
- [ ] Using thread pools instead of creating threads manually
- [ ] Not blocking the main/UI thread

### Asynchronous Processing
- [ ] Using async/await for I/O bound operations
- [ ] Using message queues for long-running tasks
- [ ] Implementing proper error handling in async code
- [ ] Not mixing sync and async code inappropriately

## String Operations

- [ ] Using StringBuilder/StringBuffer for concatenation in loops
- [ ] Using string formatting (f-strings, template literals) appropriately
- [ ] Not using regex when simple string operations suffice
- [ ] Using string interning for frequently repeated strings (when appropriate)

## Loop Optimization

- [ ] Moving invariant calculations outside loops
- [ ] Using appropriate loop type (for, while, forEach)
- [ ] Breaking early when possible
- [ ] Using loop unrolling for critical hot paths (when beneficial)
- [ ] Avoiding unnecessary iterations

## Function Calls

- [ ] Minimizing function call overhead in hot paths
- [ ] Using inline functions for small, frequently called operations
- [ ] Avoiding recursion where iteration is simpler
- [ ] Memoizing expensive function results
- [ ] Not passing large objects by value unnecessarily

## Serialization & Parsing

- [ ] Using efficient serialization formats (Protocol Buffers, MessagePack vs JSON)
- [ ] Streaming parsers for large documents (SAX vs DOM for XML)
- [ ] Not serializing unnecessary data
- [ ] Using binary formats for internal communication

## Frontend/UI Performance

### Rendering
- [ ] Minimizing DOM manipulations
- [ ] Using virtual scrolling for large lists
- [ ] Debouncing/throttling event handlers
- [ ] Using requestAnimationFrame for animations
- [ ] Lazy loading images and components

### Asset Optimization
- [ ] Minifying JavaScript and CSS
- [ ] Using code splitting
- [ ] Compressing images (WebP, AVIF)
- [ ] Using CDN for static assets
- [ ] Implementing browser caching with proper headers

### State Management
- [ ] Avoiding unnecessary re-renders
- [ ] Using memoization (React.memo, useMemo, useCallback)
- [ ] Normalizing state shape
- [ ] Using selectors to avoid derived state recalculation

## API Design

- [ ] Using pagination for large result sets
- [ ] Supporting field filtering (allowing clients to request only needed fields)
- [ ] Implementing rate limiting
- [ ] Using efficient data formats
- [ ] Supporting compression
- [ ] Implementing proper caching headers
- [ ] Using GraphQL or similar for flexible queries (when appropriate)

## Logging & Monitoring

- [ ] Not logging in tight loops
- [ ] Using appropriate log levels
- [ ] Sampling high-frequency logs
- [ ] Asynchronous logging
- [ ] Not serializing large objects just for logging

## Testing Performance

- [ ] Writing performance tests for critical paths
- [ ] Setting performance budgets
- [ ] Monitoring performance metrics in CI/CD
- [ ] Load testing before production deployment
- [ ] Profiling regularly to identify bottlenecks

## Common Anti-Patterns to Avoid

- [ ] No premature optimization (measure first)
- [ ] No copying large collections unnecessarily
- [ ] Not ignoring compiler/interpreter optimizations
- [ ] No string concatenation in loops (use join/StringBuilder)
- [ ] Not using inappropriate data structures
- [ ] No excessive use of reflection
- [ ] Not using regex for simple string operations
- [ ] No blocking operations in async contexts
- [ ] Not creating threads for every task
- [ ] No unbounded caches (implement eviction)

## Language-Specific Checks

### Python
- [ ] Using list comprehensions instead of loops where appropriate
- [ ] Using built-in functions (sum, max, min, etc.)
- [ ] Using generators for large sequences
- [ ] Not using mutable default arguments
- [ ] Using slots for classes with fixed attributes

### JavaScript/TypeScript
- [ ] Using const/let instead of var
- [ ] Avoiding global variables
- [ ] Using Map/Set instead of objects for lookups
- [ ] Using Web Workers for CPU-intensive tasks
- [ ] Implementing proper cleanup in useEffect (React)

### Java
- [ ] Using primitive types when possible
- [ ] Using StringBuilder for string concatenation
- [ ] Implementing equals() and hashCode() properly
- [ ] Using try-with-resources for auto-closing
- [ ] Sizing collections appropriately (new ArrayList<>(expectedSize))

### Go
- [ ] Using goroutines and channels appropriately
- [ ] Preallocating slices when size is known
- [ ] Using sync.Pool for frequently allocated objects
- [ ] Avoiding defer in tight loops
- [ ] Using bufio for I/O operations

### C#
- [ ] Using StringBuilder for string concatenation
- [ ] Implementing IDisposable for resource cleanup
- [ ] Using async/await properly
- [ ] Using struct for small, immutable types
- [ ] Avoiding boxing/unboxing with generics

## Profiling Checklist

Before optimizing:
- [ ] Profile the code to identify actual bottlenecks
- [ ] Measure baseline performance
- [ ] Set performance goals
- [ ] Focus on hot paths (code that runs most frequently)

After optimizing:
- [ ] Re-profile to verify improvements
- [ ] Ensure code is still correct
- [ ] Ensure code is still maintainable
- [ ] Document non-obvious optimizations

## Quick Wins

These optimizations often provide good returns with minimal effort:

1. **Add database indexes** on frequently queried columns
2. **Implement caching** for expensive operations
3. **Use connection pooling** for databases and HTTP clients
4. **Enable compression** for API responses
5. **Add pagination** for large result sets
6. **Lazy load** resources that aren't immediately needed
7. **Batch database operations** instead of individual queries
8. **Use CDN** for static assets
9. **Minimize JavaScript bundle size** with code splitting
10. **Use async I/O** instead of blocking operations

## Remember

> "Premature optimization is the root of all evil" - Donald Knuth

- Optimize only after measuring
- Focus on algorithmic improvements first
- Keep code readable and maintainable
- Document complex optimizations
- Have tests before and after optimization

## Next Steps

1. Review your code against this checklist
2. Profile your application to find bottlenecks
3. Prioritize optimizations based on impact
4. Implement changes incrementally
5. Measure improvements
6. Document learnings
