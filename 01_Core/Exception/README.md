# Exceptions

## Purpose

Folder này giải thích cách Java biểu diễn lỗi: checked vs unchecked exception, try/catch/finally, try-with-resources, và custom exception.

Mục tiêu là viết error handling có context, không nuốt lỗi, và biết khi nào exception là API contract.

## Suggested reading order

1. `001-checked-vs-unchecked.md`
2. `002-try-catch-finally.md`
3. `003-try-with-resources.md`
4. `004-custom-exception.md`

## How to use this folder

- Khi phân vân checked hay unchecked, đọc `001`.
- Khi catch exception nhưng không biết làm gì tiếp, đọc `002` và convention exception.
- Khi quản lý resource, đọc `003` trước khi dùng `finally` thủ công.
- Khi tạo domain exception, đọc `004`.

## Related notes

[[../../00_Java-Conventions/013-exception-handling-guidelines]]

[[../../04_Lessons-from-bugs/009-unsupported-operation-exception]]
