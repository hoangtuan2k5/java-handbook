# Multithreading

## Purpose

Folder này xây nền concurrency trong Java: thread, executor, race condition, synchronized, volatile, atomic classes, deadlock, thread pool, và CompletableFuture.

Mục tiêu là hiểu correctness trước performance: visibility, atomicity, ordering, shared mutable state, và lifecycle của task/thread.

## Suggested reading order

1. `001-thread-vs-runnable-vs-executor-service.md`
2. `002-race-condition.md`
3. `003-synchronized.md`
4. `004-volatile.md`
5. `005-atomic-classes.md`
6. `006-deadlock.md`
7. `007-thread-pool.md`
8. `008-completable-future.md`

## How to use this folder

- Khi data bị sai ngẫu nhiên, đọc race condition trước.
- Khi phân vân `volatile`, `synchronized`, hoặc atomic classes, đọc theo thứ tự `003` -> `004` -> `005`.
- Khi task async không chạy như mong đợi, đọc ThreadPool và CompletableFuture.

## Related notes

[[../../00_Mental-Models/003-Stack-vs-Heap]]

[[01-programming-languages/java/02_Advanced/Concurrency-Advanced/README]]
