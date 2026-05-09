# Generics

## Purpose

Folder này giải thích generic type parameter, wildcard, type erasure, và khác biệt giữa generic method với generic class.

Mục tiêu là dùng generics để tăng type safety mà không làm API khó đọc hoặc rơi vào unchecked cast không cần thiết.

## Suggested reading order

1. `001-what-is-T.md`
2. `002-wildcard.md`
3. `003-type-erasure.md`
4. `004-generic-method-vs-generic-class.md`

## How to use this folder

- Khi thấy `T`, `E`, `K`, `V`, đọc `001`.
- Khi API cần nhận nhiều subtype/supertype, đọc `002`.
- Khi gặp unchecked warning hoặc runtime cast issue, đọc `003`.
- Khi phân vân đặt type parameter ở method hay class, đọc `004`.

## Related notes

[[../../00_Java-Conventions/010-constants-and-generics-naming]]

[[../Collections/001-List-vs-Set-vs-Map]]
