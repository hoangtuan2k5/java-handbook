# Reflection and Annotation

## Purpose

Folder này giải thích reflection, annotation, custom annotation, dynamic proxy, và annotation processing.

Mục tiêu là hiểu các cơ chế framework như Spring/JUnit/Mockito thường dựa vào, đồng thời biết ranh giới giữa compile-time metadata và runtime behavior.

## Suggested reading order

1. `001-Reflection.md`
2. `002-Annotation.md`
3. `003-Custom-Annotation.md`
4. `004-Dynamic-proxy.md`
5. `005-annotation-processor.md`

## How to use this folder

- Khi framework “tự động” gọi code hoặc inject dependency, đọc reflection/annotation trước.
- Khi debug proxy behavior, đọc dynamic proxy cùng mental model class vs object.
- Khi logic chạy lúc compile/build, đọc annotation processor và compile-time/runtime note.

## Related notes

[[../../00_Mental-Models/005-JVM-load-class]]

[[../../00_Mental-Models/006-Compile-time-vs-Runtime]]

[[../../00_Java-Conventions/007-enum-record-annotation-naming]]
