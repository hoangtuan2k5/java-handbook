# Java Mental Models

## Purpose

Folder này xây mental model nền cho Java và JVM: source vs bytecode, reference value, object lifecycle, stack vs heap, class loading, immutability, và boundary giữa compile-time/runtime.

Mục tiêu là sửa các hiểu nhầm gốc trước khi học API chi tiết. Nếu mental model sai, các note core/advanced phía sau sẽ rất dễ bị hiểu lệch.

## Suggested reading order

1. `001-java-in-jvm-eyes.md`
2. `002-everything-is-object.md`
3. `003-stack-vs-heap.md`
4. `004-pass-by-value.md`
5. `011-value-type-vs-reference-type.md`
6. `009-class-vs-object.md`
7. `008-object-lifecycle.md`
8. `007-immutability.md`
9. `005-jvm-load-class.md`
10. `006-compile-time-vs-runtime.md`
11. `010-interface-vs-abstract.md`

## How to use this folder

- Khi gặp bug mutation hoặc aliasing, đọc `004` và `011` trước.
- Khi debug memory/lifecycle, đọc `003`, `008`, và `001` trước.
- Khi debug startup/static/Spring proxy behavior, đọc `005`, `006`, và `010` trước.
- Khi viết note core mới, link ngược về mental model liên quan nếu concept dựa trên reference, object identity, lifecycle, hoặc class loading.

## Related notes

[[001-java-in-jvm-eyes]]

[[004-pass-by-value]]

[[011-value-type-vs-reference-type]]

[[005-jvm-load-class]]
