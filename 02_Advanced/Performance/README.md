# Performance

## Purpose

Folder này ghi lại performance engineering trong Java: profiling tools, memory profiling, CPU profiling, JMH benchmarking, và pitfalls.

Mục tiêu là đo trước khi tối ưu, phân biệt symptom với root cause, và tránh micro-optimization không có evidence.

## Suggested reading order

1. `001-profiling-tools.md`
2. `002-memory-profiling.md`
3. `003-cpu-profiling.md`
4. `004-benchmarking-with-jmh.md`
5. `005-common-performance-pitfalls.md`

## How to use this folder

- Khi app chậm, bắt đầu từ profiling tools thay vì đoán bottleneck.
- Khi memory tăng, đọc memory profiling cùng heap dump/JVM notes.
- Khi benchmark method nhỏ, dùng JMH thay vì `System.currentTimeMillis()` thủ công.

## Related notes

[[01-programming-languages/java/02_Advanced/JVM-Internals/README]]

[[01-programming-languages/java/01_Core/Memory/README]]
