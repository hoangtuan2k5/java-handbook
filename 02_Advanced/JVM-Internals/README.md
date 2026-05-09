# JVM Internals

## Purpose

Folder này đi sâu vào JVM runtime: classloader, bytecode/class file, JIT, GC algorithms/tuning, JVM flags, Java Memory Model, và heap dump analysis.

Mục tiêu là debug hành vi runtime bằng model đúng, không biến JVM thành black box.

## Suggested reading order

1. `001-classloader-mechanism.md`
2. `002-bytecode-and-class-file.md`
3. `003-jit-compiler.md`
4. `004-gc-algorithms.md`
5. `005-gc-tuning.md`
6. `006-jvm-flags.md`
7. `007-memory-model-jmm.md`
8. `008-heap-dump-and-analysis.md`

## How to use this folder

- Khi app fail ở startup/classpath, đọc classloader và bytecode.
- Khi performance lạ, đọc JIT và profiling notes trước khi tune flags.
- Khi concurrency bug subtle, đọc JMM cùng core multithreading.
- Khi memory tăng bất thường, đọc heap dump analysis.

## Related notes

[[../../00_Mental-Models/005-JVM-load-class]]

[[01-programming-languages/java/01_Core/Memory/README]]

[[01-programming-languages/java/01_Core/Multithreading/README]]
