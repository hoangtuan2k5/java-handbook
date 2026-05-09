# Creational Patterns

## Purpose

Folder này tập trung vào cách tạo object: Singleton, Builder, Factory, và Abstract Factory.

Mục tiêu là kiểm soát construction complexity mà không giấu dependency hoặc làm object lifecycle khó hiểu.

## Suggested reading order

1. `001-Singleton.md`
2. `002-Builder.md`
3. `003-Factory.md`
4. `004-Abstract-Factory.md`

## How to use this folder

- Khi constructor quá dài hoặc object có nhiều optional fields, đọc Builder.
- Khi caller không nên biết concrete class, đọc Factory/Abstract Factory.
- Khi gặp Singleton, kiểm tra lifecycle, shared mutable state, và testability trước khi dùng.

## Related notes

[[../../../00_Mental-Models/008-Object-lifecycle]]

[[../../../00_Java-Conventions/005-class-naming]]
