# Advanced Concurrency

## Purpose

Folder này đi sâu hơn core multithreading: lock variants, synchronizers, concurrent collections internals, ForkJoinPool, reactive vs imperative, và virtual threads.

Mục tiêu là chọn concurrency primitive theo correctness và workload, không chỉ theo API quen thuộc.

## Suggested reading order

1. `001-reentrant-lock-vs-synchronized.md`
2. `002-read-write-lock.md`
3. `003-stamped-lock.md`
4. `004-countdown-latch-vs-cyclic-barrier.md`
5. `005-semaphore.md`
6. `006-concurrent-hash-map-internals.md`
7. `007-fork-join-pool.md`
8. `008-reactive-vs-imperative.md`
9. `009-virtual-threads.md`

## How to use this folder

- Khi `synchronized` chưa đủ linh hoạt, đọc ReentrantLock/ReadWriteLock/StampedLock.
- Khi cần phối hợp nhiều thread, đọc latch/barrier/semaphore.
- Khi dùng concurrent collection, đọc internals nhưng vẫn kiểm tra business-level atomicity.
- Khi chọn async model cho service, đọc reactive vs imperative và virtual threads.

## Related notes

[[01-programming-languages/java/01_Core/Multithreading/README]]

[[../JVM-Internals/007-memory-model-JMM]]
