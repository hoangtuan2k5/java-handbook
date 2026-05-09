# File Naming

## Why this file exists

File name là entry point đầu tiên khi search, grep, hoặc đọc tree. Trong Java, file naming còn gắn chặt với public class naming.

## Rules

- File name phải khớp với public top-level type trong file.
- Dùng PascalCase.
- Không dùng tên mơ hồ kiểu `Test`, `Util`, `Manager`, `Data` trừ khi nghĩa thật sự rõ.
- Test file thường dùng cùng tên class production kèm suffix `Test`, ví dụ `OrderServiceTest.java`.
- File chứa package-private top-level class vẫn nên dùng tên class chính dễ tìm nhất.

## Nuance

Java bắt buộc file name khớp với `public` top-level type như class, interface, enum, record, hoặc annotation. Nếu file không có public top-level type, JLS không ép file name theo cùng cách đó, nhưng convention tốt vẫn là đặt theo type chính để IDE, grep, stack trace, và reviewer tìm được nhanh.

Nói ngắn gọn: với public type, đây là requirement; với package-private type, đây là readability convention.

## Good examples

```text
OrderService.java
OrderServiceTest.java
PaymentRequest.java
InvalidOrderStateException.java
```

## Bad examples

```text
order service.java
exercise1.java
Helper.java
Stuff.java
```

## Notes

Nếu phải giải thích file name bằng miệng thì thường file name đó chưa đủ tốt.

## Official references

- [JLS: Top Level Class and Interface Declarations](https://docs.oracle.com/javase/specs/jls/se21/html/jls-7.html#jls-7.6)
- [Java Code Conventions: File Names](https://www.oracle.com/java/technologies/javase/codeconventions-fileorganization.html)

## Related rules

[[003-package-naming]]

[[005-class-naming]]
