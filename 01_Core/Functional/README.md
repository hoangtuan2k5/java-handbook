# Functional Java

## Purpose

Folder này ghi lại functional-style features trong Java: lambda, functional interface, Stream API, Optional, method reference, và collectors.

Mục tiêu là dùng functional style để code rõ intent hơn, không biến stream chain thành logic khó debug.

## Suggested reading order

1. `001-lambda.md`
2. `002-functional-interface.md`
3. `003-stream-api.md`
4. `004-optional.md`
5. `005-method-reference.md`
6. `006-collector-and-grouping-by.md`

## How to use this folder

- Khi mới học functional style, đọc lambda và functional interface trước.
- Khi xử lý collection pipeline, đọc Stream API và collectors.
- Khi model absence trong return value, đọc Optional nhưng không lạm dụng cho field/parameter nếu không hợp context.

## Related notes

[[01-programming-languages/java/01_Core/Collections/README]]

[[../../00_Mental-Models/010-Interface-vs-Abstract]]
