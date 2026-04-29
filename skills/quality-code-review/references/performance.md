# Performance

- Flag obvious N+1 query patterns (queries inside loops).
- Flag missing `key` props or unstable keys (`Math.random()`, array index on reorderable lists).
- Flag large objects or arrays created inside render/component body that should be extracted or memoized.
- Flag synchronous heavy computation that blocks the main thread.
