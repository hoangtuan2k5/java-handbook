# Security

## Purpose

Folder này ghi lại secure coding, cryptography trong Java, và vulnerabilities phổ biến.

Mục tiêu là tránh lỗi bảo mật do API dùng sai, logging sai dữ liệu nhạy cảm, crypto tự chế, hoặc trust boundary mơ hồ.

## Suggested reading order

1. `001-secure-coding-practices.md`
2. `002-cryptography-in-java.md`
3. `003-common-vulnerabilities.md`

## How to use this folder

- Khi xử lý input ngoài hệ thống, đọc secure coding trước.
- Khi cần hash/encrypt/sign, đọc crypto note và ưu tiên library/protocol chuẩn.
- Khi review production code, dùng vulnerabilities note như checklist.

## Related notes

[[../../00_Java-Conventions/014-logging-guidelines]]

[[../../00_Java-Conventions/013-exception-handling-guidelines]]
