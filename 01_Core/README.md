# Java Core

## Purpose

Folder này chứa nền tảng Java cần dùng thường xuyên trước khi đi sâu vào JVM internals, framework, performance, hoặc concurrency nâng cao.

Mục tiêu là giúp người học hiểu concept đủ chắc để đọc code thật, viết API nhỏ, debug bug phổ biến, và làm exercise skeleton khi cần.

## Suggested learning path

1. `Types-and-Variables/`
2. `Control-Flow/`
3. `Object-Methods/`
4. `Data-Types-Advanced/`
5. `Exception/`
6. `Collections/`
7. `Generics/`
8. `Functional/`
9. `IO/`
10. `Memory/`
11. `Reflection-and-Annotation/`
12. `Multithreading/`

## Folder map

- `Types-and-Variables/`: primitive, wrapper, `String`, `null`, casting, equality/hashCode.
- `Control-Flow/`: loop, switch, recursion, return/break/continue.
- `Object-Methods/`: `toString`, `equals`, `hashCode`, clone/finalize/Cleaner.
- `Data-Types-Advanced/`: enum, record, sealed class, pattern matching.
- `Exception/`: checked/unchecked, try-catch-finally, try-with-resources, custom exception.
- `Collections/`: List/Set/Map, Iterator, HashMap, queue/deque/stack, collection choices.
- `Generics/`: type parameter, wildcard, erasure, generic method/class.
- `Functional/`: lambda, functional interface, Stream API, Optional, collectors.
- `IO/`: stream/reader, buffering, NIO, serialization, `Path`/`Files`.
- `Memory/`: GC, leak, string pool, copy semantics, reference types, off-heap.
- `Reflection-and-Annotation/`: reflection, annotation, proxy, annotation processor.
- `Multithreading/`: thread/executor, race condition, synchronization, volatile, atomic, deadlock, thread pool, CompletableFuture.

## How to use this folder

- Đọc theo learning path nếu đang xây nền.
- Khi gặp bug thật, đi thẳng vào folder concept liên quan rồi quay lại mental model nếu thấy mơ hồ.
- Từ `Collections` trở đi, note nên có exercises theo guideline.

## Related notes

[[01-programming-languages/java/00_Mental-Models/README]]

[[../note-and-exercise-guidelines]]
