# Memory

## Purpose

Folder này giải thích memory model ở mức core: GC, memory leak, string pool, shallow/deep copy, reference strengths, và off-heap memory.

Mục tiêu là debug lifecycle và memory issue bằng reachability, reference semantics, và JVM behavior thay vì đoán mò.

## Suggested reading order

1. `001-GC.md`
2. `002-Memory-leak.md`
3. `003-String-pool-vs-heap.md`
4. `004-shallow-copy-vs-deep-copy.md`
5. `005-strong-soft-weak-phantom-reference.md`
6. `006-off-heap-memory.md`

## How to use this folder

- Khi object không được thu hồi như mong đợi, đọc GC và memory leak.
- Khi copy object/collection bị side effect, đọc shallow/deep copy.
- Khi cache hoặc listener giữ object quá lâu, đọc reference strengths.

## Related notes

[[../../00_Mental-Models/003-Stack-vs-Heap]]

[[../../00_Mental-Models/008-Object-lifecycle]]
