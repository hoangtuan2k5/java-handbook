# Types and Variables

## Purpose

Folder này xây nền về type, value, reference, `null`, casting, và equality/hashCode trong Java.

Đây là nhóm nên đọc sớm vì rất nhiều bug Java đến từ nhầm lẫn giữa primitive/wrapper, identity/equality, hoặc object có thể `null`.

## Suggested reading order

1. `001-primitive-vs-wrapper.md`
2. `002-string-immutable-and-pool.md`
3. `003-var-type-inference.md`
4. `004-casting.md`
5. `005-null.md`
6. `006-equals-vs-hash-code.md`

## How to use this folder

- Khi gặp bug `NullPointerException`, đọc `005-null.md` trước.
- Khi object so sánh sai hoặc HashMap/HashSet behave lạ, đọc `006-equals-vs-hash-code.md`.
- Khi code có boxing/unboxing hoặc generic collection, đọc `001-primitive-vs-wrapper.md`.

## Related notes

[[../../00_Mental-Models/002-Everything-is-object]]

[[../../00_Mental-Models/011-value-type-vs-reference-type]]
