# Advanced Data Types

## Purpose

Folder này ghi lại các type hiện đại hoặc Java-specific hơn: enum, record, sealed class, và pattern matching.

Mục tiêu là biết khi nào nên dùng language feature để model domain rõ hơn, và khi nào feature đó làm thiết kế cứng hoặc phức tạp quá mức.

## Suggested reading order

1. `001-enum.md`
2. `002-record.md`
3. `003-sealed-class.md`
4. `004-pattern-matching.md`

## How to use this folder

- Khi một field chỉ có tập giá trị hữu hạn, đọc `001`.
- Khi cần immutable data carrier, đọc `002`.
- Khi muốn giới hạn subtype, đọc `003`.
- Khi code có nhiều `instanceof`/cast, đọc `004`.

## Related notes

[[../../00_Java-Conventions/007-enum-record-annotation-naming]]

[[../../00_Mental-Models/010-Interface-vs-Abstract]]
