# IO

## Purpose

Folder này giải thích Java IO/NIO ở mức core: byte stream vs character reader, buffering, `Path`/`Files`, serialization, và khác biệt IO/NIO.

Mục tiêu là xử lý input/output rõ encoding, resource lifecycle, và error boundary.

## Suggested reading order

1. `001-input-stream-vs-reader.md`
2. `002-buffered-reader.md`
3. `003-nio-vs-io.md`
4. `004-serialization.md`
5. `005-path-and-files.md`

## How to use this folder

- Khi đọc text, phân biệt byte với character trước.
- Khi quản lý file/path, ưu tiên `Path` và `Files` thay vì string path rời rạc.
- Khi thấy serialization, đọc note riêng để hiểu trade-off và rủi ro.

## Related notes

[[../Exception/003-try-with-resources]]

[[../../00_Java-Conventions/013-exception-handling-guidelines]]
