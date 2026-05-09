# Object Methods

## Purpose

Folder này tập trung vào các method và contract nền của object trong Java: `toString`, `equals`, `hashCode`, cloning, finalization, và cleanup.

Mục tiêu là hiểu object behavior ở level API, không chỉ override method cho đủ IDE suggestion.

## Suggested reading order

1. `001-to-string.md`
2. `002-equals-and-hash-code-contract.md`
3. `003-clone-and-cloneable.md`
4. `004-finalize-and-cleaner.md`

## How to use this folder

- Khi object cần log/debug tốt hơn, đọc `001`.
- Khi object làm key trong `Map` hoặc nằm trong `Set`, đọc `002` trước.
- Khi cần copy object, đọc `003` cùng memory note về shallow/deep copy.
- Khi thấy code dùng `finalize`, đọc `004` để hiểu vì sao nên tránh.

## Related notes

[[../Types-and-Variables/006-equals-vs-hash-code]]

[[../Memory/004-shallow-copy-vs-deep-copy]]
