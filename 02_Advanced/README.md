# Java Advanced

## Purpose

Folder này chứa các topic Java nâng cao: JVM internals, concurrency nâng cao, design patterns, performance, security, và testing.

Mục tiêu là nối core Java với production engineering: hiểu runtime, chọn abstraction đúng, đo performance thật, viết test tốt, và tránh lỗi bảo mật phổ biến.

## Suggested reading path

1. `JVM-Internals/`
2. `Concurrency-Advanced/`
3. `Design-Patterns/`
4. `Testing/`
5. `Performance/`
6. `Security/`

## Folder map

- `JVM-Internals/`: classloader, bytecode, JIT, GC, flags, JMM, heap dump.
- `Concurrency-Advanced/`: locks, latches/barriers, semaphore, ConcurrentHashMap internals, ForkJoinPool, reactive vs imperative, virtual threads.
- `Design-Patterns/`: creational, structural, behavioral patterns.
- `Testing/`: unit/integration test, JUnit 5, Mockito, TDD, coverage.
- `Performance/`: profiling, memory/CPU analysis, JMH, pitfalls.
- `Security/`: secure coding, cryptography, common vulnerabilities.

## How to use this folder

- Đừng đọc advanced topics như checklist thuộc lòng; đọc khi đã có use case hoặc bug cụ thể.
- Với JVM/performance/concurrency, luôn phân biệt mental model, spec guarantee, và implementation detail.
- Với design patterns, ưu tiên problem/force/trade-off hơn tên pattern.

## Related notes

[[01-programming-languages/java/01_Core/README]]

[[01-programming-languages/java/00_Mental-Models/README]]
