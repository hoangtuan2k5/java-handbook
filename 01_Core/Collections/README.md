# Collections

## Purpose

Folder này là nền tảng chọn và dùng Java Collections: List, Set, Map, Iterator, queue/deque/stack, sorted collections, utility methods, và concurrent map cơ bản.

Mục tiêu là chọn data structure theo semantics và cost, không chỉ theo thói quen dùng `ArrayList` hoặc `HashMap`.

## Suggested reading order

1. `001-list-vs-set-vs-map.md`
2. `002-array-list-vs-linked-list.md`
3. `003-hash-map.md`
4. `004-iterator.md`
5. `005-comparable-vs-comparator.md`
6. `006-hash-set-vs-tree-set-vs-linked-hash-set.md`
7. `007-hash-map-vs-linked-hash-map-vs-tree-map.md`
8. `008-queue-vs-deque-vs-stack.md`
9. `009-collections-utility-class.md`
10. `010-concurrent-hash-map-vs-hash-map.md`

## How to use this folder

- Khi cần membership/uniqueness, bắt đầu từ Set notes.
- Khi cần key-value lookup, đọc HashMap và Map comparison notes.
- Khi cần ordering, đọc Comparator và Tree/Linked variants.
- Khi code multithread dùng map, đọc `010` nhưng không nhầm collection thread-safe với business logic thread-safe.

## Related notes

[[../../00_Mental-Models/011-value-type-vs-reference-type]]

[[../Types-and-Variables/006-equals-vs-hash-code]]
