# Behavioral Patterns

## Purpose

Folder này tập trung vào cách object phối hợp behavior: Observer, Strategy, Template Method, và Chain of Responsibility.

Mục tiêu là tách decision/flow/change point rõ ràng để code dễ mở rộng mà không rơi vào inheritance hoặc callback rối.

## Suggested reading order

1. `001-Observer.md`
2. `002-Strategy.md`
3. `003-Template-Method.md`
4. `004-Chain-of-responsibility.md`

## How to use this folder

- Khi nhiều object cần phản ứng với event, đọc Observer.
- Khi thuật toán thay đổi theo context, đọc Strategy.
- Khi cần giữ skeleton flow nhưng cho subclass customize bước nhỏ, đọc Template Method.
- Khi request đi qua nhiều handler có thể xử lý, đọc Chain of Responsibility.

## Related notes

[[../../../00_Mental-Models/010-Interface-vs-Abstract]]

[[../../../01_Core/Functional/002-Functional-Interface]]
