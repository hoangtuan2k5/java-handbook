# Structural Patterns

## Purpose

Folder này tập trung vào cách ghép object/class để tạo interface hoặc behavior mới: Proxy, Decorator, Adapter, và Facade.

Mục tiêu là giảm coupling và làm boundary dễ đọc hơn, không tạo thêm wrapper chỉ để code trông “pattern-like”.

## Suggested reading order

1. `001-Proxy.md`
2. `002-Decorator.md`
3. `003-Adapter.md`
4. `004-Facade.md`

## How to use this folder

- Khi cần intercept call, đọc Proxy.
- Khi cần thêm behavior quanh object cùng interface, đọc Decorator.
- Khi bridge API không khớp nhau, đọc Adapter.
- Khi subsystem quá phức tạp với caller, đọc Facade.

## Related notes

[[../../../01_Core/Reflection-and-Annotation/004-Dynamic-proxy]]

[[../../../00_Mental-Models/010-Interface-vs-Abstract]]
