# Lessons from Java Bugs

## Purpose

Folder này lưu bài học từ lỗi thật hoặc hiểu nhầm thật khi học/viết Java.

Mục tiêu là bắt đầu từ symptom cụ thể, tìm root cause, rồi rút ra prevention có thể áp dụng lại.

## Suggested reading order

1. `001-null-pointer-exception.md`
2. `002-concurrent-modification-exception.md`
3. `003-equals-vs-double-equal.md`
4. `004-stack-overflow-error.md`
5. `005-class-cast-exception.md`
6. `006-out-of-memory-error.md`
7. `007-number-format-exception.md`
8. `008-array-index-out-of-bounds-exception.md`
9. `009-unsupported-operation-exception.md`

## How to use this folder

- Khi gặp exception cụ thể, đọc bug lesson tương ứng trước khi đọc lý thuyết dài.
- Sau khi hiểu root cause, đi tới note core/advanced liên quan để củng cố mental model.
- Khi thêm bug lesson mới, không dừng ở “lần sau cẩn thận hơn”; phải có prevention cụ thể.

## Related notes

[[01-programming-languages/java/01_Core/Exception/README]]

[[01-programming-languages/java/01_Core/Memory/README]]

[[01-programming-languages/java/01_Core/Multithreading/README]]
